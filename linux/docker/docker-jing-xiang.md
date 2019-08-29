# Docker镜像

镜像是一个包含程序运行必要依赖环境和代码的只读文件，它采用分层的文件系统，将每一次改变以读写层的形式增加到原来的只读文件上。镜像是容器运行的基石。

* 其中，镜像的最底层必须是一个称为启动文件系统\(bootfs\)的镜像，用户不会与这一层直接打交道。bootfs的上层镜像就是我们熟知的根镜像。 
* 镜像的本质是磁盘上一系列文件的集合。

### 查看

```text
docker image [COMMAND]
```

REPOSITORY：仓库名称。

* \[namespace/centos\]：由命名空间和实际的仓库名称组成。当你再Docker Hub上注册一个账户时，账户名自动成为你的命名空间，该命名空间是用来区分Docke Hub上注册的不同用户或者组织的。 
  * \[centos\]：只有仓库名。属于顶级命名空间，只用于官方镜像。 
  * \[dl.dockerpool.com:5000\centos:7\]：指定URL路径的方式。适用于自己搭建的Hub或者第三方Hub上获取镜像。 
* TAG：用于区分同一个仓库中的不同镜像。 
* IMAGE ID：镜像的唯一标识：64位HashID。 
* CREATED：镜像 的创建时间。 
* SIZE：镜像所占用的虚拟大小，该大小包含了所有共享文件的大小。 

Commands: 

* build Build an image from a Dockerfile 
* history Show the history of an image 
* import Import the contents from a tarball to create a filesystem image 
* inspect Display detailed information on one or more images 
* load Load an image from a tar archive or STDIN 
* ls List images 
* prune Remove unused images 
* pull Pull an image or a repository from a registry 
* push Push an image or a repository to a registry 
* rm Remove one or more images save Save one or more images to a tar archive \(streamed to STDOUT by default\) 
* tag Create a tag TARGET\_IMAGE that refers to SOURCE\_IMAGE

其中，tag可以使用通配符进行匹配。

```text
docker inspect [NAME]/[CONTAINER ID]
```

images只会列出镜像的基本信息，详细信息可以通过`inspect`命令查看

### 下载

* `docker run`：命令运行时会在本地寻找镜像，找不到的时候就会去Docker Hub上面搜索并下载后运行。
* `docker search [NAME]`：下载之前可以通过**search**命令查找搜索符合的镜像
  * NAME：镜像名称。
  * DESCRIPTION：镜像的简要描述。
  * STARS：用户对镜像的评分。
  * OFFICIAL：是否为官方镜像。
  * AUTOMATED：是否使用了自动构建。
* `docker pull [NAME]`：可以预先将镜像拉到本地。镜像名必须完整地包含命名空间和仓库名。如果一个仓库中存在多个镜像，还必须制定TAG，否则使用默认TAG：latest。

### 删除

```text
docker rmi [NAME]/[CONTAINER ID]
```

对于不需要的镜像，可以使用rmi命令删除。与移除容器的命令rm相比，删除镜像的命令多了一个i，i即image的意思。

* 删除多个:多个镜像之间使用空格隔开。 
* -f：强制删除，大部分删不掉的情况可能是因为这个镜像被容器依赖了，可以选择先移除容器。 

docker rm $\(docker ps -a -q\)：如果本地有很多已经停止运行的容器，一个个删除很麻烦，可以使用下面的命令将这些容器一次性删除，这样就能减少无用容器对镜像的依赖。

* docker ps -a -q：用来列出所有容器的ID

