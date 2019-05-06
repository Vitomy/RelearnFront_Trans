### 普通索引与唯一索引
在不同的业务场景下，应该选择普通索引还是唯一索引？<br>
change buffer。将更新操作记录在changebuffer，从磁盘读取数据的时候，在操作更新，减少读磁盘，语句的执行速度会得到明显提升。而且数据读入内存是需要占用buffer pool，所以这种方式还能避免占用内存，提高内存利用率。<br>
#### 什么条件下可以使用change buffer呢？
唯一索引，更新操作得先判断，先读磁盘，所以不需要change buffer。但是普通索引可以使用。<br>
change buffer的大小通过参数`innodb_change_buffer_max_size`来动态设置<br>
将数据从磁盘读入内存涉及随机IO的访问，是数据库里面成本最高的操作之一。change buffer因为减少了随机磁盘访问，所以对更新性能的提升是会很明显的。<br>

redo_log主要节省的是随机写磁盘的IO消耗(转成顺序写)，而change buffer主要节省的则是随机读磁盘的IO消耗<br>


```mysql
mysql> select 
  count(distinct left(email,4)）as L4,
  count(distinct left(email,5)）as L5,
  count(distinct left(email,6)）as L6,
  count(distinct left(email,7)）as L7,
from SUser;

```