---
title: "Redis入门教程(一)---安装与配置"
description: "Linux下通过编译方式安装redis"
date: 2019-02-20 22:00:00
draft: false
tags: ["Redis"]
categories: ["Redis"]
---

本文主要记录了如何在 Linux(CentOS 7) 下安装 Redis 与 Redis 基本使用与配置。

<!--more-->

> **[Redis系列教程目录](https://www.lixueduan.com/categories/Redis/)**
>
> [Redis入门教程(一)---安装与配置](https://www.lixueduan.com/posts/876962d5.html)
>
> [Redis入门教程(二)---五大基础数据类型与常用命令](https://www.lixueduan.com/posts/8380a4fa.html)
>
> [Redis入门教程(三)---安全性、事务、发布订阅](https://www.lixueduan.com/posts/5df38113.html)
>
> [Redis入门教程(四)---主从复制与持久化](https://www.lixueduan.com/posts/84dd6d72.html)
>
> [Redis入门教程(五)---搭建Redis集群](https://www.lixueduan.com/posts/397ed67.html)
>
> [Redis入门教程(六)---通过JavaApi(Jedis)操作Redis](https://www.lixueduan.com/posts/e456b1e5.html)
>
> [Redis入门教程(七)---通过 Docker 安装 Redis](https://www.lixueduan.com/posts/96375af.html)
>
> .......
>
> 更多文章欢迎访问我的个人博客-->[幻境云图](https://www.lixueduan.com/)

## 1. Redis安装

### 1.1 下载

官网`https://redis.io/download` 下载文件，这里下的是`redis-5.0.3.tar.gz`

然后上传到服务器。这里是放在了`/usr/software`目录下

### 1.2 环境准备

**安装编译源码所需要的工具和库**:

```linux
# yum install gcc gcc-c++ ncurses-devel perl 
```

### 1.3 解压安装

#### 1. 解压

```linux
[root@localhost software]# tar -zxvf redis-5.0.3.tar.gz -C /usr/local
//解压到/usr/local目录下
```

#### 2. 编译

进入刚才解压的后的文件夹`redis-5.0.3`进行编译

```linux
[root@localhost redis-5.0.3]# make
```

如果提示`Hint: It's a good idea to run 'make test' ;)`就说明编译ok了，接下来进行安装。

#### 3. 安装

进入`src`目录下

```linux
[root@localhost redis-5.0.3]# cd src/
[root@localhost src]# make install
```

出现下面的提示代表安装ok

```shell
    CC Makefile.dep

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install

```

#### 4. 文件复制

创建两个文件夹来存放Redis命令和配置文件。

```linux
[root@localhost local]# mkdir -p /usr/local/redis/etc
[root@localhost local]# mkdir -p /usr/local/redis/bin
```

把``redis-5.0.3`下的`redis.conf`复制到`/usr/local/redis/etc`目录下

```linux
[root@localhost redis-5.0.3]# cp redis.conf /usr/local/redis/etc/
```

把`redis-5.0.3/src`里的`mkreleasehdr.sh`、`redis-benchmark`、`redis-check-aof`、`redis-check-rdb`、`redis-cli`、`redis-server` 文件移动到`/usr/local/redis/bin`下

```linux
[root@localhost src]# mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-rdb redis-cli redis-server /usr/local/redis/bin
```

## 2. 启动

### 2.1 前台启动

启动时并指定配置文件：.

```linux
[root@localhost etc]# /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
```

出现如下提示代表启动成功

```shell
[root@localhost etc]# /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
10869:C 05 Mar 2019 13:33:39.041 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
10869:C 05 Mar 2019 13:33:39.042 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=10869, just started
10869:C 05 Mar 2019 13:33:39.042 # Configuration loaded
10869:M 05 Mar 2019 13:33:39.044 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10869
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

10869:M 05 Mar 2019 13:33:39.046 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
10869:M 05 Mar 2019 13:33:39.046 # Server initialized
10869:M 05 Mar 2019 13:33:39.047 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
10869:M 05 Mar 2019 13:33:39.047 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
10869:M 05 Mar 2019 13:33:39.047 * DB loaded from disk: 0.000 seconds
10869:M 05 Mar 2019 13:33:39.047 * Ready to accept connections

```

**退出**：`CTRL+C`

### 2.2 后台启动

(**注意要使用后台启动需要修改`redis.conf`里的`daemonize`改为`yes`**)

```shell
[root@localhost etc]# vim redis.conf 
#主要修改下面这个daemonize
# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# daemonize no 把这个改为yes no代表前台启动 yes代表后台启动
daemonize yes

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
# dir ./  这个是工作区 默认为./ 即上级目录 这里也改一下
dir /usr/local/redis/etc
```

再次启动

```linux
[root@localhost etc]# /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
```

**验证启动是否成功**：

```linux
[root@localhost etc]#/ps aux|grep redis

root      11012  0.2  0.2 153880  2344 ?        Ssl  13:36   0:00 /usr/local/redis/bin/redis-server 127.0.0.1:6379
root      11126  0.0  0.0 112708   976 pts/2    R+   13:39   0:00 grep --color=auto redis
```

redis启动成功端口号也是默认的6379。

### 2.3 使用

#### 1. 进入客户端

进入redis客户端

```shell
#语法redis-cli -h host -p port -a password
#连接本机则不用写host和port 这里没设置密码也不用写

[root@localhost etc]# /usr/local/redis/bin/redis-cli 
127.0.0.1:6379> 
```

成功进入Redis客户端

随意操作一下：

```shell
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> set name illusory
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> get name
"illusory"
127.0.0.1:6379> set age 22
OK
127.0.0.1:6379> get age
"22"
127.0.0.1:6379> quit #退出命令
[root@localhost etc]# 
```

**退出客户端**：`quit`

#### 2. 关闭Redis

退出redis服务的三种方法：

- 1.`pkill redis-server`、
- 2.`kill 进程号`、
- 3.`/usr/local/redis/bhi/redis-cli shutdown`

#### 3. dump.rdb文件

由于前面配置文件中配置的是`dir /usr/local/redis/etc`,所以Redis的所有数据都会存储在这个目录

```linux
[root@localhost etc]# ll
total 68
-rw-r--r--. 1 root root    92 Mar  5 13:36 dump.rdb
-rw-r--r--. 1 root root 62174 Mar  5 13:36 redis.conf
```

确实有这个文件。**这个文件删除后Redis中的数据就真的没了**。



## 3. 参考

`http://www.runoob.com/redis/redis-hashes.html`

