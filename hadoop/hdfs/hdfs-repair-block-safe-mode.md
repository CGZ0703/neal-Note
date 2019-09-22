# 查看修复HDFS中丢失的块过多导致进入安全模式

我的测试机器有次意外关机没关hadoop，出现了以下日志：

```
  ......
Name node is in safe mode.

The reported blocks 632758 needs additional 5114 blocks to reach the threshold 0.9990 of total blocks 638510.

The number of live datanodes 3 has reached the minimum number 0.

Safe mode will be turned off automatically once the thresholds have been reached.
  ......
```
上面日志大意是，namenode正在安全模式中，接收到的datanode块数量(632758)不足总块数量(638510)的99.90%，活动数据块3的数量已经达到最小的数量0，安全模式将会打开，并在恢复到设置的阀值时自动关闭。

阀值的定义，在 hdfs 的配置文件 `hdfs-site.xml` 中有以下两个属性：
```xml
<property>
  <name>dfs.namenode.safemode.threshold-pct</name>
  <value>0.999f</value>
  <description>
    Specifies the percentage of blocks that should satisfy 
    the minimal replication requirement defined by dfs.namenode.replication.min.
    Values less than or equal to 0 mean not to wait for any particular
    percentage of blocks before exiting safemode.
    Values greater than 1 will make safe mode permanent.
  </description>
</property>

<property>
  <name>dfs.namenode.safemode.min.datanodes</name>
  <value>0</value>
  <description>
    Specifies the number of datanodes that must be considered alive
    before the name node exits safemode.
    Values less than or equal to 0 mean not to take the number of live
    datanodes into account when deciding whether to remain in safe mode
    during startup.
    Values greater than the number of datanodes in the cluster
    will make safe mode permanent.
  </description>
</property>
```

一般是因磁盘空间不足，内存不足，系统掉电等其他原因导致dataNode datablock丢失

## 安全模式

- 查看文件系统的文件
- 不可以改变文件系统的命名空间
  - 不可以创建文件夹
  - 不可以上传文件
  - 不可以删除文件

正常的HDFS系统Safemode是关闭的

HDFS刚开启namenode时会进入安全模式，HDFS的namenode等待dataNode向其发送块报告，当namenode统计总模块和发送过来的块报告中的统计信息达到99.999%的时候，表示不存在块的丢失，此时安全模式才会退出。

可以在Hadoop的WebUI界面中看到

### HDFS的进入与退出

```
Usage: hdfs dfsadmin
Note: Administrative commands can only be run as the HDFS superuser.
    [-report [-live] [-dead] [-decommissioning] [-enteringmaintenance] [-inmaintenance]]
    [-safemode <enter | leave | get | wait>]
......
```

使用`-safemode enter/leave`进入或离开安全模式

使用`-safemode get`查看安全模式


## 解决问题

执行命令退出安全模式：
```
hadoop dfsadmin -safemode leave
```

#### 1.执行健康检查，删除损坏掉的block。

```
hdfs fsck / -files
hdfs fsck / -delete
```
注意: 这种方式会出现数据丢失，损坏的block会被删掉

#### 2.排查问题

检测缺失块
```
hdfs fsck -list-corruptfileblocks
hdfs fsck / | egrep -v '^\.+$' | grep -v eplica
```

查看上面某一个文件的情况
```
hdfs fsck /path/to/corrupt/file -locations -blocks -files
```

接下来。。想办法修复这个文件。

根据集群的文件信息进行修复

