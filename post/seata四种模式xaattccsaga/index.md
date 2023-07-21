# Seata四种模式（XA、AT、TCC、SAGA）


Seata四种模式（XA、AT、TCC、SAGA）

<!--more-->

# Seata四种模式（XA、AT、TCC、SAGA）

### 1、XA模式

#### XA模式原理：

XA 规范 是 X/Open 组织定义的分布式事务处理（DTP，Distributed Transaction Processing）标准，XA 规范 描述了全局的TM与局部的RM之间的接口，几乎所有主流的数据库都对 XA 规范 提供了支持。

![image-20230126220152871](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126220152871.png)

> 如果有失败的就会回滚事务

#### seata的XA模式

seata的XA模式做了一些调整，但大体相似：

![image-20230126220326443](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126220326443.png)

RM一阶段的工作：

1. 注册分支事务到TC
1. 执行分支业务sql但不提交
1. 报告执行状态到TC

TC二阶段的工作：

1. TC检测各分支事务执行状态
1. 如果都成功，通知所有RM提交事务

3. 如果有失败，通知所有RM回滚事务

RM二阶段的工作：

1. 接收TC指令，提交或回滚事务

xa模式的优点：

1. 事务的强一致性，满足ACID原则。

2. 常用数据库都支持，实现简单，并且没有代码侵入

xa模式的缺点：

1. 因为一阶段需要锁定数据库资源，等待二阶段结束才释放，性能较差
   依赖关系型数据库实现事务

实现XA模式

Seata的starter已经完成了XA模式的自动装配，实现非常简单，步骤如下：
修改application.yml文件（每个参与事务的微服务），开启XA模式：

```
配置seata的注册中心
seata:
  data-source-proxy-mode: XA # 选择XA模式
```

注意：是每一个微服务都需要

给发起全局事务的入口方法添加@GlobalTransactional注解，本例中是OrderServiceImpl中的create方法：

启动所有微服务，postman进行接口测试

先进行正确的测试，再继续错误的测试

> 错误设置，购买商品超出原来剩余的商品数就会让数据库报错测试是否可以事务回滚

注意：如果测试接口报错响应时间过长，那么就应该设置响应的时间大一点，如下图，然后重启seata

![image-20230126220912891](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126220912891.png)

成功的可以查看seate的控制输出，可以看到事务回滚

IDEA输出

![image-20230126220944119](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126220944119.png)

查看数据库的数据是否有被更新

### 2、AT模式

#### 2.1、 AT模式同样是分阶段提交的事务模型，不过缺弥补了XA模型中资源锁定周期过长的缺陷。

![image-20230126221049428](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126221049428.png)

**阶段一RM的工作**：

注册分支事务
记录undo-log（数据快照，JSON格式）
执行业务sql并提交
报告事务状态

**阶段二提交时RM的工作**：

删除undo-log即可

**阶段二回滚时RM的工作**：

根据undo-log恢复数据到更新前

执行示例
例如，一个分支业务的SQL是这样的：update tb_account set money = money - 10 where id = 1

![image-20230126221227467](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126221227467.png)

**AT模式与XA模式最大的区别是什么**

XA模式一阶段不提交事务，锁定资源；AT模式一阶段直接提交，不锁定资源。
XA模式依赖数据库机制实现回滚；AT模式利用数据快照实现数据回滚。
XA模式强一致；AT模式最终一致

#### 2.2、AT模式的脏写问题

![image-20230126221354825](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126221354825.png)

#### 2.3、AT模式的写隔离

![image-20230126221551524](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126221551524.png)

#### 2.4、AT模式的优缺点

**优点：**

1. 一阶段完成直接提交事务，释放数据库资源，性能比较好
1. 利用全局锁实现读写隔离
1. 没有代码侵入，框架自动完成回滚和提交

**缺点：**

1. 两阶段之间属于软状态，属于最终一致
1. 框架的快照功能会影响性能，但比XA模式要好很多

#### 2.5、实现AT模式

AT模式中的快照生成、回滚等动作都是由框架自动完成，没有任何代码侵入，因此实现非常简单。

1、创建相关的数据库文件

![image-20230126221859755](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126221859755.png)

lock_table（全局锁）表导入到TC服务关联的数据库

```
DROP TABLE IF EXISTS `lock_table`;
CREATE TABLE `lock_table`  (
  `row_key` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `xid` varchar(96) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `transaction_id` bigint(20) NULL DEFAULT NULL,
  `branch_id` bigint(20) NOT NULL,
  `resource_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `table_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `pk` varchar(36) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `gmt_create` datetime NULL DEFAULT NULL,
  `gmt_modified` datetime NULL DEFAULT NULL,
  PRIMARY KEY (`row_key`) USING BTREE,
  INDEX `idx_branch_id`(`branch_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

undo_log（记录快照）表导入到微服务关联的数据库

```
DROP TABLE IF EXISTS `undo_log`;
CREATE TABLE `undo_log`  (
  `branch_id` bigint(20) NOT NULL COMMENT 'branch transaction id',
  `xid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'global transaction id',
  `context` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'undo_log context,such as serialization',
  `rollback_info` longblob NOT NULL COMMENT 'rollback info',
  `log_status` int(11) NOT NULL COMMENT '0:normal status,1:defense status',
  `log_created` datetime(6) NOT NULL COMMENT 'create datetime',
  `log_modified` datetime(6) NOT NULL COMMENT 'modify datetime',
  UNIQUE INDEX `ux_undo_log`(`xid`, `branch_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = 'AT transaction mode undo table' ROW_FORMAT = Compact;
```

2、修改application.yml文件，将事务模式修改为AT模式即可：

```
#配置seata的注册中心
seata:
  data-source-proxy-mode: AT #选择XA模式
```

3、重启并测试

查看当前的数据库库存数量继续超库存创建订单进行执行错误回滚测试

查看IDEA的错误日志

![image-20230126221956361](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126221956361.png)

### 3、TCC模式

#### 3.1、tcc模式原理

TCC模式与AT模式非常相似，每阶段都是独立事务，不同的是TCC通过人工编码来实现数据恢复。需要实现三个方法：

**Try：资源的检测和预留；**
**Confirm：完成资源操作业务；要求 Try 成功 Confirm 一定要能成功。**
**Cancel：预留资源释放，可以理解为try的反向操作。**
举例，一个扣减用户余额的业务。假设账户A原来余额是100，需要余额扣减30元。

![image-20230126222108140](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126222108140.png)

TCC的工作模型图：

![image-20230126222214788](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126222214788.png)

#### 3.2、小结

TCC模式的优点：

1. 一阶段完成直接提交事务，释放数据库资源，性能好
1. 相比AT模型，无需生成快照，无需使用全局锁，性能最强
1. 不依赖数据库事务，而是依赖补偿操作，可以用于非事务型数据库

TCC模式的缺点：

1. 有代码侵入，需要人为编写try、Confirm和Cancel接口，太麻烦
1. 软状态，事务是最终一致
1. 需要考虑Confirm和Cancel的失败情况，做好幂等处理

#### 3.2、TCC模式实现案例

不是所有的业务都适合TCC模式，如库存，金额等就比较适合

改造account-service服务，利用TCC实现分布式事务

需求如下：

修改account-service，编写try、confirm、cancel逻辑：

1. try业务：添加冻结金额，扣减可用金额
1. confirm业务：删除冻结金额
1. cancel业务：删除冻结金额，恢复可用金额

保证confirm、cancel接口的幂等性允许空回滚拒绝业务悬挂

**TCC的空回滚和业务悬挂**

当某分支事务的try阶段阻塞时，可能导致全局事务超时而触发二阶段的cancel操作。在未执行try操作时先执

行了cancel操作，这时cancel不能做回滚，就是空回滚。

对于已经空回滚的业务，如果以后继续执行try，就永远不可能confirm或cancel，这就是业务悬挂。应当阻止

执行空回滚后的try操作，避免悬挂

**业务分析**

为了实现空回滚、防止业务悬挂，以及幂等性要求。我们必须在数据库记录冻结金额的同时，记录当前事务id和执行状态，为此我们设计了一张表（添加在微服务的数据库seata-demo中）：

```
DROP TABLE IF EXISTS `account_freeze_tbl`;
CREATE TABLE `account_freeze_tbl`  (
  `xid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `user_id` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `freeze_money` int(11) UNSIGNED NULL DEFAULT 0,
  `state` int(1) NULL DEFAULT NULL COMMENT '事务状态，0:try，1:confirm，2:cancel',
  PRIMARY KEY (`xid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = COMPACT;
```

Try业务：记录冻结金额和事务状态到account_freeze表，扣减account表可用金额

Confirm业务：根据xid删除account_freeze表的冻结记录

Cancel业务：修改account_freeze表，冻结金额为0，state为2，修改account表，恢复可用金额

如何判断是否空回滚：cancel业务中，根据xid查询account_freeze，如果为null则说明try还没做，需要空回

滚

如何避免业务悬挂：try业务中，根据xid查询account_freeze ，如果已经存在则证明Cancel已经执行，拒绝执

行try业务

**业务实现**

```java
1、声明TCC接口
@BusinessActionContextParameter()注解的参数才可以被BusinessActionContext获取到

package cn.itcast.account.service;

import io.seata.rm.tcc.api.BusinessActionContext;
import io.seata.rm.tcc.api.BusinessActionContextParameter;
import io.seata.rm.tcc.api.LocalTCC;
import io.seata.rm.tcc.api.TwoPhaseBusinessAction;

/**

 * 项目名称：seata-demo

 * 描述：TCC实现接口
   *

 * @author zhong

 * @date 2022-06-08 16:05
   */
   @LocalTCC
   public interface AccountTCCService {

   /**

    * 定义try,注释里面的值必须是与方法同名
    * @param userId
    * @param money
      */
      @TwoPhaseBusinessAction(name = "deduct",commitMethod = "confirm",rollbackMethod = "cancel")
      void deduct(@BusinessActionContextParameter(paramName = "userId") String userId,
              @BusinessActionContextParameter(paramName = "money")int money);

   /**

    * 定义Confirm
    * @param ctx 获取事务类型参数
    * @return
      */
      boolean confirm(BusinessActionContext ctx);

   /**

    * 定义Cancel
    * @param ctx
    * @return
      */
      boolean cancel(BusinessActionContext ctx);
      }



```

说明：对于account_freeze_tbl数据库表的操作与其他业务一样的，使用MP的CURD进行快速开发，需要实体类、mapper

2.、创建接口实现类

```java
package cn.itcast.account.service.impl;

import cn.itcast.account.entity.AccountFreeze;
import cn.itcast.account.mapper.AccountFreezeMapper;
import cn.itcast.account.mapper.AccountMapper;
import cn.itcast.account.service.AccountTCCService;
import io.seata.core.context.RootContext;
import io.seata.rm.tcc.api.BusinessActionContext;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**

 * 项目名称：seata-demo

 * 描述：
   *

 * @author zhong

 * @date 2022-06-08 16:25
   */
   @Slf4j
   @Service
   public class AccountTCCServiceImpl implements AccountTCCService {

   /**

    * 注入可用余额dao
      */
      @Autowired
      private AccountMapper accountMapper;

   /**

    * 注入冻结表dao
      */
      @Autowired
      private AccountFreezeMapper freezeMapper;

   /**

    * 资源检测和预留

    * 在数据库中设置了非负数字段限定，这里可以直接简化一步，如果为负数就会报错

    * @param userId

    * @param money
      */
      @Override
      @Transactional
      public void deduct(String userId, int money) {
      // 获取全局事务id
      String xid = RootContext.getXID();
      // 业务悬挂判断，如果freeze中有冻结记录，一定是CANCEL执行过，我要拒绝业务
      AccountFreeze accountFreeze = freezeMapper.selectById(xid);
      if(accountFreeze != null){
          // 一定是CANCEL执行过，我要拒绝业务
          return;
      }

      // 1、扣减可用余额
      accountMapper.deduct(userId,money);
      // 2、记录冻结金额，事务状态
      AccountFreeze freeze = new AccountFreeze();
      freeze.setXid(xid);
      freeze.setUserId(userId);
      freeze.setState(AccountFreeze.State.TRY);
      freeze.setFreezeMoney(money);
      freezeMapper.insert(freeze);
      }

   @Override
   public boolean confirm(BusinessActionContext ctx) {
       // 1、获取事务id
       String xid = ctx.getXid();
       // 2、根据事务id删除冻结数据
       int count = freezeMapper.deleteById(xid);
       return count == 1;
   }

   @Override
   public boolean cancel(BusinessActionContext ctx) {
       // 0、查询冻结记录
       String xid = ctx.getXid();
       // 获取用户id
       String userId = ctx.getActionContext("userId").toString();
       AccountFreeze freeze = freezeMapper.selectById(xid);
       // 空回滚的判断，为null代表没有try没有执行
       if(freeze == null){
           freeze = new AccountFreeze();
           freeze.setXid(xid);
           freeze.setUserId(userId);
           freeze.setState(AccountFreeze.State.CANCEL);
           freeze.setFreezeMoney(0);
           freezeMapper.insert(freeze);
       }

       // 幂等判断
       if(freeze.getState() == AccountFreeze.State.CANCEL){
           // 已经执行过了一次CANCEL，无需重复处理
           return true;
       }
       
       // 1、恢复可用余额
       accountMapper.refund(freeze.getUserId(), freeze.getFreezeMoney());
       // 2、将冻结金额清零，修改状态
       freeze.setFreezeMoney(0);
       freeze.setState(AccountFreeze.State.CANCEL);
       int count = freezeMapper.updateById(freeze);
       return count == 1;

   }
   }
```


3、修改业务层连接方式
改为上面定义的TCCM业务实现

![img](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/93b574a9f6a4408cb371dfd1d3c4a5bc.png)

4、测试数据
直接测试错误的信息

![image-20230126223228185](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126223228185.png)

当出现错误的时候会将会数据保存到数据库，如果是成功的那么就会删除

IDEA的报错输出

![image-20230126223248820](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126223248820.png)

4、SAGA模式

![image-20230126223308411](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126223308411.png)


Saga模式是SEATA提供的长事务解决方案。也分为两个阶段：

一阶段：直接提交本地事务
二阶段：成功则什么都不做；失败则通过编写补偿业务来回滚
Saga模式优点：

1. 事务参与者可以基于事件驱动实现异步调用，吞吐高

1. 一阶段直接提交事务，无锁，性能好
1. 不用编写TCC中的三个阶段，实现简单

缺点：

1. 软状态持续时间不确定，时效性差
1. 没有锁，没有事务隔离，会有脏写

![image-20230126223410575](https://bucket-typora-kw.oss-cn-beijing.aliyuncs.com/typora-image/image-20230126223410575.png)

---

> 作者: wang  
> URL: https://codingroam.github.io/post/seata%E5%9B%9B%E7%A7%8D%E6%A8%A1%E5%BC%8Fxaattccsaga/  

