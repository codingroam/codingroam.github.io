# PostgreSQL查询出所有表的记录数

PostgreSQL查询出所有表的记录数
<!--more-->

项目所有用的数据库从SQLServer 换成PostgreSQL,项目中很多sql 是针对SQLServer 写的，所以不得不从新写SQL，项目中有一个功能是要统计出数据库的情况，包括所有表的记录数。对数据库不太熟悉，找了半天，大致还是要从系统表pg_class上入手。 
 
有关pg_class字段介绍：https://wizardforcel.gitbooks.io/postgresql-doc/content/714.html 
 
查询出pg_class表中的reltuples就是表的记录数： 
select relname as TABLE_NAME, reltuples as rowCounts from pg_class where relkind = 'r' order by rowCounts desc 
 
这样查出来的有一个问题，就是会把系统表的数据也查出来，这显然不是我想要的。怎么去掉系统表。 
 
可以查询Schema下的每张表的记录数 
select relname as TABLE_NAME, reltuples as rowCounts from pg_class where relkind = 'r' and relnamespace = (select oid from pg_namespace where nspname='public') order by rowCounts desc; 




---

> 作者: wang  
> URL: https://codingroam.github.io/post/postgresql%E6%9F%A5%E8%AF%A2%E5%87%BA%E6%89%80%E6%9C%89%E8%A1%A8%E7%9A%84%E8%AE%B0%E5%BD%95%E6%95%B0/  

