# B树与B+树


B树、B+树 mysql存储引擎

<!--more-->

### 一，b树

b树（balance tree）和b+树应用在数据库索引，可以认为是m叉的多路平衡查找树，但是从理论上讲，二叉树查找速度和比较次数都是最小的，为什么不用二叉树呢？

​         平衡二叉树的查找效率是非常高的，并可以通过降低树的深度来提高查找的效率。但是当数据量非常大，树的存储的元素数量是有限的，这样会导致二叉查找树结构由于树的深度过大而造成磁盘I/O读写过于频繁，进而导致查询效率低下。另外数据量过大会导致内存空间不够容纳平衡二叉树所有结点的情况。B树是解决这个问题的很好的结构。数据库索引是存储在磁盘上的，当数据量大时，就不能把整个索引全部加载到内存了，只能逐一加载每一个磁盘页（对应索引树的节点）。所以我们**要减少IO次数，对于树来说，IO次数就是树的高度，而“矮胖”就是b树的特征之一，它的每个节点最多包含m个孩子，m称为b树的阶，m的大小取决于磁盘页的大小**。

​            **b树在查询时的比较次数并不比二叉树少，尤其是节点中的数非常多时，但是内存的比较速度非常快，耗时可以忽略，所以只要树的高度低，IO少，就可以提高查询性能，这是b树的优势之一。**

### 二，b+树

b+树，是b树的一种变体，查询性能更好。m阶的b+树的特征：

有n棵子树的非叶子结点中含有n个关键字（b树是n-1个），这些关键字不保存数据，只用来索引，所有数据都保存在叶子节点（b树是每个关键字都保存数据）。
所有的叶子结点中包含了全部关键字的信息，及指向含这些关键字记录的指针，且叶子结点本身依关键字的大小自小而大顺序链接。
所有的非叶子结点可以看成是索引部分，结点中仅含其子树中的最大（或最小）关键字。
通常在b+树上有两个头指针，一个指向根结点，一个指向关键字最小的叶子结点。
同一个数字会在不同节点中重复出现，根节点的最大元素就是b+树的最大元素。

### **三、b+树相比于b树的查询优势：** 

1. **b+树的中间节点不保存数据，所以磁盘页能容纳更多节点元素，更“矮胖”；**
2. **b+树查询必须查找到叶子节点，b树只要匹配到即可不用管元素位置，因此b+树查找更稳定（并不慢）；**
3. **对于范围查找来说，b+树只需遍历叶子节点链表即可，b树却需要重复地中序遍历**。

### 四、图示

1. B+树

![img](https://csdn-img-blog.oss-cn-beijing.aliyuncs.com/701810cad4564258a60d7fe8d4460039.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzE1NjY5OQ==,size_16,color_FFFFFF,t_70#pic_center)

2. InnoDB

   ![img](https://csdn-img-blog.oss-cn-beijing.aliyuncs.com/fa349875a26c49bcb797c63a4fcd9125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzE1NjY5OQ==,size_16,color_FFFFFF,t_70#pic_center)

3. MyISAM

   ![img](https://csdn-img-blog.oss-cn-beijing.aliyuncs.com/c063223b04864d39af577876642e0ddf.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzE1NjY5OQ==,size_16,color_FFFFFF,t_70#pic_center)


---

> 作者: wang  
> URL: https://codingroam.github.io/post/b%E6%A0%91%E4%B8%8Eb+%E6%A0%91/  

