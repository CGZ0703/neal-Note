# Docker容器

## 容器的管理

### 创建

`docker create`:创建容器，处于停止状态。 

```text
$ docker create centos:latest
```

* * `centos:latest`:centos容器：最新版本\(也可以指定具体的版本号\)。
  * 本地有就使用本地镜像，没有则从远程镜像库拉取。
  * 创建成功后会返回一个容器的ID。
* `docker run`:创建并启动容器。

#### **交互型容器：**

运行在前台，容器中使用exit命令或者调用docker stop、docker kill命令，容器停止。

```text
# docker run -it --rm [--name=docker_run] centos bash
...
[root@e59b62752b03 /]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

docker run 就是运行容器的命令，说明一下上面用到的参数。

* -it：这是两个参数，一个是 -i：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。 
  * -i:打开容器的标准输入。
  * -t:告诉docker为容器建立一个命令行终端。 
* --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
* --name:指定容器名称，可以不填\(随机\)，建议根据具体使用功能命名，便于管理。
* centos:这是指用centos 镜像为基础来启动容器。 
* bash:放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash。

进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了 cat /etc/os-release，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是 CentOS Linux 系统。 最后我们通过 exit 退出了这个容器。 



在宿主机可以通过以下命令来便捷的查看镜像、容器、数据卷所占用的空间。 

```text
[root@hadoop01 ~]# docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              2                   1                   201.9MB             201.9MB (99%)
Containers          2                   0                   0B                  0B
Local Volumes       0                   0                   0B                  0B
Build Cache         0                   0                   0B                  0B
```



#### **后台型容器：**

运行在后台，创建后与终端无关，只有调用docker stop、docker kill命令才能使容器停止。

```text
# docker run --name=docker_run_d -d centos /bin/bash -c "while true; do echo hello world; sleep 1; done"
```

* d:使用-d参数，使容器在后台运行。 
* c: 通过-c可以调整容器的CPU优先级。默认情况下，所有的容器拥有相同的CPU优先级和CPU调度周期，但你可以通过Docker来通知内核给予某个或某几个容器更多的CPU计算周期。比如，我们使用-c或者–cpu-shares =0启动了C0、C1、C2三个容器，使用-c/–cpu-shares=512启动了C3容器。这时，C0、C1、C2可以100%的使用CPU资源（1024），但C3只能使用50%的CPU资源（512）。如果这个主机的操作系统是时序调度类型的，每个CPU时间片是100微秒，那么C0、C1、C2将完全使用掉这100微秒，而C3只能使用50微秒。
*  -c后的命令是循环，从而保持容器的运行。 

### 查看

* docker ps  : 查看当前运行的容器
* docker ps -a  : 查看所有容器，包括停止的。
* docker ps -l   : 查看最新创建的容器，只列出最后创建的。
* docker ps -n=2  : -n=x选项，会列出最后创建的x个容器。

#### 标题含义：

* CONTAINER ID:容器的唯一表示ID。
* IMAGE:创建容器时使用的镜像。 
* COMMAND:容器最后运行的命令。 
* CREATED:创建容器的时间。 
* STATUS:容器状态。 
* PORTS:对外开放的端口。 
* NAMES:容器名。可以和容器ID一样唯一标识容器，同一台宿主机上不允许有同名容器存在，否则会冲突。 

### 启动

通过docker start来启动之前已经停止的docker\_run镜像。

* 容器名：docker start docker\_run，或者ID：docker start 43e3fef2266c。 
* -restart\(自动重启\)：默认情况下容器是不重启的，–restart标志会检查容器的退出码来决定容器是否重启容器。 

```text
docker run --restart=always --name docker_restart -d centos /bin/sh -c "while true;do echo hello world; sleep;done": 
```

* `--restart=always`:不管容器的返回码是什么，都会重启容器。 
* `--restart=on-failure:5`:当容器的返回值是非0时才会重启容器。5是可选的重启次数。

**使用命令运行容器**

```text
$ docker run -it -v /home/build:/root/build -h master --name master centos /bin/bash
```

以centos镜像启动一个容器，容器名是master，主机名是master，并且将基于容器的centos系统的/root/build目录与本地/home/hadoop/build共享。

参数解释：

* -v 表示基于容器的centos系统的/root/build目录与本地/home/build共享；这可以很方便将本地文件上传到Docker内部的centos系统； （做了个目录映射
* -h 指定主机名为master 
* --name 指定容器名 
* /bin/bash 使用bash命令

### 停止

* `docker stop [NAME]/[CONTAINER ID]`:将容器退出。
* `docker kill [NAME]/[CONTAINER ID]`:强制停止一个容器。

### 删除

容器终止后，在需要的时候可以重新启动，确定不需要了，可以进行删除操作。

* `docker rm [NAME]/[CONTAINER ID]`:不能够删除一个正在运行的容器，会报错。需要先停止容器。  

**一次性删除**：docker本身没有提供一次性删除操作，但是可以使用如下命令实现：

* `$ docker rm 'docker ps -a -q'`：-a标志列出所有容器，-q标志只列出容器的ID，然后传递给rm命令，依次删除容器。

## 容器的使用

### 依附容器 

依附操作attach通常用在由docker start或者docker restart启动的交互型容器中。由于docker start启动的交互型容器并没有具体终端可以依附，而容器本身是可以接收用户交互的，这时就需要通过attach命令来将终端依附到容器上。

* docker start docker\_run：先启动docker\_run容器。 
* 启动后docker ps可以看到启动的容器，这时我们发现客户端显示的宿主机\(\[root@local ~\]\#\)。 
* 执行docker attach docker\_run，终端就已经依附到了容器上，ls显示的就是容器中的目录内容。 
* **注意**：后台型容器是无法依附终端的，因为它本身就不接受用户交互输入。 
* 另：使用“docker attach”命令进入container（容器）有一个缺点，那就是每次从container中退出到前台时，container也跟着退出了。要想退出container时，让container仍然在后台运行着，可以使用“docker exec -it”命令。（详情见下文 使用exec进入容器操作）

### 查看容器日志

首先创建一个不断输出一些内容的后台型容器，我命名为docker\_logs，是一个包含循环输出的自然数容器：

```text
$ docker run -d --name docker_logs centos /bin/bash -c "for((i=0;1;i++)); do echo $i; sleep 1; done;" 
```

* `$ docker logs -f docker_logs`
  * 此命令默认情况下是输出从容器启动到执行命令时的所有输出，但是之后的输出就不显示了，-f命令会实时显示日志。
* `$ docker logs -f --tail=5 docker_logs`
  * 此命令中–tail是控制logs输出的行数为最后5行。

### 查看容器进程

```text
$ docker top
```

可以查看容器中正在运行的进程。

### 查看容器信息

* `docker inspect [NAME]/[CONTAINER ID]`：用于查看容器的配置信息，包含容器名、环境变量、运行命令、主机配置、网络配置和数据卷配置等： 
* -f 或 --format格式化标志，可以查看指定部分的信息。
  * `docker inspect --format=''{{.State.Running}}' [NAME]/[CONTAINER ID]`
    * 查看容器的运行状态。 
  * `docker inspect --format='{{.NetworkSettings.IPAddress}} [NAME]/[CONTAINER ID]'`
    * 查看容器的IP地址。
  * `docker inspect --format '{{.Name}} {{.State.Running}}' [NAME]/[CONTAINER ID]`
    * 同时查看多个信息，查看容器名和运行状态。

### 容器内执行命令

在容器启动的时候，通常需要指定其需要执行的程序，然而有时候我们需要在容器运行之后中途启动另一个程序。从Docker1.3开始，我们可以用docker exec命令在容器中运行新的任务，它可以创建两种任务：后台型和交互型。

```text
$ docker exec -d docker_logs touch /etc/exec_new_file
```

-d:后台型，并在容器中创建一个文件。

#### 使用exec进入容器操作

每次使用这个命令进入container，当退出container后，container仍然在后台运行，命令使用方法如下

```text
$ docker exec -it docker_run /bin/bash
```

* docker\_run：要启动的container的名称
* /bin/bash：在container中启动一个bash shell

这样输入“exit”或者按键“Ctrl + C”退出container时，这个container仍然在后台运行。

## 容器的导入和导出 <a id="&#x5BB9;&#x5668;&#x7684;&#x5BFC;&#x5165;&#x548C;&#x5BFC;&#x51FA;"></a>

用户不仅可以把容器提交到公共服务器上，还可以将容器导出到本地文件系统中。同样，我们也可以将导出的容器重新导入到Docker运行环境中。导入：`import`,导出：`export`。

```text
docker export docker_logs > docker_logs_export.tar
```

把容器的文件系统以tar包的格式导出到标准输出。 

```text
cat docker_logs_export.tar | docker import - [res]:[tag]
```

把打包的容器导入为一个镜像，**res**代表镜像。**tag**代表标记。 

```text
docker import url res:tag
```

还可以通过一个url链接来导入网络上的容器。

