## 全局锁和表锁：给表加个字段怎么有这么多阻碍
锁实现访问规则的重要数据结构。<br>
#### 全局锁、表级锁和行锁
全局锁，对整个数据库实例加锁。MySQL提供了一个加全局读锁的方法，命令是`Flush tables with read lock(FTWRL)`，当需要整个库处于只读状态的时候，可以使用这个命令。<br>
一致性读，引擎要支持这个隔离级别，不然就需要FTWRL，`set global readonly=true`，使得整个库处于readonly状态，不可写。<br>
表级锁，lock tables。表锁常用的处理并发的方式。MDL(metadata lock)。读写锁互斥。<br>
MDL锁，执行开始时申请，提交整个事务后再释放。<br>
single-transaction方法适用于所有的表使用事务引擎的库。InnoDB支持可重复读事务更新。<br>
如果需要变更一个热点表，理想的机制是拿写锁，拿不到也不要阻塞锁。
```
ALTER TABLE tbl_name NOWAIT add column ...
ALTER TABLE tbl_name WAIT N add column ... 
```
##### 行锁 减少行锁对性能的影响
引擎支持行锁。InnoDB支持行锁。如何通过减少锁冲突，提升业务并发度。A更新后B才能更新。<br>
直到Acommit才能执行B事务。⚠️如果事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。<br>
##### 死锁与死锁检测
* 等到超时检测，`innodb_lock_wait_timeout`(默认50s)来设置
* 发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数`innodb_deadlock_detect`设置为on，表示开启这个逻辑。

死锁检测 耗费大量CPU资源。确保这个业务一定不会出现死锁，可以临时把死锁检测关掉。<br>
控制并发度。中间件实现控制并发数量，进入引擎之前进行排队。<br>
也可以考虑多行设计。随机选取，多行随机选取更新。


