# mysql-deadlocks

在工作过程中偶尔会遇到死锁问题，虽然这种问题遇到的概率不大，但每次遇到的时候要想彻底弄懂其原理并找到解决方案却并不容易。这个项目收集了一些常见的 MySQL 死锁案例，大多数案例都来源于网络，并对其进行分类汇总，试图通过死锁日志分析出每种死锁的原因，还原出死锁现场。

我将这些死锁按事务执行的语句和正在等待或已持有的锁进行分类汇总：

|事务一语句|事务二语句|事务一等待锁|事务二等待锁|事务二持有锁|案例|
|---------|-----------|---------|-----------|-----------|---|
|insert|insert|lock_mode X insert intention|lock_mode X insert intention|lock_mode X|1|
|insert|insert|lock_mode X insert intention|lock_mode X insert intention|lock_mode S|2|
|delete|delete|lock_mode X|lock mode S|lock_mode X locks rec but not gap|4|
|delete|delete|lock_mode X|lock mode X|lock_mode X locks rec but not gap|6|
|delete|delete|lock_mode X locks rec but not gap|lock_mode X|lock_mode X|3|
|delete|delete|lock_mode X locks rec but not gap|lock mode X|lock_mode X locks rec but not gap|7|
|delete|delete|lock_mode X locks rec but not gap|lock_mode X locks rec but not gap|lock_mode X locks rec but not gap|8,9|
|delete|insert|lock_mode X|lock_mode X locks gap before rec insert intention|lock_mode X locks rec but not gap|5|
|delete|insert|lock_mode X|lock_mode X locks gap before rec insert intention|lock_mode S|10|

表中的语句虽然只列出了 delete 和 insert，但实际上绝大多数的 delete 语句和 update 或 select ... for update 加锁机制是一样的，所以为了避免重复，对于 update 语句就不在一起汇总了。

对于这种分类方法我感觉并不是很好，但也想不出什么其他更好的方案，如果你有更好的建议，欢迎讨论。另外，如果你有新的死锁案例，或者对某个死锁的解释有异议，欢迎给我提 Issue 或 PR。
