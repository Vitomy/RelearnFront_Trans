### 事务到底是隔离还是不隔离的？
事务遇到行锁，就会进入等待状态。`begin/start transaction`命令并不是一个事务的起点，执行到他们之后的第一个操作InnoDB表的语句，事务才真正启动，如果需要马上启动一个事务，使用`start transaction with consistent snapshot`。
#### "快照"在MVCC里是怎么工作的？
RR中，拍快照，基于整库。