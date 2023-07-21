# docker环境搭建redis集群并模拟扩缩容


docker环境搭建redis集群并模拟扩缩容
<!--more-->

## docker环境搭建redis集群并模拟扩缩容



在docker环境拉下最新的redis镜像

```
docker pull redis
```

有了redis镜像，开始搭建redis集群

### 1关闭防火墙

```java
[root@vm001 redis]# firewall-cmd --zone=public --add-port=6381-6390/tcp --permanent
success
[root@vm001 redis]# firewall-cmd --reload
success
```

### 2配置案例设计

master 6381-6383
slave 6384-6386
master1() - slave1()
master2() - slave2()
master3() - slave3()
master-slave实例具体的映射关系，会由redis自己匹配

### 3 新建6个docker容器实例

```
# 这里redis 使用的是latest版本，所以最后没有带版本号
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis --cluster-enabled yes --appendonly yes --port 6382

docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis --cluster-enabled yes --appendonly yes --port 6383

docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis --cluster-enabled yes --appendonly yes --port 6384

docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis --cluster-enabled yes --appendonly yes --port 6385

docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis --cluster-enabled yes --appendonly yes --port 6386

```

### 4 命令分步解析

```
docker run							-------- 创建并运行docker容器实例
--name redis-node-4					-------- 容器名字
--net host							-------- 使用宿主机的IP和端口，默认
--privileged=true					-------- 获取宿主机root用户权限
-v /data/redis/share/redis-node-4:/data			-------- 容器卷，宿主机地址:docker内部地址
redis											-------- redis镜像和版本号，这里redis是latest版本，所以不显示
--cluster-enabled yes							-------- 开启redis集群
 --appendonly yes 								-------- 开启持久化
 --port 6384									-------- redis端口号

```

### 5 进入容器redis-node-1并为6台机器构建集群关系

```
# 进入容器
docker exec -it redis-node-1 /bin/bash
# 构建集群关系
redis-cli --cluster create 192.168.226.128:6381 192.168.226.128:6382 192.168.226.128:6383 192.168.226.128:6384 192.168.226.128:6385 192.168.226.128:6386 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
## 槽位分配
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.226.128:6385 to 192.168.226.128:6381
Adding replica 192.168.226.128:6386 to 192.168.226.128:6382
Adding replica 192.168.226.128:6384 to 192.168.226.128:6383
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
### M: 指 master(主)， S: 指 slave(从)   三主三从
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[0-5460] (5461 slots) master
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[5461-10922] (5462 slots) master
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[10923-16383] (5461 slots) master
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
## 尝试自动分配槽位并请求确认，此时需要打 'yes'
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 192.168.226.128:6381)
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
## 告诉用户所有槽位都得到覆盖
[OK] All 16384 slots covered.


```

### 6 以6381作为切入点，查看集群状态

```
root@vm001:/data# redis-cli -p 6381
## cluster info
127.0.0.1:6381> cluster info
cluster_state:ok
cluster_slots_assigned:16384		# 节点分配
cluster_slots_ok:16384				# 节点个数
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6				# 集群内部总实例个数
cluster_size:3
cluster_current_epoch:6				# 集群内如当前运行实例个数
cluster_my_epoch:1
cluster_stats_messages_ping_sent:469
cluster_stats_messages_pong_sent:448
cluster_stats_messages_sent:917
cluster_stats_messages_ping_received:443
cluster_stats_messages_pong_received:469
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:917




root@vm001:/data# redis-cli -p 6381
## cluster nodes
127.0.0.1:6381> cluster nodes
# master
56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383@16383 master - 0 1651595835135 3 connected 10923-16383
# master
aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382@16382 master - 0 1651595837279 2 connected 5461-10922
# slave， 挂载在6383 master机上---根据id可得出
e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385@16385 slave 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 0 1651595832000 3 connected
# slave，挂载在6382 master机上---根据id可得出
8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384@16384 slave aa89bba715780e6bf30ecbb734580b9da68e394f 0 1651595836216 2 connected
# slave，挂载在6381 master机上---根据id可得出
71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386@16386 slave 94714f899698acb6e3d3584474c8bbbc7677e882 0 1651595834063 1 connected
# master， 当前机器
94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381@16381 myself,master - 0 1651595835000 1 connected 0-5460

```

**映射关系：**

```
master1(6381) - slave1(6386)
master2(6382) - slave2(6384)
master3(6383) - slave3(6385)
```

### 7 数据的读写储存

```
[root@vm001 admin]# docker exec -it redis-node-1 /bin/bash
root@vm001:/data# redis-cli -p 6381
127.0.0.1:6381> keys *
(empty array)
127.0.0.1:6381> set k1 v1
(error) MOVED 12706 192.168.226.128:6383
127.0.0.1:6381> set k2 v2
OK
127.0.0.1:6381> set k3 v3
OK
127.0.0.1:6381> set k4 v4
(error) MOVED 8455 192.168.226.128:6382
127.0.0.1:6381> exit

```

上面出现某些key不能设置的原因：
这是一个使用了哈希槽分区算法的redis集群，但是我们连接的是其中的某一台机器，如果设置的key不在它的槽位范围，那么会报错，无法设置值

**为了防止路由失效加参数-c并新增两个key**

```
redis-cli -p 6381 -c			##	-c: cluster
```

```
root@vm001:/data# redis-cli -p 6381 -c
127.0.0.1:6381> flushall
OK
127.0.0.1:6381> keys *
(empty array)
## 如果在6381其能力范围之外，则会跳转到集群中另外的机器6383上，以下同理
127.0.0.1:6381> set k1 v1
-> Redirected to slot [12706] located at 192.168.226.128:6383
OK
## 同上理
192.168.226.128:6383> set k2 v2
-> Redirected to slot [449] located at 192.168.226.128:6381
OK
192.168.226.128:6381> set k3 v3
OK
## 同上理
192.168.226.128:6381> set k4 v4
-> Redirected to slot [8455] located at 192.168.226.128:6382
OK
192.168.226.128:6382> 

```

查看集群信息

**redis-cli --cluster check 192.168.226.128:6381**

通过任何一台机器都可以查询，可以看到：
----a. 目前机器有3主3从，以及相应的映射关系
----b. 而且集群目前储存了5个key

```
root@vm001:/data# redis-cli --cluster check 192.168.226.128:6381
192.168.226.128:6381 (94714f89...) -> 2 keys | 5461 slots | 1 slaves.
192.168.226.128:6382 (aa89bba7...) -> 1 keys | 5462 slots | 1 slaves.
192.168.226.128:6383 (56f6c3cc...) -> 2 keys | 5461 slots | 1 slaves.
[OK] 5 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.226.128:6381)
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 8 主从扩容 – 四主四从或更多

#### 8.1 新建6387、6388两个节点

```
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis --cluster-enabled yes --appendonly yes --port 6387

docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis --cluster-enabled yes --appendonly yes --port 6388

docker ps
# 结果显示8个节点

```

#### 8.2 进入6387容器实例内部

```
docker exec -it redis-node-7 /bin/bash
```

#### 8.3 将新增的节点(空槽号)6387作为master节点加入集群

```
# redis-cli --cluster add-node 自己实际IP地址:6387 自己实际IP地址:6381
# 6387 就是将要作为master新增节点
# 6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群
redis-cli --cluster add-node 192.168.226.128:6387 192.168.226.128:6381

```

#### 8.4 检查集群情况第1次

```
redis-cli --cluster check 真实ip地址:6381
redis-cli --cluster check 192.168.226.128:6381
```

```
############ 检查集群情况
root@vm001:/data# redis-cli --cluster check 192.168.226.128:6381
192.168.226.128:6381 (94714f89...) -> 2 keys | 5461 slots | 1 slaves.
192.168.226.128:6382 (aa89bba7...) -> 1 keys | 5462 slots | 1 slaves.
############## 新增 redis server 没有分配到槽
192.168.226.128:6387 (cd58f466...) -> 0 keys | 0 slots | 0 slaves.
192.168.226.128:6383 (56f6c3cc...) -> 2 keys | 5461 slots | 1 slaves.
[OK] 5 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.226.128:6381)
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
############ 新增 redis master
M: cd58f46691ae515f63739726b0c70018cd765bc7 192.168.226.128:6387
   slots: (0 slots) master
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```

#### 8.5 重新分派槽号

```
# redis-cli --cluster reshard IP地址:端口号
redis-cli --cluster reshard 192.168.226.128:6381
How many slots do you want to move (from 1 to 16384)? 4094 //16,376个槽位由4个master节点均分
What is the receiving node ID? cd58f46691ae515f63739726b0c70018cd765bc7//即为8.4查出来的node7 master节点id
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: all


```

#### 8.6 检查集群情况第2次

```
# redis-cli --cluster check 真实ip地址:6381
root@vm001:/data# redis-cli --cluster check 192.168.226.128:6381
192.168.226.128:6381 (94714f89...) -> 1 keys | 4096 slots | 1 slaves.
192.168.226.128:6382 (aa89bba7...) -> 1 keys | 4096 slots | 1 slaves.
192.168.226.128:6387 (cd58f466...) -> 1 keys | 4096 slots | 0 slaves.
192.168.226.128:6383 (56f6c3cc...) -> 2 keys | 4096 slots | 1 slaves.
[OK] 5 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.226.128:6381)
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
############### 新增 redis server 分配到槽号了 [0-1364],[5461-6826],[10923-12287] 
############### 每个其他节点都分部分槽号给到6387
M: cd58f46691ae515f63739726b0c70018cd765bc7 192.168.226.128:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```

#### 8.7 为主节点6387分配从节点6388

```
# 命令：redis-cli --cluster add-node ip:新slave端口 ip:新master端口 --cluster-slave --cluster-master-id 新主机节点ID
  redis-cli --cluster add-node 192.168.226.128:6388 192.168.226.128:6387 --cluster-slave --cluster-master-id cd58f46691ae515f63739726b0c70018cd765bc7 
##  cd58f46691ae515f63739726b0c70018cd765bc7 这个是6387的编号，按照自己实际情况
```

```
root@vm001:/data# redis-cli --cluster add-node 192.168.226.128:6388 192.168.226.128:6387 --cluster-slave --cluster-master-id cd58f46691ae515f63739726b0c70018cd765bc7 
>>> Adding node 192.168.226.128:6388 to cluster 192.168.226.128:6387
>>> Performing Cluster Check (using node 192.168.226.128:6387)
M: cd58f46691ae515f63739726b0c70018cd765bc7 192.168.226.128:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.226.128:6388 to make it join the cluster.
Waiting for the cluster to join

>>> Configure node as replica of 192.168.226.128:6387.
[OK] New node added correctly.
```

#### 8.8 检查集群情况第3次

```
redis-cli --cluster check 192.168.226.128:6381

root@vm001:/data# redis-cli --cluster check 192.168.226.128:6381
192.168.226.128:6381 (94714f89...) -> 1 keys | 4096 slots | 1 slaves.
192.168.226.128:6382 (aa89bba7...) -> 1 keys | 4096 slots | 1 slaves.
192.168.226.128:6387 (cd58f466...) -> 1 keys | 4096 slots | 1 slaves.
192.168.226.128:6383 (56f6c3cc...) -> 2 keys | 4096 slots | 1 slaves.
[OK] 5 keys in 4 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.226.128:6381)
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
M: cd58f46691ae515f63739726b0c70018cd765bc7 192.168.226.128:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
   1 additional replica(s)
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
S: 3e67538095c080957dbafbc596b8cb8a5f7a9702 192.168.226.128:6388
   slots: (0 slots) slave
   replicates cd58f46691ae515f63739726b0c70018cd765bc7
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 9 主从缩容

目的：6387和6388下线

#### 9.1  检查集群情况1获得6388的节点ID

```
redis-cli --cluster check 192.168.226.128:6381

S: 3e67538095c080957dbafbc596b8cb8a5f7a9702 192.168.226.128:6388
   slots: (0 slots) slave
   replicates cd58f46691ae515f63739726b0c70018cd765bc7

```

#### 9.2 从集群中将4号从节点6388删除（先删从节点）

```
## redis-cli --cluster del-node 192.168.226.128:6388 3e67538095c080957dbafbc596b8cb8a5f7a9702
root@vm001:/data# redis-cli --cluster del-node 192.168.226.128:6388 3e67538095c080957dbafbc596b8cb8a5f7a9702
>>> Removing node 3e67538095c080957dbafbc596b8cb8a5f7a9702 from cluster 192.168.226.128:6388
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.

root@vm001:/data# redis-cli --cluster check 192.168.226.128:6382
## 结果只有 7 台机器

```

#### 9.3 将6387的槽号清空，重新分配

```
本例将清出来的槽号都给6381，4096个槽位都指给6381，它变成了8192个槽位，相当于全部都给6381了，不然要输入3次，一锅端
root@vm001:/data# redis-cli --cluster reshard 192.168.226.128:6381
>>> Performing Cluster Check (using node 192.168.226.128:6381)
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
M: aa89bba715780e6bf30ecbb734580b9da68e394f 192.168.226.128:6382
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: 71d1bc72e53985d25570ccd1accea81c7989b008 192.168.226.128:6386
   slots: (0 slots) slave
   replicates 94714f899698acb6e3d3584474c8bbbc7677e882
M: cd58f46691ae515f63739726b0c70018cd765bc7 192.168.226.128:6387
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
S: e4c3904b51cfed5a8f9e6d2a4d83236a906fda33 192.168.226.128:6385
   slots: (0 slots) slave
   replicates 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2
S: 8bd2ff973fa218f9022262c0815a9e5bd3c192d4 192.168.226.128:6384
   slots: (0 slots) slave
   replicates aa89bba715780e6bf30ecbb734580b9da68e394f
M: 56f6c3cc46dc5fa3eac29ccfa01d755f346fa2c2 192.168.226.128:6383
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
######## 本次移动 6387 上面所有4096个槽
How many slots do you want to move (from 1 to 16384)? 4096
######## 移动的槽由 6381 接收
What is the receiving node ID? 94714f899698acb6e3d3584474c8bbbc7677e882
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
######## 输入 6387 的ID
Source node #1: cd58f46691ae515f63739726b0c70018cd765bc7
Source node #2: done

########## xxxxxxxxxxxxx 前面会刷新很多槽的信息
Do you want to proceed with the proposed reshard plan (yes/no)? yes
########## xxxxxxxxxxxxx 后面会刷新很多槽的信息

```

#### 9.4 检查集群情况第二次

```
root@vm001:/data# redis-cli --cluster check 192.168.226.128:6382
# 6381 的槽变成8192 个
M: 94714f899698acb6e3d3584474c8bbbc7677e882 192.168.226.128:6381
   slots:[0-6826],[10923-12287] (8192 slots) master
   1 additional replica(s)
# 6387 的槽变成0 个
M: cd58f46691ae515f63739726b0c70018cd765bc7 192.168.226.128:6387
   slots: (0 slots) master

```

#### 9.5 删除6387 服务

```
root@vm001:/data# redis-cli --cluster del-node 192.168.226.128:6387 cd58f46691ae515f63739726b0c70018cd765bc7
>>> Removing node cd58f46691ae515f63739726b0c70018cd765bc7 from cluster 192.168.226.128:6387
>>> Sending CLUSTER FORGET messages to the cluster...
>>> Sending CLUSTER RESET SOFT to the deleted node.
```

#### 9.6 检查集群情况第三次

```
root@vm001:/data# redis-cli --cluster check 192.168.226.128:6382
192.168.226.128:6382 (aa89bba7...) -> 1 keys | 4096 slots | 1 slaves.
192.168.226.128:6381 (94714f89...) -> 2 keys | 8192 slots | 1 slaves.
192.168.226.128:6383 (56f6c3cc...) -> 2 keys | 4096 slots | 1 slaves.
[OK] 5 keys in 3 masters.
## 剩下三主三从了
```

### 10 命令总结

1. **构建集群关系：**

   ```
   # 构建集群关系
   redis-cli --cluster create 192.168.226.128:6381 192.168.226.128:6382 192.168.226.128:6383 192.168.226.128:6384 192.168.226.128:6385 192.168.226.128:6386 --cluster-replicas 1
   ```

2. **检查集群情况**

   ```
   # redis-cli --cluster check 真实ip地址:6381
   root@vm001:/data# redis-cli --cluster check 192.168.226.128:6381
   ```

3. **添加节点**

   ```
   # redis-cli --cluster add-node 自己实际IP地址:6387 自己实际IP地址:6381
   # 6387 就是将要作为master新增节点
   # 6381 就是原来集群节点里面的领路人，相当于6387拜拜6381的码头从而找到组织加入集群
   redis-cli --cluster add-node 192.168.226.128:6387 192.168.226.128:6381
   
   ```

4. **分配槽号**

   ```
   # redis-cli --cluster reshard IP地址:端口号
   redis-cli --cluster reshard 192.168.226.128:6381
   ```

   

5. **删除节点**

   ```
   ## redis-cli --cluster del-node ip:从机端口 从机6388节点ID
   redis-cli --cluster del-node 192.168.226.128:6388 3e67538095c080957dbafbc596b8cb8a5f7a9702
   
   ```

   

---

> 作者: wang  
> URL: https://codingroam.github.io/post/docker%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BAredis%E9%9B%86%E7%BE%A4%E5%B9%B6%E6%A8%A1%E6%8B%9F%E6%89%A9%E7%BC%A9%E5%AE%B9/  

