# HBase安装配置

## HBase与Hadoop的版本选择

X : not supported 已知功能不完整或者存在CVE，因此放弃了支持
Y : supported 经过测试全功能支持
NT : Not tested 未经过测试


### Java support by release line

| HBase Version | JDK 7 | JDK 8 | JDK 9+ |
| --- | --- | --- | --- |
| 2.1+ | X | Y | NT |
| 1.3+ | Y | Y | NT |


### Hadoop version support matrix

|  | HBase-1.3.x | HBase-1.4.x | HBase-1.5.x | HBase-2.1.x | HBase-2.2.x |
| --- | --- | --- | --- | --- | --- |
| Hadoop-2.4.x | Y | X | X | X | X |
| Hadoop-2.5.x | Y | X | X | X | X |
| Hadoop-2.6.0 | X | X | X | X | X |
| Hadoop-2.6.1 | Y | X | X | X | X |
| Hadoop-2.7.0 | X | X | X | X | X |
| Hadoop-2.7.1+ | Y | Y | X | Y | X |
| Hadoop-2.8.\[0-2\] | X | X | X | X | X |
| Hadoop-2.8.\[3-4\] | NT | NT | X | Y | X |
| Hadoop-2.8.5+ | NT | NT | Y | Y | Y |
| Hadoop-2.9.\[0-1\] | X | X | X | X | X |
| Hadoop-2.9.2 | NT | NT | Y | NT | Y |
| Hadoop-3.0.\[0-2\] | X | X | X | X | X |
| Hadoop-3.0.3+ | X | X | X | Y | X |
| Hadoop-3.1.0 | X | X | X | X | X |
| Hadoop-3.1.1+ | X | X | X | Y | Y |


## 安装

下载并解压

配置环境变量：

```bash
export HBASE_HOME=/opt/framework/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

hbase-env.sh
```bash
export JAVA_HOME=/opt/framework/jdk

# Tell HBase whether it should manage it's own instance of Zookeeper or not.
export HBASE_MANAGES_ZK=true
```

### 单机节点配置

hbase-site.xml
```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///opt/framework/hbase/hbasedata</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/framework/hbase/zkdata</value>
  </property>
</configuration>
```

启动Hbase

```bash
$ start-hbase.sh
$ jps
...
4593 HMaster
...
```

### 伪分布式
hbase-site.xml
```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop:9000/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/framework/hbase/zkdata</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
</configuration>
```

启动Hbase

```bash
$ start-hbase.sh
$ jps
...
11283 HMaster
11411 HRegionServer
11212 HQuorumPeer
...
```


### 完全分布式

HBASE是一个分布式系统
其中有一个管理角色：  HMaster(一般2台，一台active，一台backup)
其他的数据节点角色：  HRegionServer(很多台，看数据容量)

实际上，需要一个完全分布式的配置来全面测试HBase，并在实际场景中使用它。在分布式配置中，集群包含多个节点，每个节点运行一个或多个HBase守护进程。这些包括主实例和备份主实例、多个ZooKeeper节点和多个RegionServer节点。

**角色分配**
```
hadoop1:namenode hmaster
hadoop2:datanode regionserver zookeeper backup-hmaster
hadoop3:datanode regionserver zookeeper
hadoop4:datanode regionserver zookeeper
```

hbase-env.sh
```bash
export JAVA_HOME=/usr/local/java/jdk1.8.0_45

# Tell HBase whether it should manage it's own instance of Zookeeper or not.
export HBASE_MANAGES_ZK=false
```

hbase-site.xml
```xml
<configuration>
  <!-- 指定hbase在HDFS上存储的路径 -->
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://hadoop1:9000/hbase</value>
  </property>
  <!-- 指定hbase是分布式的 -->
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <!-- 指定zk的地址，多个用“,”分割 -->
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>hadoop2:2181,hadoop3:2181,hadoop4:2181</value>
  </property>
</configuration>
```

regionservers
```
hadoop2
hadoop3
hadoop4
```

在hbase的conf目录下创建backup-masters文件，并添加主机名hadoop2

baskup-masters
```
hadoop2
```

分发至所有节点

```bash
for i in 2 3 4; do scp -r hbase hadoop${i}:/opt/framework/
```

启动Hbase 
先启动zookeerper 
再启动hbase：使用`start-hbase.sh`

