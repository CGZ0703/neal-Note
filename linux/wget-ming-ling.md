# wget命令

- wget 是一个用于下载文件的免费工具，它支持大多数常用的Internet协议，包括 HTTP, HTTPS, 以及 FTP.
- wget这个名字来源于 World Wide Web + get. wget 有很多功能，可以很方便地做到下载大型文件,递归下载,一次下载多个文件以及镜像web网站和FTP站点.
- wget是非交互式的，但是使用起来相当的灵活. 你可以在脚本，cron任务，终端等地方调用它. 
- 它可以在用户未登陆的情况下运行在后台. 也就是说你可以开始下载文件，然后退出系统，wget会在后台运行直到完成任务.

## 下载单个文件

### 最常用的用法就是用来下载单个文件

```
wget https://example.com/latest.zip
```
- wget会显示出下载的进度, 当前下载速度, 文件大小, 当前日期时间 以及待下载文件的名称.
- 文件会保存为 latest.zip 存放在当前目录

### 下载文件重命名

```
wget -O example.zip https://example.com/latest.zip
```
- 文件会保存为 example.zip 存放在当前目录

### 指定下载的目录

```
wget -P /home/example https://example.com/latest.zip
```
- 文件会存放在 /home/example 目录中

### 限制下载速度

```
wget --limit-rate=300k https://example.com/latest.zip
```
- 限制速度为 300KB/s

### 断点续传

```
wget -c https://example.com/latest.zip
```
- 若下载中断后你没有用 -c 进行断点续传，而是重新下载, wget 会在文件名后加上 “.1” 防止与前面下载的文件重名.

### 后台下载

```
wget -b http://example.com/big-file.zip
```
- 使用 -b 指令,输出内容会写入同目录下的 wget-log 文件, 可以用 `tail -f wget-log` 命令来检查下载状态

### 设置重试次数

```
wget -tries=100 https://example.com/file.zip
```
- 若网络有问题导致下载时常中断,就可以使用 -tries 选项增加重试次数

## 下载多个文件

```
wget -i download.txt
```
- 使用 -i 下载 download.txt 文件中所有的地址

## 下载FTP文件

```
wget --ftp-user=username --ftp-password=password ftp://url-to-ftp-file
```
- 使用账户名和密码，下载FTP文件

## 下载整个网站

```
wget --mirror --convert-links --page-requisites --no-parent -P /home/example/download https://example.com
```
1. —mirror 会开启镜像所需要的所有选项.
2. -convert-links 会将所有链接转换成本地链接以便离线浏览.
3. –page-requisites 表示下载包括CSS样式文件，图片等所有所需的文件，以便离线时能正确地现实页面.
4. –no-parent 用于限制只下载网站的某一部分内容.
5. -P 设置下载路径

