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



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191130144632400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

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

## SQL优化实践

### SQL解析过程

sql编写过程：

select dinstinct  .. from  ..join  ..on ..where ..group by  ..having  ..order by ...limit ...

from .. on  .. join ..where  ..group by ...having ...select  distinct ...order by limit ..

### 初始化

课程表，教师表，教师证表，并给里面插入数据（先开始初始化没有主键的和有逐渐的explain是不一样的）

```SQl
 CREATE TABLE `course` (
  `id` int(4) NOT NULL,
  `name` varchar(12) DEFAULT NULL,
  `tid` int(4) DEFAULT NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

```SQL
 CREATE TABLE `teacher` (
  `id` int(4) NOT NULL,
  `name` varchar(10) DEFAULT NULL,
  `tcid` int(4) DEFAULT NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

```SQL
 CREATE TABLE `card` (
  `id` int(4) NOT NULL,
  `descr` varchar(30) DEFAULT NULL,
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

### 实践

1. 查询学生证编号为2或者所授课程编号为1的教师信息

   需要三表查询，先把course和teacher，teacher和card链接起来，然后写条件

   ```SQL
    explain select t.* from teacher t,course cou,card c 
    where t.id=cou.tid and t.tcid = c.id and(c.id=2 or cou.id=1);
   ```

2. 查询教授Mysql老师的描述

   多表连接查询

   ```SQL
   explain select c.descr from teacher t,course cou,card c where t.id = cou.tid and t.tcid = c.id and cou.name = 'Mysql';
   ```

   嵌套子查询

   ```SQL
   
   ```

   连接+子查询

   ```SQL
   
   ```

**注意：研究索引的前提要有索引**

### explain

> 在Mysql8.0会有些不同

#### id

id标志的就是SELECT的查询序列号：id越大，先查询。id相同，按表中数据个数从小到大查询。

1. 连接查询id是相同的，**id相同的情况下（并且不加任何约束包括主键约束）**，查找顺序从小到大排序

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071412015862.png)

   如上图，t数据条数最下，接下来是cou，接下来是c，如果给t加记录条数，顺序会变化，原因就是**在计算笛卡尔积的时候，存储器更倾向于存储较小的数**，

   例如：2 * 3 * 4 =6 * 4=24 

   ​            4 * 3 * 2 =12 * 2 =24

   虽然结果相同但是，中间值一个是6一个是12

2. 嵌套查询id是不同的，在id不同的情况下，先查id最大的，就如SQL的嵌套子查询，先得出最里层的，才能逐渐往外扩展查询。

   > 多表查询的时候id都相等的，嵌套子查询的时候id是不等的，这就可以看出id的含义和id的关系

3. 多表+嵌套，当然就会出先多表的id是相同的，嵌套的id是不同的，如下图所示

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190714121238695.png)

   id值从大的开始执行，id相同就按顺序执行

#### select_type

此列表示查询的类型

* primary：子查询中的最外层，也就是要查的目的对象

* subquery：子查询中的内层，除过最外层的所有层

* simple：简单查询（不包含子查询或者UNION）

* derived：衍生查询（在查询中用到了临时表）

  * from后跟一个子查询，查出来的是一个表（本来不存在这个表，这个表是临时从子查询中查出来的），table表示是那个表，derived2表示这个衍生表是从id为2的表中来的

    ![1563081093833](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1563081093833.png)

  * 在子查询里，如果有table1 union table2，则table1 就是derived

* union

* union result：显示谁和谁的union关系

#### table

这个值可能是表名、表的别名或者一个为查询产生临时表的标识符，如派生表、子查询或集合。

#### type

官方文档称之为“访问类型”，常见的就那么几种，按照效率最好到最差依次排列依次是

**null > system/const > eq_ref > ref > ref_or_null  >index_merge >  range > index >  all** 

* NULL：在优化过程中就已得到结果，不用再访问表或索引。

* system/const：在整个查询过程中这个表最多只会有一条匹配的行，比如主键 id=1 就肯定只有一行

  表最多有一个匹配行，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以一定是用到primary key 或者unique 情况下才会是const。没有索引就是system。

* eq_ref：每个索引键的查询返回唯一行数据（有且只能有一个，不能多，不能少，也不能为0）

  简单来说，就是多表连接中使用primary key或者 unique key作为关联条件，而且连接的某项个数要相同

* ref：非唯一性索引，对于每个索引键的查询返回的数据不唯一（0，多个）

* range：只检索给定范围的行

  一般来说就是where后是大于，小于等等，或者是between，in（in有时候会失效，转化为ALL）等等

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715001419306.png)

* index ： **索引扫描**（查出来的结果仅仅是索引列），这个比all效率要好一点，只查表的一部分必然比查所有快。（只把索引列查找一遍，就相当于遍历B+树）

* all : 这个就是**全表扫描**了，一般这样的出现这样的SQL而且数据量比较大的话那么就需要进行优化了，要么是这条SQL没有用上索引，要么是没有建立合适的索引。

> 基本可能实现的最好情况应该是ref

#### possible_key

指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用（该查询可以利用的索引，如果没有任何索引显示 null）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191130145149266.png)
#### key

实际用到的索引

#### key_len

索引长度，经常用来判断复合索引是否完全被使用，一个char四个字节

测试：创建一张表，并加上索引

```SQL
 CREATE TABLE `len` (
  `name` char(20) DEFAULT NULL,
  KEY `i_len` (`name`)
)
```

```SQL
mysql> explain select * from len where name = 'null';
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+--------------------------+
| id | select_type | table | partitions | type | possible_keys | key   | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | len   | NULL       | ref  | i_len         | i_len | 81      | const |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+------+---------------+-------+---------+-------+------+----------+--------------------------+
1 row in set, 1 warning (0.01 sec)
```

**len的结果是81，如果将name修改为not null 结果是80，因为如果可以为空，mysql会留出一位作为占用。**

如果是复合索引的话，可以根据len的长度来判断到底执行到那一步，比如一个索引是（name1，name2），如果只查到第一个就查出来的话len就是80，如果查到第二个才查出来的话len就是160

如果是varchar的话，varchar（20）结果是83（一位是空标志位，两位是varchar标志位），

#### ref

就是在连接的时候用到**两个或多个索引相关联**ref就是与它关联的属性列，如果**索引的值是一个固定值，那就是const**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715105141707.png)

#### rows

估算的找到所需的记录所需要读取的行数

#### Extra

测试

```SQL
create table test01(
	a1 char(3),
    a2 char(3),
    a3 char(3),
    index idx_a1(a1),
    index idx_a2(a2),
    index idx_a3(a3)
);

explain select * from test01 where a1='' order by a1;

explain select * from test01 where a1='' order by a2;

drop index idx_a1 on test01;
drop index idx_a2 on test01;
drop index idx_a3 on test01;

//增加一个复合索引
alter table test01 add index idx_a1_a2_a3(a1,a2,a3); 

//跨列演示
explain select * from test01 where a1='' order by a3;

//using temporary
explain select * from test01 where a1 in('1','2','3') group by a1;

explain select * from test02 where a1 in('1','2','3') group by a2

```

##### using jion buffer

Mysql给加上了连接缓存，SQL写的太差了

##### using filesort

是因为在查找后对另外一个字段进行排序，造成了第二次排序。性能消耗较大，常见于order by

**针对于单索引**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715111856471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

**复合索引(不能跨列)**

跨列就是上边建立的inx_a1_a2_a3扫描a1后直接扫描a3，跳过了a2

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715112737743.png)

如果a1接下来order by a2就不会出现using filesort

![1563161330601](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1563161330601.png)

##### using temporary

用到了临时表，性能消耗比较大，一般出现在group by里(where 在 group by前执行)

已经有表了，但是这张表不适用于后边的操作，就和下图中查a1却根据a2分组

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715113936377.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

##### using index

说明性能提升了，索引覆盖。不需要读取表，从索引中就可以得到所以想要的字段。

索引覆盖就是使用到的列全在索引中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715115718512.png)

如果用到了索引覆盖会对key和possible key产生影响

如果没有where 索引只会出现在key里

如果有where有possible key和key

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715120541599.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

##### using where

需要回表查询，如果age是索引，使用age查信息，需要回原表，索引会有using where

##### impossible where

不可能成立的where条件

比如:select * from test01 where id=1 and id =2;不可能成立

### 优化器优化实例

test01表上有a1,a2,a3,a4复合索引

```SQL
select * from test01 where a1='' and a2 ='' and a3 ='';（推荐使用）
//两个的结果是相同的，虽然索引顺序不一致（这是优化过的，不一定每次都可以）
select * from test01 where a3='' and a2= '' and a1='';

//a1，a2不需要回表查询，a4需要回表查询（跨列了，导致索引失效，因此有using where，也可一十一key len校验）
select * from test01 where a1='' and a2='' and a4='' order by a3;

//这个就不能进行优化了，因为where和order by里的列不能连接在一起
select * from test01 where a1='' order by a3;

//这个就可以
select * from test01 where a1='' and a2='' order by a3,a4;
```



#### 优化意见

* 对于单索引，where那个字段就尽量order by那个字段
* 对于复合索引，where和order按照复合索引顺序来，不要跨列或无序使用

### 优化

#### 单表优化

数据准备

```SQL
create table book(

    bin int(4) primary key,
    name varchar(20) not null,
    authorid int(4) not null,
    publicid int(4) not null,
    typeid int(4) not null
);

insert into book values(1,'java',1,1,1);
insert into book values(2,'py',2,2,1);

```

查询authorid=1且typeid为1,2或3的bid并且按照typeid的顺序递减

```SQL
select bid from book where typeid in(1,2,3) and authorid = 1 order by typeid desc;
```

查看其查询计划，可以看到type是ALL，Extra是filesort，当然这不是我们想要的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715150832278.png)

按序加索引

alter table book add index idx_bta(bid,typeid,authorid);

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715153346931.png)

可以发现type变为index，extra 也有了Using index，但是filesort还存在，这就说明顺序是不对的，按照我们前面的编写顺序和执行顺序可以看出select 的是在后面的

因此可以优化为下面，bid可以查表得出，但是没有索引快

alter table book add index idxx_tab(typeid,authorid,bid);

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715153802834.png)

可以看到filesort果然消失了

注意：如果索引进行了升级替换就删掉原来索引防止数据干扰

```SQL
SQL drop index idx_bta on book;
```

根据前面所知道的index并不是一个很好的选择，我们需要进行改进，而且前面说到过**in有时候会导致索引失效**，因此尽量把in放在最后，避免影响其它的条件，一次我们需要改一下复合索引

```SQL
drop index idxx_tab on book;

alter table book add index idx_atb(authorid,typeid,bid);

select bid from book where authorid = 1 and typeid in(1,2,3)  order by typeid desc;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715155641897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

可以看出in是不稳定的因此放在where的最后面

个人认为可以使用between  and 可以使得type变为range

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715160201447.png)

但是in有时也会出现ref。

**疑问**：在Extra中同时出现Using where和Using index不会会有矛盾？

当然不会，using where 是需要回原表，而using index是不需要回原表，为什么会出现这种问题呢？

因为上述查询中(authorid,typeid,bid)，authorid在索引表中有，因此不需要回原表，typeid虽然也建立的索引，但是in使得typeid的索引失效，从而导致typeid回原表。

我们如果把查询条件中的in换位等号

```SQL
select bid from book where authorid = 1 and typeid =2 order by typeid desc;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715160920951.png)

可以发现就没有using where了

#### 多表优化

准备工作

```SQL
create table teacher2(

    tid int(4) primary key,
    cid int(4) not null
);

insert into teacher2 values(1,2);
insert into teacher2 values(2,1);
insert into teacher2 values(3,3);


create table course2(

    cid int(4),
    cname varchar(20)
);

insert into course2 values(1,'JAVA');
insert into course2 values(2,'Python');
insert into course2 values(3,'kotlin');

//通过一个左外连接查询	
select * from teacher2 t left outer join course2 c on t.cid=c.cid where c.cname ='JAVA';

```

查看执行计划

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715233208354.png)

如果此时要加索引，该往哪加，该怎么加？

**左外连接给左表加索引，右外连接给右表加索引（根据频繁程度）**

比如有一个小表10条数据，大表300条数据，**在where的时候把小表的条件放在前面**（根据计算机的空间性原理）

使用小表驱动大表（**索引建立在用的频繁的字段上**） 

在实际生活中一定是cid用的比较多，通过course的cid和教师表关联，因此给教师表的cid加索引

```SQL
alter table teacher2 add index idx_cid(cid);

//根据课程名进行检索也很经常用，如上边的例子，因此可以给课程名加索引
alter table course2 add index index_course2_name(cname);
```

接下来再查看执行计划

```SQL
select * from teacher2 t left outer join course2 c on t.cid=c.cid where c.cname ='JAVA';
```

先加了idx_cid索引后执行计划，发现变化

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715233439461.png)

再加了cname索引后再次查看执行计划

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715233408717.png)

三张表以及多张表和两张表是一样的

1. 小表驱动大表
2. 索引建立在经常查询的字段上

### 避免索引失效

怎么可以避免索引失效

1. 复合索引不要跨行，无序使用（最优左前缀）

2. 复合索引，尽量使用全索引匹配（就相当于多级目录）

3. 不要在索引上进行任何操作（计算，类型转换，函数）

   如select * from teacher t,course c where t.id*3=c.tid;

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190715235746442.png)

4. 复合索引写不等于（！=， <>）或者大于或is null,is not null就失效了，范围查询一般是本身有用，后面失效。

   最直接最有效的补救方法就是覆盖索引

5. like中尽量少用’%‘开头，否则会导致索引失效，如果必须的话可以采用索引覆盖，可以稍稍提高性能

6. 尽量不要进行显示或隐式的类型转换（底层进行类型转换，造成索引失效）

   如：select * from teacher where tname = '111';  和 select * from teacher where tname = 111;

7. 尽量不要使用or，会导致索引失效（甚至可以把左边的索引失效）

> 注意：对于复合索引，如果前面条件失效，后面全部失效，因此把容易失效的放在最后边,SQL优化是一个概率性的并不是绝对的。	

为什么是概率性的：因为SQL中有优化器会干扰我们的优化。

### SQL一些优化方法

1. exist和in

   ```SQL
   //将主查寻的结果放到子查询中进行校验，如果字查询中存在就保留数据
   select .. from table where exist(子查询) 
   
   //就相当于给teacher的tname字段做了映射
   select tname from teacher where exists(select * from teacher)
   
   //和exist差不多，将主查询字段和子查询中此字段相同的保留下来
   select .. from table where column in (子查询)
   ```

   如果主查询数据集大，就使用In，效率高。

   如果子查询数据集大就使用exist，效率高。

2. order by 优化

   using filesort 有两种排序算法：双路排序，单路排序（根据I/O的次数来划分）

   Mysql4.1前使用双路排序 ，需要扫描两次磁盘

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/201907161122238.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

   1. 从磁盘读取排序字段，对读取出来的字段在buffer（缓冲区）中排序
   2. 再次扫描其它字段

   Mysql4.1后为单路排序，只读取一次（全部字段）

   只进行一次读取，然后在buffer中排序。但是单路不仅仅是单路，如果数据量太大的话，buffer容量有限，因此就得进行多次的“分片操作”来完成（多次I/O）

   ```SQL
   //设置buffer的大小
   set max_length_for_sort_data = xx字节
   ```

   如果需要的空间大于设置的buffer，Mysql就会自动从单路切换为双路

   因此order by优化方法如下：

   1. 选择合适的buffer，调整多路和单路
   2. 避免使用select * ....  where         因为Mysql内部需要对*进行解析，并且 * 很难实现所以覆盖，因此原则就是用到那个字段写那个
   3. 复合索引不要跨列使用
   4. 尽量保持排序字段的升降一致性（order by a desc,c desc），全升或者全降。

## Mysql慢查询

是一种记录日志，用于记录Mysql中相应实践查过阈值的SQL（slow_query_log）

```SQL
//通过SQL查看是否开启慢查询
show variables like '%slow_query_log%';
```

### 开启和关闭慢查询

slow_query_log默认是ON，有两种方法可以进行设置

1. 暂时性开启/关闭

   set_global slow_query_log=1/0;

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716113901643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

2. 永久性开启/关闭

   /ect/my.cnf中配置，在mysqld下

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716124147193.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

3. 慢查询阈值

   show variables like '%long_query_time%';

4. 设置临时慢查询阈值

   set global long_query_time=5;

5. 永久慢查询阈值

   /ect/my.cnf追加

   在mysqld下

   long_query_time=xx;

查询慢查询条数

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716132152241.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

### 排查慢查询的方法

1. 使用配置的日志文件路径查看

2. 使用mysqldumpslow查看慢SQL，如果日志多的话，可以通过过滤条件定位出慢SQL

   mysqldumpslow --help

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716132714159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

## 海量数据分析

模拟并通过profiles分析海量数据

```SQL
create table dept(

    dno int(5) primary key default 0,
    dname varchar(30) default '',
    loc varchar(30) default ''
)engine=innodb default charset=utf8;

create table emp(

    eid int(10) primary key,
    ename varchar(20) not null default '',
    job varchar(20) not null default '',
    deptno int(5) not null default 0
)engine=innodb default charset=utf8;

set global log_bin_trust_function_creators=1;

//随机字符串函数
delimiter $
create function randstring(n int) returns varchar(255)
begin
        declare all_str varchar(255) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
        declare return_str varchar(255) default '';
        declare i int default 0;
        while i<n
        do
              set return_str = concat(return_str,substring(all_str,FLOOR(1+rand()*52),1));
              set i=i+1;
         end while;
         return return_str;
 end $

//中间有一段写错了，修改后删除重新执行了
DROP function IF EXISTS randstring;       
//产生随机整数

create function ran_num() returns int(5)
begin
      declare i int default 0;
      set i = floor(rand()*100);
      return i;
end$ $

//通过存储过程插入大量数据
//给emp表插
create procedure insert_emp(in eid_start int(10),in data_times int(10))
begin 
         declare i int default 0;
         set autocommit = 0;
         while i<data_times
         repeat
                  insert into emp values(eid_start,randstring(5),'job',ran_num());
				set eid_start = eid_start+1;
                set i = i+1;
				  until i = data_times
		  end repeat;
		  commit;
end $

//在编写时候出错，
DROP procedure IF EXISTS insert_emp; 

//给dept拆入数据
create procedure insert_dept(in dno_start int(10),in data_times int(10))
begin
		declare i int default 0;
		set autocommit = 0;
		repeat
			insert into dept values(dno_start+1,randstring(6),randstring(8));
			set i = i+1;
			until i = data_times
			end repeat;
			commit;
end $

delimiter ;

call insert_emp(1000,8000000);
call insert_dept(10,30);

```

1. 分析海量数据：profiles

   show profile ：打开它只有的语句都会被记录下来还有执行时间（不精确）

```SQL
//只有在打开后才会有记录
set profiling = on;
//查询profiles的状态
show variables like '%profiling%';
```

2. 精确分析：

   show profile all for query id          //id是profiles里面出现的id

3. 全局日志：记录开启后的全部SQL

   ```SQL
   show variables like ='%general_log%';
   
   //记录在系统表
   set global general_log=1;
   set global log_output='table';--将SQL记录在表中
   
   //记录在文件
   set global general_log=1;
   set global log_output='file';--将SQL记录在表中
   set global log_output_file = 'xxx';
   ```

   开启后记录会被记录在mysql.general_log表中

   ```sql
   select * from mysql.general_log;
   ```

   ## 锁机制

   锁按照方式可以分为

   * 读锁
   * 写锁

   锁按照粒度分

   * 行锁
   * 页锁
   * 表锁

   锁的粒度越小，开销就越大，表锁范围大，无死锁，容易冲突，行锁范围小，有死锁，并发强，不易发生冲突，

   关于锁的部分在Mysql笔记中有详细的分析。

   ```SQL
   //加锁语句
   lock table table1 read/write ,table2 ....
   
   //查看有没有加索
   show open tables;
   ```

### 表锁

使用MyISAM存储引擎创建tablelock表

如果是读锁的话，所有会话都可以进行读操作，都不能在此锁打开之前进行修改操作

如果是写锁，只能有一个会话进行写操作，其余会话就原地等待占用锁的会话的释放，只有释放后才能使用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716203914921.png)

Mysql表级锁的锁模式

MyISAM在执行select前，会自动的给涉及的所有的表加读锁

在执行更新操作之前（DML），会自动给涉及的表加写锁，如果多个操作同时操作一张表，只能有一个进去，其余的在外面等待。

### 行锁

行锁的执行过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071621220335.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

只要多个用于执行的不是同一个数据项，就不会出现这种状况

行锁的注意事项

1. 如果该字段没有索引，那么行锁就会转变为表锁

   值得注意的是，前面的那些能导致索引失效的原因也会间接的决定是行锁还是表锁

   例如：因为类型转换而导致索引失效

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716213637533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

> 行锁只能通过事务（commit）来释放锁，而表锁可以通过unlock tables，也可以通过commit。

2. 行锁的特殊情况：间隙锁

   间隙就是比如在一段连续的条件中，不存在的值，比如update ...where id>1 adn id<10 ,但是中间没有id为5，id为5就称为间隙

   **Mysql会自动给此类间隙加锁成为间隙锁。在这个修改语句为commit时，另一个用户是不能对它进行操作的。**



### 分析表锁定

#### 关闭自动提交的几种常用方式

1. set autocommit=0;
2. start transaction;
3. begin;

#### 表锁

```SQL
//查看加锁状态,0代表未加锁，1代表加锁
show open tables;
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716204139969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)



```SQL
//分析表锁定的程度
show status like 'table%';

Table_locks_immediate:当前能获取到的锁的个数
Table_locks_waited:当前需要等待的表锁数（锁数量不变，该数越大，说明竞争越大）
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716230705669.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

可以用此来分析该用那种存储引擎

一般如果Table_locks_immediate/Table_locks_waited>5000，建议采用InnoDB，支持更高的并发。

#### 行锁

```SQL
//在innodb下查看行锁
show status like '%innodb_row_lock%';

//当前等待锁的数量
Innodb_row_lock_current_waits
//从系统启动到现在等的总时长
Innodb_row_lock_time
//平均等待时长
Innodb_row_lock_time_avg
//最大等待时长
Innodb_row_lock_time_max
//从系统启动到现在一共等待的次数
Innodb_row_lock_waits
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071623054582.png)

> 给查询语句可以通过后边加一个for update加锁。

## 主从同步

远程访问需要授权设置

```SQL
GRANT ALL PRIVILES ON ...........
然后
FLASH PRIVILEGES;
```

原理图1：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190717004536660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

原理图2：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190717004433617.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

实现主从同步的原理（这个过程要由三个线程来完成）

1. master将改变的记录值记录在本地的二进制日志里（binary log）——该过程称为二进制日志事件
2. slave将master的binary log拷贝到自己的relay log（中继日志文件）中
3. 中继日志事件将数据读取到自己的数据库里

主从复制的和Reids中的主从复制类似

[一篇主从复制的博客](https://blog.csdn.net/darkangel1228/article/details/80004222)

