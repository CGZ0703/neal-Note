# Docker运行Hadoop

```text
$ docker run -it -v /home/hadoop01/build:/root/build -h hadoop01 -v /etc/localtime:/etc/localtime:ro --name hadoop01 neal_zhao/hadoop_basic /bin/bash

-- /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

以centos镜像启动一个容器，容器名是master，主机名是master，并且将基于容器的centos系统的/root/build目录与本地/home/hadoop/build共享。

参数解释：

* -v 表示基于容器的centos系统的/root/build目录与本地/home/hadoop/build共享；这可以很方便将本地文件上传到Docker内部的centos系统； 
* -h 指定主机名为master 
* --name 指定容器名 
* /bin/bash 使用bash命令


docker commit -m="java_base" --author="neal_zhao" 8916871c4954 neal_zhao/java_base:v1

docker run --name hadoop01 --hostname hadoop01 -d -P -p 50070:50070 -p 8088:8088 neal_zhao/hadoop-base:v1.0.0

docker run --name hadoop02 --hostname hadoop02 -d -P neal_zhao/hadoop-base:v1.0.0

docker run --name hadoop03 --hostname hadoop03 -d -P neal_zhao/hadoop-base:v1.0.0