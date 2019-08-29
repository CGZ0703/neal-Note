# Docker

### **安装docker**

[doc.docker](https://docs.docker.com/install/)

建立docker组

默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

```text
建立 docker 组：
$ sudo groupadd docker
将当前用户加入 docker 组：
$ sudo usermod -aG docker $USER
退出当前终端并重新登录，进行如下测试。
```

### 从 Docker 镜像仓库获取镜像的命令是 docker pull。其命令格式为： 

```text
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

可以直接使用docker pull centos:7命令安装镜像

下载好之后，使用docker image ls查看拥有的镜像：

```diff
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        7 months ago        1.84kB
```

这是我们之前使用docker run hello-world命令下载的镜像。

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

有了镜像后，我们就能够以这个镜像为基础启动并运行一个容器。

### 测试是否是否安装成功

```text
$ docker run hello-world
```

如果有如下输出，那么就安装成功

```text

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### Recap and cheat sheet <a id="recap-and-cheat-sheet"></a>

```text
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

### DockerFile

These portable images are defined by something called a `Dockerfile`.

Create an empty directory on your local machine. Change directories \(`cd`\) into the new directory, create a file called `Dockerfile`, copy-and-paste the following content into that file, and save it. Take note of the comments that explain each statement in your new Dockerfile.

```text
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

This `Dockerfile` refers to a couple of files we haven’t created yet, namely`app.py` and `requirements.txt`. Let’s create those next.

> 太懒了 以上摘自官方文档  嘻嘻

