# SQL优化

## 为什么会有SQL优化

SQL优化就是来解决由于SQL语句编写导致性能低，执行时间长，索引失效，服务器参数不合理等问题

## 关于SQL优化

SQL优化器有时会干扰人为的SQL优化，因此这就是学SQL优化的理由，可以通过explain来模拟SQL优化的过程，可以让开发人员知道自己写的SQL是否管用

## Mysql服务器架构分层

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713175822312.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

如上图，Mysql服务器的分层大致可以分为三层

1. 连接层：处理一些数据库连接和认证授权等
2. 服务层：服务层有各种命令接口（所以的内置函数），所有的跨存储引擎的功能也在这一层实现（存储过程，触发器，视图等等），并且可以对客户端命令进行解析和优化
3. 存储引擎层：存储引擎负责数据的存储和提取，常见的存储引擎有InnoDB，MyIASM等等

## Mysql常用的存储引擎

下图是所以的存储引擎，Mysql对这些存储引擎除了fedrated都支持，InnoDB是默认的存储引擎

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713180530253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

在基础的已经介绍了很多关于存储引擎，下面就简单的说说InnoDb和MyISAM

可以通过查询系统表来查看当前使用的存储引擎是哪一个

```SQl
show variables like '%storage_engine%';
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713181340555.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

### InnoDB

InnoDB的思想是事务优先

对事务有很好的支持，并且锁的级别是行锁，因此比较适合高并发操作，但是由于是行锁，执行速度会慢

### MyISAM

MyISAM的思想是性能优先

MyISAM的性能远高于InnoDB，它不支持事务和行级锁，支持表锁，因此性能和安全不可得兼

### 切换存储引擎

可以在创建表的末尾切换存储引擎，自增的长度，默认的字符集编码等等

```SQl
create table tb(
    id int(5) auto_increment ,
    name varchar(6),
    primary key(id)
)engine=MyISAM AUTO_INCREMENT=1 default charset=utf8;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713182806891.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)





## 索引

[一篇不错的博客](https://www.cnblogs.com/annsshadow/p/5037667.html)



![1563017872969](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1563017872969.png)

如上图所示，B+树的所以数据都在叶子节点，所以每个数字的查询次数都是相同的，上边的蓝色块是为了提供临界，共黄色指针选择

### 索引的类型和创建索引

索引主要分为以下几类

* 单值索引：就是单列作为索引

  create index index_name on table(colum)

* 唯一索引（主键索引）：不能重复，但是主键索引和唯一索引的区别就是主键索引不能为null，但是唯一索引可以

  create unique index_name table(colum)

* 复合索引：多个列构成，按照列的先后顺序进行查找（相当于是多级目录，第一个查完，然后查第二个等）

  create index index_name on table(colum1,colum2...)

通过修改表属性来创建索引

alter table tableName add [index]|[unique index] index_name(colum)

> 注意：主键默认就是一个主键索引

### 删除索引

drop index index_name on tableName

### 查看索引

show index from tableName/G

show index from tableName;