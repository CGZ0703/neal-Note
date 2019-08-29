# Docker运行Hadoop

```text
$ docker run -it -v /home/hadoop01/build:/root/build -h hadoop01 --name hadoop01 centos /bin/bash
```

以centos镜像启动一个容器，容器名是master，主机名是master，并且将基于容器的centos系统的/root/build目录与本地/home/hadoop/build共享。

参数解释：

* -v 表示基于容器的centos系统的/root/build目录与本地/home/hadoop/build共享；这可以很方便将本地文件上传到Docker内部的centos系统； 
* -h 指定主机名为master 
* --name 指定容器名 
* /bin/bash 使用bash命令



