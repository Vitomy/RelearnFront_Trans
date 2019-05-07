### order by 怎么工作的
```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```
SQL语句可以这么写<br>
```mysql
select city,name,age from t where city='杭州' order by name limit 1000  ;
```
这个order by怎么工作的呢？
#### 全字段排序
在city字段加上索引。Extra字段中使用“Using fliesort”表示的就是需要排序，MySQL会给每个线程分配一块内存用于排序，称为sort_buffer<br>
找出所有符合条件数据，进行快速排序。按照排序结果取前1000行返回给客户端。<br>
排序需要的内存和参数sort_buffer_size，就是MySQL为排序开辟的内存(sort_buffer)的大小。<br>
外部排序就是使用归并排序。  <br>
#### rowid排序
如果，MySQL认为排序的单行长度太大会怎么做呢？<br>
修改参数，让MySQL采用另外一种算法
```mysql
SET max_length_for_sort_data = 16;
```
专门控制用于排序的行数据的长度的一个参数。如果单行长度超过这个值，MySQL就认为单行太大，要换一个算法。<br>
rowid 需要从主表中重新再查找一次。<br>
通过索引可以直接取到对应的值，直接返回<br>
通过建立联合索引
```mysql
alter table t add index city_user(city, name);
```
通过建立覆盖索引
```mysql
alter table t add index city_user_age(city, name, age);
```
但是建立索引是有代价的，所以需要权衡考虑。<br>
```mysql
mysql> select * from t where city in ('杭州'," 苏州 ") order by name limit 100;
```
SQL查询语句这么写，执行过程中，有了联合索引city_name(city,name)这个联合索引。需要排序，因为已经可以在索引上是有序的。

