# Sqoop1.4.7+Hive2.3.6+Hadoop2.9.2遇到的问题

在自己的测试机器中搭建的服务是新版的，会遇到些之前没遇到的问题

## 1.

```
Sqoop:Import failed:java.lang.ClassNotFoundException:org.apache.hadoop.hive.conf.HiveConf
```

在profile中或sqoop-env.sh中加入：
```
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HIVE_HOME/lib/*
HIVE_CONF_DIR=$HIVE_HOME/conf
```

## 2.

```
java.lang.NoClassDefFoundError:com/fasterxml/jackson/databind/ObjectMapper

ava.lang.NoSuchMethodError: com.fasterxml.jackson.databind.ObjectMapper.readerFor(Ljava/lang/Class;)Lcom/fasterxml/jackson/databind/ObjectReader;
```

解决办法：
```
将$SQOOP_HOME/lib/jackson*.jar文件bak，

再把$HIVE_HOME/lib/jackson*.jar拷贝至 $SQOOP_HOME/lib目录中。
```

## 3.

```
main ERROR Could not register mbeans java.security.AccessControlException: access denied ("javax.management.MBeanTrustPermission" "register")

```

在$JAVA_HOME/jre/lib/security/java.policy中添加：
```
permission javax.management.MBeanTrustPermission "register"
```

