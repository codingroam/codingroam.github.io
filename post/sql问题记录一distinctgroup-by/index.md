# Sql问题记录一（distinct、group by）

mysql使用distinct问题记录
<!--more-->

distinct可以单字段去重，也可以多字段去重，

假如有表&nbsp; A,其有id，name,sex,addr,tel,createTime等如下字段，其中id是主键，故唯一

那么我们就可以对其他字段进行去重操作了。

1、按单字段去重，

&nbsp; &nbsp; &nbsp; select distinct name  from&nbsp; A

&nbsp; &nbsp; &nbsp; 这样，可以仅仅可以查询name这一列，且name没有相同数据。

2、多字段去重

&nbsp; &nbsp; &nbsp;select distinct name, sex, addr  from A

&nbsp; &nbsp; mysql会对dintinct所有字段当作整体去重，这样，只有当这三个字段全部相同时，才能成功去重，有一个

字段不同就不会过滤掉。

如果我们去重后想查看所有字段，想写出类似这样一个sql：

select distinct(name,sex,addr) name,sex,addr,createTime from A&nbsp;

这种写法是不被允许的，但是用group by来实现

select&nbsp; * from&nbsp; A&nbsp; where id&nbsp; in (select max(id) from&nbsp; A&nbsp; group by&nbsp; name,sex,addr)

子表是按group by 分组，也就相当于去重，去重之后，只找到一个最大的id,然后select *&nbsp; in 这些id中就可以

完成去重后查看所有的字段了。



---

> 作者: wang  
> URL: https://codingroam.github.io/post/sql%E9%97%AE%E9%A2%98%E8%AE%B0%E5%BD%95%E4%B8%80distinctgroup-by/  

