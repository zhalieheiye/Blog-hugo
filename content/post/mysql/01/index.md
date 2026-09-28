---
title: "在docker内配置mysql"
linkTitle: " " # 侧边栏/卡片上显示这个
date: "2026-09-28T18:25:44+08:00"
draft: false
tags: ['docker','mysql']
categories: []
weight: 20
keywords: [] #keywords: ["Hugo", "教程"]
cover: false
# description: "这是给搜索引擎看的页面描述。" # 不会显示在文章列表里
# summary: "这是给读者看的预览，会显示在首页卡片上。" # 会显示在文章列表卡片里
---

# 前置要求：一个装有docker的linux系统。本文教程基于wsl，具体版本信息如图所示
~~~
lsb_release -a
~~~


```
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 26.04 LTS
Release:        26.04
Codename:       resolute
```

# 思路

1. 先创建一个docker-compose.yml文件，并创建对应的挂载目录。

2. 通过docker-compose.yml文件，拉取镜像（本地没有的话，才会拉取镜像。如果有网络问题，可以先通过镜像网站拉取镜像，并修改对应的mysql版本信息）并创建容器。

3. 在容器内进行mysql的操作和练习（也可以通过其他第三方软件使用该数据库）

## 创建docker-compose.yml文件

创建文件，文件位置并没有限制。没有vim的也可以使用其他文本编辑器，如vi。
```
vim docker-compose.yml
```
如果使用vim，需要注意处于编辑模式。vim的三个模式的切换请自行上午查阅。文件内容仅供参考
```
version: '3'
services:
  mysql: 
    image: mysql:8 # 镜像版本
    container_name: mysql8 # 容器名
    environment:
      - MYSQL_ROOT_PASSWORD=123456 # root用户密码
    volumes:
    # 挂载目录：容器内需要挂载目录
      - /home/miku/docker/mysql8/log:/var/log/mysql # 日志文件挂载
      - /home/miku/docker/mysql8/data:/var/lib/mysql
      - /home/miku/docker/mysql8/conf.d:/etc/mysql/conf.d
      - /etc/localtime:/etc/localtime:ro
```

## 创建挂载目录

创建需要被挂载的目录，以上述文件为例，需要依次执行下列命令

```
mkdir -p /home/miku/docker/mysql8/log
mkdir -p /home/miku/docker/mysql8/data
mkdir -p /home/miku/docker/mysql8/conf.d
```

这里可以选择执行，mysql的配置文件

```
vim //home/miku/docker/mysql8/conf.d/my.cnf
```

文件内容，仅供参考

```
[client]
default-character-set=utf8mb4
socket=/var/lib/mysql/mysql.sock

[mysql]
default-character-set=utf8mb4
socket=/var/lib/mysql/mysql.sock

[mysqld]
port=3306
```

执行下列命令

~~~
docker-compose -f <文件路径> up -d
~~~

查看容器，容器的id为b5ee5333067d。

```
$ docker$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                 NAMES
b5ee5333067d   mysql:8   "docker-entrypoint.s…"   42 minutes ago   Up 42 minutes   3306/tcp, 33060/tcp   mysql8？
```

STATUS这里写着up，表示启动了，可以直接进入容器内进行操作。如果是下图，则是容器关闭了，需要手动启动

```
$ docker$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                     PORTS     NAMES
b5ee5333067d   mysql:8   "docker-entrypoint.s…"   44 minutes ago   Exited (0) 4 seconds ago             mysql8
```

执行如下命令，启动mysql容器

~~~
docker container start b
~~~

* 注：只要容器前缀id唯一，则可以只输入部分前缀即可。比如，这里只有一个容器，则输入b这个id就只能唯一确定这个容器，无需输入全部的容器id号


## 进入容器并运行mysql

```
docker exec -it <容器id> /bin/bash
```

登录数据库，数据库输入密码时看不见密码。密码在docker-compose.yml文件内配置了

```
mysql -u root -p
```

如果要导入数据库文件，则需要先创建数据库，进入数据库后，再进行导入。这里创建了数据库course，并设置了编码。下文course都指的是这个新创建的数据库

```
CREATE DATABASE course CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
```

使用数据库，并导入数据库文件

```
use course;
```

导入数据库（sql文件）。此处文件路径是容器内的路径

```
source <文件路径>;
```

* 注：容器内导入文件命令

  ~~~
  docker cp <宿主机文件路径，本文指的是wsl内的路径> <容器名称>:<容器内路径>
  ~~~

  例如 

  ~~~
  docker cp /path/to/local/file.txt my_container:/path/in/container/
  ~~~
此时，数据库已经导入，后续可以使用导入的数据库文件了

# 参考
[使用docker-compose 部署 MySQL（所有版本通用）](https://blog.csdn.net/weixin_44606481/article/details/132840940)