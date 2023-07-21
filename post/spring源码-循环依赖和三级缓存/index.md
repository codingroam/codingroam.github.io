# Spring源码-循环依赖和三级缓存

Spring源码（循环依赖和三级缓存）
<!--more-->




目录 
循环依赖 多级缓存 一级缓存 二级缓存 当循环依赖遇上AOP 三级缓存 Spring三级缓存源码实现 总结



  BeanFactory作为bean工厂管理我们的单例bean，那么肯定需要有个缓存来存储这些单例bean，在Spring中就是一个Map结构的缓存，key为beanName，value为bean。在获取一个bean的时候，先从缓存中获取，如果没有获取到，那么触发对象的实例化和初始化操作，完成之后再将对象放入缓存中，这样就能实现一个简单的bean工厂。   由于Spring提供了依赖注入的功能，支持将一个对象自动注入到另一个对象的属性中，这就可能出现**循环依赖**（区别于dependsOn）的问题：A对象在创建的时候依赖于B对象，而B对象在创建的时候依赖于A对象。 ![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/20210518132315114.png)   可以看到，由于A依赖B，所以创建A的时候触发了创建B，而B又依赖A，又会触发获取A，但是此时A正在创建中，还不在缓存中，就引发了问题。


## 一级缓存

  前面引出了循环依赖的问题，那么该如何解决呢？其实很简单，单单依赖一级缓存就能解决。对于一级缓存，我们不再等对象初始化完成之后再存入缓存，而是等对象实例化完成后就存入一级缓存，由于此时缓存中的bean还没有进行初始化操作，可以称之为**早期对象**，也就是将A对象**提前暴露**了。这样B在创建过程中获取A的时候就能从缓存中获取到A对象，最后在A对象初始化工作完成后再更新一级缓存即可，这样就解决了循环依赖的问题。   但是这样又引出了另一个问题：早期对象和完整对象都存在于一级缓存中，如果此时来了其它线程并发获取bean，就可能从一级缓存中获取到不完整的bean，这明显不行，那么我们不得已只能在从一级缓存获取对象处加一个互斥锁，以避免这个问题。   而加互斥锁也带来了另一个问题，容器刷新完成后的普通获取bean的请求都需要竞争锁，如果这样处理，在高并发场景下使用spring的性能一定会极低。

## 二级缓存

  既然只依赖一级缓存解决循环依赖需要靠加锁来保证对象的安全发布，而加锁又会带来性能问题，那该如何优化呢？答案就是引入另一个缓存。这个缓存也是一个Map结构，key为beanName，value为bean，而这个缓存可以称为**二级缓存**。我们将早期对象存到二级缓存中，一级缓存还是用于存储完整对象（对象初始化工作完成后），这样在接下来B创建的过程中获取A的时候，先从一级缓存获取，如果一级缓存没有获取到，则从二级缓存获取，虽然从二级缓存获取的对象是早期对象，但是站在对象内存关系的角度来看，二级缓存中的对象和后面一级缓存中的对象（指针）都指向同一对象，区别是对象处于不同的阶段，所以不会有什么问题。   既然获取bean的逻辑是先从一级缓存获取，没有的话再从二级缓存获取，那么也可能出现其它线程获取到不完整对象的问题，所以还是需要加互斥锁。不过这里的加锁逻辑可以下沉到二级缓存，因为早期对象存储在二级缓存中，从一级缓存获取对象不用加锁，这样的话当容器初始化完成之后，普通的getBean请求可以直接从一级缓存获取对象，而不用去竞争锁。

## 当循环依赖遇上AOP

  似乎二级缓存已经解决了循环依赖的问题，看起来也非常简单，但是不要忘记Spring提供的另一种特性：AOP。Spring支持以CGLIB和JDK动态代理的方式为对象创建代理类以提供AOP支持，在前面总结bean生命周期的文章中已经提到过，代理对象的创建(通常)是在bean初始化完成之后进行（通过BeanPostProcessor后置处理器）的，而且按照正常思维来看，一个代理对象的创建也应该在原对象完整的基础上进行，但是当循环依赖遇上了AOP就不那么简单了。   还是在前面A和B相互依赖的场景中，试想一下如果A需要被代理呢？由于二级缓存中的早期对象是原对象，而代理对象是在A初始化完成之后再创建的，这就导致了B对象中引用的A对象不是代理对象，于是出现了问题。   要解决这问题也很简单，把代理对象提前创建不就行了？也就是如果没有循环依赖，那么代理对象还是在初始化完成后创建，如果有循环依赖，那么就提前创建代理对象。那么怎么判断发生了循环依赖呢？在B创建的过程中获取A的时候，发现二级缓存中有A，就说明发生了循环依赖，此时就为A创建代理对象，将其覆盖到二级缓存中，并且将代理对象复制给B的对应属性，解决了问题。当然，最终A初始化完成之后，在一级缓存中存放的肯定是代理对象。   如果在A和B相互依赖的基础上，还有一个对象C也依赖了A：A依赖B，B依赖A，A依赖C，C依赖A。那么在为A创建代理对象的时候，就要注意不能重复创建。

>可以在对象实例化完成之后，将其beanName存到一个Set结构中，标识对应的bean正在创建中，而当其他对象创建的过程依赖某个对象的时候，判断其是否在这个set中，如果在就说明发生了循环依赖。

## 三级缓存

  虽然仅仅依靠二级缓存能够解决循环依赖和AOP的问题，但是从解决方案上看来，维护代理对象的逻辑和getBean的逻辑过于耦合，Spring没有采取这种方案，而是引入了另一个**三级缓存**。三级缓存的key还是为beanName，但是value是一个函数（ObjectFactory#getBean方法），在该函数中执行获取早期对象的逻辑：getEarlyBeanReference方法。 在getEarlyBeanReference方法中，Spring会调用所有SmartInstantiationAwareBeanPostProcessor的getEarlyBeanReference方法，通过该方法可以修改早期对象的属性或者替换早期对象。这个是Spring留给开发者的另一个扩展点，虽然我们很少使用，不过在循环依赖遇到AOP的时候，代理对象就是通过这个后置处理器创建。


  那么三级缓存在spring中是如何体现的呢？我们来到DefaultSingletonBeanRegistry中：

```java
//一级缓存，也就是单例池，存储最终对象
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
//二级缓存，存储早期对象
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
//三级缓存，存储的是一个函数接口，
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```

  其实所谓三级缓存，在源码中就是3个Map，一级缓存用于存储最终单例对象，二级缓存用于存储早期对象，三级缓存用于存储函数接口。   在容器刷新时调用的十几个方法中，**finishBeanFactoryInitialization**方法主要用于实例化单例bean，在其逻辑中会冻结BeanDefinition的元数据，接着调用beanFactory的preInstantiateSingletons方法，循环所有被管理的beanName，依次创建Bean，我们这里主要关注创建bean的逻辑，也就是AbstractBeanFactory的doGetBean方法（该方法很长，这里只贴了**部分代码**）：

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
   
		//解析FactoryBean的name(&)和别名
		final String beanName = transformedBeanName(name);
		Object bean;
		//尝试从缓存中获取对象
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
   
			//包含了处理FactoryBean的逻辑，可以通过&+beanName获取原对象，通过beanName获取真实对象
			//FactoryBean的真实bean有单独的缓存factoryBeanObjectCache（Map结构）存放
			//如果是普通的bean，那么直接返回对应的对象
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}
		else {
   
			//只能解决单例对象的循环依赖
			if (isPrototypeCurrentlyInCreation(beanName)) {
   
				throw new BeanCurrentlyInCreationException(beanName);
			}
			//如果存在父工厂，并且当前工厂中不存在对应的BeanDefinition，那么尝试到父工厂中寻找
			//比如spring mvc整合spring的场景
			BeanFactory parentBeanFactory = getParentBeanFactory();
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
   
				//bean的原始名称
				String nameToLookup = originalBeanName(name);
				if (parentBeanFactory instanceof AbstractBeanFactory) {
   
					return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
							nameToLookup, requiredType, args, typeCheckOnly);
				}
				else if (args != null) {
   
					return (T) parentBeanFactory.getBean(nameToLookup, args);
				}
				else {
   
					return parentBeanFactory.getBean(nameToLookup, requiredType);
				}
			}

			try {
   
			
				//处理dependsOn的依赖(不是循环依赖)
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
   
					for (String dep : dependsOn) {
   
						if (isDependent(beanName, dep)) {
   
							//循环depends-on 抛出异常
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
						}
						registerDependentBean(dep, beanName);
						try {
   
							getBean(dep);
						}
						catch (NoSuchBeanDefinitionException ex) {
   
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
									"'" + beanName + "' depends on missing bean '" + dep + "'", ex);
						}
					}
				}
				//创建单例bean
				if (mbd.isSingleton()) {
   
					//把beanName 和一个ObjectFactory类型的singletonFactory传入getSingleton方法
					//ObjectFactory是一个函数，最终创建bean的逻辑就是通过回调这个ObjectFactory的getObject方法完成的
					sharedInstance = getSingleton(beanName, () -> {
   
						try {
   
							//真正创建bean的逻辑
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
   
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
			}
			catch (BeansException ex) {
   
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}
		return (T) bean;
	}
```

  对于单例对象，调用doGetBean方法的时候会调用getSingleton方法，该方法的入参是beanName和一个ObjectFactory的函数，在getSingleton方法中通过回调ObjectFactory的getBean方法完成bean的创建，那我们来看看getSingleton方法(部分代码)：

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
   
		Assert.notNull(beanName, "Bean name must not be null");
		//互斥锁
		synchronized (this.singletonObjects) {
   
			//首先尝试从尝试从单例缓存池中获取对象，如果获取到了对象则直接返回，否则进入创建的逻辑
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
   
				//在创建之前将要创建的beanName存入singletonsCurrentlyInCreation中，这个是一个Map结构，用于标识单例正在创建中
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
   
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
   
					//回调singletonFactory的getObject方法，该函数在外层doGetBean方法中传入
					//实际调用了createBean方法
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				} finally {
   
					if (recordSuppressedExceptions) {
   
						this.suppressedExceptions = null;
					}
					//将beanName从正在创建单例bean的集合singletonsCurrentlyInCreation中移除
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
   
					//将创建好的单例bean放入一级缓存，并且从二级缓存、三级缓存中移除
					//添加到registeredSingletons中
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}
```

  getSingleton方法的主线逻辑很简单，就是通过传入的函数接口创建单例bean，然后存入一级缓存中，同时清理bean在二级缓存、三级缓存中对应的数据。前面已经分析过了，传入的函数接口调用的是createBean方法，那么我们又来到createBean方法，createBean方法主要调用doCreateBean方法，在doCreateBean调用之前会先调用nstantiationAwareBeanPostProcessor的**postProcessBeforeInstantiation**拦截bean的实例化，如果这里的后置处理器返回了bean，则不会到后面的doCreateBean方法中，不过我们这里不用关心这个逻辑，直接跳到doCreateBean方法（部分代码）：

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
   
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
   
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
   
			//创建bean，也是就是实例化，会把实例化后的bean包装在BeanWrapper中
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		//eanWrapper中获取到对象
		final Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
   
			mbd.resolvedTargetType = beanType;
		}
		synchronized (mbd.postProcessingLock) {
   
			if (!mbd.postProcessed) {
   
				try {
   
					//MergedBeanDefinitionPostProcessor后置处理器的处理
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
   
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		//判断该对象是否支持提前暴露，核心条件就是要求单例对象
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
   
			//把我们的早期对象包装成一个singletonFactory对象 该对象提供了一个getObject方法,该方法内部调用getEarlyBeanReference方法
			//将早期对象存入三级缓存，三级缓存存的是一个接口实现（ObjectFactory接口），其getBean方法会调用getEarlyBeanReference方法
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
   
			//属性赋值操作，这里就可能出现循环依赖问题
			populateBean(beanName, mbd, instanceWrapper);
			//实例化操作：调用三个Aware、后置处理器beforeInitialization（包含@PostConstruct的实现）、afterPropertiesSet、init-method、后置处理器afterInitialization等
			//个方法里的后置处理器可能会改变exposeObject
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
   
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
   
				throw (BeanCreationException) ex;
			}
			else {
   
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}
		if (earlySingletonExposure) {
   
			//这里的入参为false,表示只从一级缓存和二级缓存中获取
			//非循环依赖的情况下，不会调用到三级缓存，三级缓存中的接口会在单例对象入一级缓存的时候移除
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
   
					//如果获取到了早期暴露对象earlySingletonReference不为空
					if (exposedObject == bean) {
   
						//如果exposeObject在经过initializeBean方法后没有改变，那么说明
						//没有被后置处理器修改，那么用earlySingletonReference替换exposeObject
						exposedObject = earlySingletonReference;
				}
			}
		}

		return exposedObject;
	}
```

  这个方法的逻辑就是先实例化对象，如果对象支持暴露早期对象，那么会将早期对象作为入参传入getEarlyBeanReference方法，然后包装成一个ObjectFactory存入三级缓存，接着再调用populateBean方法填充对象的属性。而在填充对象属性的过程中，就可能发现循环依赖，比如当前正在创建A，然后在populateBean方法中发现了依赖B，那么就会调用getBean(B)，B也会走上面A走的这个流程，当B也走到populateBean方法填充属性的时候，又发现依赖了A，那么又会调用getBean(A)。那么在populateBean创建A的时候，会从三级缓存中获取bean，如果A需要被代理，那么会创建代理对象，这样B中的A属性就是代理对象了，接着把三级缓存获取的对象存入二级缓存中。 在B处理完成之后就回到了A的逻辑，假设populateBean(A)的逻辑完成了，那么接着进入initializeBean方法进行A的初始化操作，注意这里执行初始化操作的对象是A原对象，代理对象存储在二级缓存中。由于initializeBean方法会调用后置处理器的before-afterInitialization方法，这个后置处理器可能会改变对象，所以在后面的逻辑中，如果从二级缓存中获取到了A的代理对象，会判断原对象经过后置处理器后有没有变化，如果还是原对象，那么用二级缓存中的代理对象覆盖原对象，所以doCreateBean方法返回的就是代理对象，最终存入一级缓存中。流程就是：创建A实例对象-&gt;设置A的B属性-&gt;创建B实例对象-&gt;设置B的A属性-&gt;创建A的代理对象存入二级缓存-&gt;初始化A对象-&gt;从二级缓存取出A的代理对象覆盖A对象-&gt;A的代理对象存入一级缓存   我们需要关注的就是在B的populateBean逻辑里调用getBean(A)是什么样的逻辑。当然也是getBean-&gt;doGetBean-&gt;getSingleton这个逻辑，我们着重关注一下getSingleton方法的实现：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
   
		//首先从一级缓存中获取
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
   
			//如果从一级缓存没有获取到，并且获取的对象正在创建中，这里已经能够确定发生了循环依赖
			synchronized (this.singletonObjects) {
   
				//为了防止其它线程获取到不完整工单对象，这里使用同步锁控制
				//从二级缓存中获取早期对象
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
   
					//如果二级缓存中也没有获取到，且allowEarlyReference为true（这里传入的是true）
					//那么尝试从三级缓存中获取
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
   
						//从三级缓存中获取到了对象，注意三级缓存存储的是ObjectFactory
						//调用getObject方法，前面提到了早期暴露的对象存入三级缓存的ObjectFactory的getBean方法调用
						//的是getEarlyBeanReference方法，所以这里会调用getEarlyBeanReference方法
						singletonObject = singletonFactory.getObject();
						//把早期对象存入二级缓存
						this.earlySingletonObjects.put(beanName, singletonObject);
						//三级缓存中的接口没用了，直接移除
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```

  在getSingleton方法的逻辑中，先从一级缓存获取，如果一级缓存没有找到，那么如果获取的bean正在创建中，则从二级缓存获取，如果二级缓存没有找到，那么从三级缓存获取，三级缓存中存的是ObjectFactory实现，最终会调用其getBean方法获取bean，然后存入二级缓存中，同时清除三级缓存。同时提供了一个allowEarlyReference参数控制是否能从三级缓存中获取。   对于循环依赖的情况，getBean(A)-&gt;存入正在创建缓存-&gt;存入三级缓存-&gt;populateBean(A)-&gt;getBean(B)-&gt;populateBean(B)-&gt;getBean(A)-&gt;getSingleton(A)，当在populateBean(B)的过程中调用getSingleton(A)的时候，明显一级缓存和二级缓存都为空，但是三级缓存不为空，所以会通过三级缓存获取bean，三级缓存的创建逻辑如下：

```java
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
   
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
```

  所以我们直接来到getEarlyBeanReference方法获取早期对象：

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
   
		Object exposedObject = bean;
		//如果容器中有InstantiationAwareBeanPostProcessors后置处理器
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
   
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
   
				if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
   
					//找到SmartInstantiationAwareBeanPostProcessor后置处理器
					SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
					//调用SmartInstantiationAwareBeanPostProcessor的getEarlyBeanReference方法
					exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
				}
			}
		}
		return exposedObject;
	}
```

  这个getEarlyBeanReference方法的逻辑很简单，但是非常重要，该方法主要就是对SmartInstantiationAwareBeanPostProcessor后置处理器的调用，而循环依赖时的AOP就是通过这个**SmartInstantiationAwareBeanPostProcessor**的**getEarlyBeanReference**方法实现的，相关的具体类是AnnotationAwareAspectJAutoProxyCreator，这个留待AOP原理分析的文章中详细说明。



  多级缓存并不只是为了解决循环依赖和AOP的问题，还考虑到了逻辑的分离、结构的扩展性和保证数据安全前提下的效率问题等。由于存在二级缓存提前暴露不完整对象的情况，所以为了防止getSingleton方法返回不完整的bean，在该方法中使用了synchronized加锁，而为了不影响容器刷新完成后正常获取bean，从一级缓存获取对象没有加锁，事实上也不用加锁，因为一级缓存中要么没有对象，要么就是完整的对象。   通常情况下，容器刷新会触发单例对象的实例化和初始化，大致流程如下：![image-20230130143730546](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230130143730546.png)循环依赖发生在对象都属性赋值，也就是populateBean阶段，在对象实例化完成后将其包装成一个函数接口ObjectFactory存入三级缓存，通过getEarlyBeanReference方法触发SmartInstantiationAwareBeanPostProcessor后置处理器的执行，如果循环依赖遇上了AOP，那么就在这里进行处理。   这里每个单例对象实例化之后存入三级缓存，即使没有发生循环依赖，这个是为之后可能发生的循环依赖做准备，如果最终没有发生循环依赖，那么对象实例化之后，正常初始化，然后存入一级缓存即可，而在存入一级缓存的时候会清空二级和三级缓存。 

![image-20230130144022403](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230130144022403.png)



---

> 作者: wang  
> URL: https://codingroam.github.io/post/spring%E6%BA%90%E7%A0%81-%E5%BE%AA%E7%8E%AF%E4%BE%9D%E8%B5%96%E5%92%8C%E4%B8%89%E7%BA%A7%E7%BC%93%E5%AD%98/  

