# MySQL优化  innodb_flush_log_at_trx_commit

学习过程中，为方便测试写了个存储过程生成数据，在自己的服务器上运行存储过程发现生成数据特别慢，

开始以为是内存的原因，后来经过排查跟内存没什么关系

修改很多参数，无果

> bulk_insert_buffer_size=100M

后来考虑过存储过程写法的原因，变量，时间，数据类型等。

> 改写所有 insert into 语句为 insert delayed into
> 这个insert delayed不同之处在于：立即返回结果，后台进行处理插入。

找了很久原因，发现只是多条插入语句执行很慢。（用shell脚本生成的10000条`insert into`语句执行也是很慢）

上网查阅多方资料，发现是mysql的安全设置

- innodb_flush_log_at_trx_commit = 0，Innodb 中的Log Thread 没隔1 秒钟会将logbuffer中的数据写入到文件，同时还会通知文件系统进行文件同步的flush操作，保证数据确实已经写入到磁盘上面的物理文件。但是，每次事务的结束（commit 或者是rollback）并不会触发LogThread 将log buffer 中的数据写入文件。所以，当设置为0 的时候，当MySQL Crash 和OS Crash或者主机断电之后，最极端的情况是丢失1 秒时间的数据变更。
- innodb_flush_log_at_trx_commit = 1，这也是Innodb的默认设置。我们每次事务的结束都会触发Log Thread 将log buffer中的数据写入文件并通知文件系统同步文件。这个设置是最安全的设置，能够保证不论是MySQL Crash 还是OS Crash或者是主机断电都不会丢失任何已经提交的数据。
- innodb_flush_log_at_trx_commit = 2，当我们设置为2 的时候，Log Thread会在我们每次事务结束的时候将数据写入事务日志，但是这里的写入仅仅是调用了文件系统的文件写入操作。而我们的文件系统都是有缓存机制的，所以LogThread的这个写入并不能保证内容真的已经写入到物理磁盘上面完成持久化的动作。文件系统什么时候会将缓存中的这个数据同步到物理磁盘文件LogThread 就完全不知道了。所以，当设置为2 的时候，MySQL Crash 并不会造成数据的丢失，但是OS Crash或者是主机断电后可能丢失的数据量就完全控制在文件系统上了。


1. 普通硬盘情况下，innodb_flush_log_at_trx_commit参数分别是1和2的情况，两者差别巨大，差异几乎到了9倍。
2. 在做了raid的情况下，innodb_flush_log_at_trx_commit参数分别是1和2的情况，两者有10%左右的差异。
3. 硬盘的IO性能对性能的影响是巨大的。

