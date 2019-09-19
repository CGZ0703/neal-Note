# MySQL事务

### 事务

* 事务指逻辑上的一组操作，组成这组操作的各个单元，要么全成功，要么全不成功

#### 事务的使用

* mysql默认自动提交事务，每条语句都处在单独的事务中
* 手动设置开启事务 connention.setAutoCommit\(false\);// \(start transaction\|begin\)
* 执行sql语句
* 提交sql语句 connection.commit\(\);
* 执行过程中异常进行回滚  connection.rollback\(\)

#### 事务的隔离

* 脏读：一个事务读取了另一个事务未提交的数据
* 不可重复读：在一个事务内读取表中的某一行数据，多次读取结果不同。一个事务读取到了另一个事务提交后的数据\(update\)
* 虚读：是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致\(insert\)

#### 隔离级别

* 1、READ UNCOMMITTED:脏读、不可重复读、虚读都有可能发生。
* 2、READ COMMITTED:避免脏读。不可重复读、虚读都有可能发生。\(Oracle默认\)
* 3、REPEATABLE READ:避免脏读、不可重复读。虚读有可能发生。\(MySQL默认\)
* 4、SERIALIZABLE:脏读、不可重复读、虚读都避免发生。
* 级别越高，性能越低，数据越安全
* mysql中
  * 查看当前事务 SELECT @@TX\_ISOLATION
  * 更改当前事务的隔离级别 SET TRANSACTION ISOLATION LEVEL 级别
  * 设置隔离级别必须在事务之前
* 手动设置隔离级别：connection.set\(Connection.TRANSACTION\_SERIALIZABLE\)

#### 事务的特性

* 原子性：事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
* 一致性：事务必须使数据库从一个一致性状态变换到另一个一致性状态，例如：转账前和转账后总金额不变。
* 隔离性：事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
* 持久性：一个事务一旦被提交，它对数据库中的数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响。

| 时间 | 线程1 | 线程2 | 说明 |
| :--- | :--- | :--- | :--- |
| t1 | begin; |  |  |
| t3 | select \* from account where name='zs'; 结果1000块 |  |  |
| t4 |  | begin; |  |
| t5 | select \* from account where name='zs'; 结果1100块 |  | 读到了另一个线程未提交事务 的数据。赃读发生了 |
| t6 |  | commit; |  |
| t7 | select \* from account where name='zs'; 结果1100块 |  | 读到了另一个线程提交事务的 update数据。不可重复读发生了 |
| t8 | select \* from account; 查到4条数据 |  |  |
| t9 | select \* from account; 查到4条数据 |  | 读到了另一个线程自动提交事务的 insert语句数据。虚读发生了 |
| t10 | commit; |  |  |

