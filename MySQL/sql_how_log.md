## 一条SQL更新语句怎么样执行
创建表，主键ID和一个整型字段c
```shell
create table T(ID int primary key,c int);
```
更新语句会把缓存清空，缓存失效。<br>
更新会涉及日志模块 redo log(重做日志)和bin log(归档日志)<br>
#### redo log保证数据库发生异常重启，之前提交的记录都不会丢失 crash-safe
MySQL的WAL技术，Write Ahead Logging，先写日志，再写磁盘。<br>
InnoDB引擎会在适当的时候，将这个操作记录更新到磁盘里面。<br>
粉板共4GB，从头开始写，写到末尾又回到开头循环写。<br>

![redoLog](./img/redo_log.png)

write pos是当前记录的位置，一边写，一边后移，写到第三号文件末尾后，就会到0号文件开头。checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。两个指针循环，如果write pos追上checkpoint，表示粉板满了，不能再执行新的更新，停下来，先擦掉，把check point推进一下。<br>
#### binlog
redo log是InnoDB引擎特有的日志，Server层也有自己的日志，bin log<br>
binlog日志只能用于归档，而InnoDB是另一个公司以插件形式引入MySQL<br>
* redo log是InnoDB引擎特有的；binlog是MySQL的server层实现的，所有引擎都可以使用
* redo log是物理日志，记录修改情况；binlog是逻辑日志，记录的是这个语句的原始逻辑
* redo log是循环写，bin log是追加写

两阶段提交:
![两阶段提交](./img/bin_redo_log.png)

两阶段为让两个状态保持逻辑上的一致。<br>

在两阶段提交的不同瞬间，MySQL如果发生异常重启，是怎么保证数据完整性的？<br>


