---
layout: post
title:  "Zookeeper集群搭建"
date:   2017-04-01 15:18:23 +0700
categories: [linux]
---

* content
{:toc}


# Zookeeper环境安装

Zookeeper配置服务器集群遵循2N+1的原则,即奇数台服务器原则(因为半数服务器挂掉即停掉所有服务)

[打开最新稳定版本包下载地址](http://apache.01link.hk/zookeeper/stable/)
选中需要下载的包右键获取连接地址,通过命令行下载并解压如下:

```shell
$ wget http://apache.01link.hk/zookeeper/stable/zookeeper-3.4.10.tar.gzls

$ tar -zxvf zookeeper-3.4.10.tar.gz

pwd

```

### 创建data和log目录


```shell
$ mkdir zkdata
$ mkdir zkdataLog
```
进入配置文件夹

```shell
$ cd zookeeper-3.4.10/conf
```


### 修改配置文件


拷贝并从新命名官方提供的样本文件zoo_sample.cfg为zoo.cfg


#### 从命名配置文件


```shell
$ cp zoo_sample.cfg zoo.cfg
$ vi zoo.cfg
```

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
# 总花费时间为2000*10
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
# 总花费时间为2000*5
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```


### 修改默认配置项


* 修改`dataDir`为我们刚刚创建的zkdata路径(快照日志目录)
* 添加新的配置项`dataLogDir`在`dataDir`下方,路径为刚刚创建的`zkdataLog`路径.(事务目录,如果不配置讲放入到`dataDir`目录,造成性能下降)
* 修改端口为任意不冲突端口
如下

```shell
dataDir=/home/roc/fabric/zkdata
dataLogDir=/home/roc/fabric/zkdataLog
clientPort=21811
```


### 添加集群通信配置项


```shell
#机器标识为1的ip地址,跟随者端口默认2888:选举端口3888
server.1=192.168.186.40:2888:3888
server.2=192.168.186.41:2888:3888
server.3=192.168.186.42:2888:3888
```
在对应服务器中,转到`dataDir`目录,创建一个**对应的**标识文件`myid`,内容为“1”(可以命名其他标识，为了便于记忆我们用数字编号作为标识)
```shell
$ cd /home/roc/fabric/zkdata
$ echo "1" > myid
```
>在例子中我配置了三台服务器,那么我需要在这三台服务器上面分别执行上面操作,为了方便可以直接copy配置文件,但是需要手动创建对应标识文件`myid`文件

### 启动服务


```shell
#进入`bin`目录,执行脚本zkServer.sh
cd /home/roc/fabric/zookeeper-3.4.10/bin
#查看命令
./zkServer.sh

ZooKeeper JMX enabled by default
Using config: /home/roc/fabric/zookeeper-3.4.10/bin/../conf/zoo.cfg
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}

./zkServer.sh start

ZooKeeper JMX enabled by default
Using config: /home/roc/fabric/zookeeper-3.4.10/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED

#查看集群状态

```

>对其他服务器做同样的操作

# Zookeeper环境安装(Docker)


[安装docker](https://sunnyroc.github.io/2015/06/01/Ubuntu-install-docker/)
[Zookeeper](https://hub.docker.com/r/library/zookeeper/)

```shell
$ sudo docker pull zookeeper
#进入zookeeper安装目录，创建docker-compose.yml配置文件
$ cd /home/roc/fabric/zookeeper-3.4.10
$ touch docker-compose.yml
$ vim docker-compose.yml
```

```shell
version: '2'
services:
    zoo1:
        image: zookeeper
        restart: always
        ports:
            - 2181:2181
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo2:
        image: zookeeper
        restart: always
        ports:
            - 2182:2181
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo3:
        image: zookeeper
        restart: always
        ports:
            - 2183:2181
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

```

>上面配置了3个服务器,客户端端口为2888，追谁者端口为3888


```shell
#后台执行compose,为了避免与其他项目混淆,我们可以使用COMPOSE_PROJECT_NAME=project_name作为命令行前缀来区分不同的compose服务
#例如:COMPOSE_PROJECT_NAME=project_name docker-compose up -d
$ docker-compose up -d
#查看启动容器列表
#例如:COMPOSE_PROJECT_NAME=project_name docker-compose ps
$ docker-compose ps

Name                   Command               State                     Ports
---------------------------------------------------------------------------------------------------
zktest_zoo1_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
zktest_zoo2_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2182->2181/tcp, 2888/tcp, 3888/tcp
zktest_zoo3_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2183->2181/tcp, 2888/tcp, 3888/tcp
```
```shell

$ docker run -it -d --rm \
        --link zoo1:zk1 \
        --link zoo2:zk2 \
        --link zoo3:zk3 \
        --net zktest_default \
        zookeeper zkCli.sh -server zk1:2181,zk2:2181,zk3:2181

$ zkCLI -server localhost:2181,localhost:2182,localhost:2183

#hostname -i查询服务器ip
$ hostname -i
192.168.186.40

```

切换到客户端,查看集群状态

```shell
$ echo stat | nc 127.0.0.1 2181
Zookeeper version: 3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
Clients:
 /192.168.186.27:53356[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/6/53
Received: 26
Sent: 25
Connections: 1
Outstanding: 0
Zxid: 0x40000000a
Mode: follower
Node count: 4

$ echo stat | nc 127.0.0.1 2182

Zookeeper version: 3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
Clients:
 /172.18.0.5:40178[1](queued=0,recved=47,sent=47)
 /192.168.186.27:53364[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/8
Received: 49
Sent: 48
Connections: 2
Outstanding: 0
Zxid: 0x40000000b
Mode: leader
Node count: 4

$ echo stat | nc 192.168.186.40 2183

Zookeeper version: 3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT
Clients:
 /192.168.186.27:53365[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/2/14
Received: 8
Sent: 7
Connections: 1
Outstanding: 0
Zxid: 0x40000000b
Mode: follower
Node count: 4
```

### Leader的作用
leader响应客户端的读写请求，想follower发送数据,follower从leader中同步数据,如果失败则从新选择一个leader, leader停机后会从follower中重新选择一个服务器作为新的leader
通过`jps`命令查看主进程


### 重要的配置文件和配置项

* myid文件和server.myid 服务标识,用来注册和发现服务
* zoo.cfg文件 主配置文件
* log4j.properties文件 日志配置文件
* `zkEnv.sh` 启动时设置的环境变量
* `zkCleanup.sh` **对于事务日志要定期清理**

>官方提供的清理脚本
java -cp zookeeper.jar:lib/slf4j-api-1.6.1.jar:lib/slf4j-log4j12-1.6.1.jar:lib/log4j-1.2.15.jar:conf org.apache.zookeeper.server.PurgeTxnLog `<dataDir>` `<snapDir>` -n `<count>`
也可以直接使用自带的`zkCleanup.sh`脚本,将`$*` 改成指定数字`-n 5`(保留多少个快照日志,官方建议是3个,为了安全我们改成5)

```shell
# use POSTIX interface, symlink is followed automatically
ZOOBIN="${BASH_SOURCE-$0}"
ZOOBIN="$(dirname "${ZOOBIN}")"
ZOOBINDIR="$(cd "${ZOOBIN}"; pwd)"

if [ -e "$ZOOBIN/../libexec/zkEnv.sh" ]; then
  . "$ZOOBINDIR"/../libexec/zkEnv.sh
else
  . "$ZOOBINDIR"/zkEnv.sh
fi

ZOODATADIR="$(grep "^[[:space:]]*dataDir=" "$ZOOCFG" | sed -e 's/.*=//')"
ZOODATALOGDIR="$(grep "^[[:space:]]*dataLogDir=" "$ZOOCFG" | sed -e 's/.*=//')"

if [ "x$ZOODATALOGDIR" = "x" ]
then
"$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
     -cp "$CLASSPATH" $JVMFLAGS \
     org.apache.zookeeper.server.PurgeTxnLog "$ZOODATADIR" -n 5
else
"$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
     -cp "$CLASSPATH" $JVMFLAGS \
     org.apache.zookeeper.server.PurgeTxnLog "$ZOODATALOGDIR" "$ZOODATADIR" -n 5
fi

```
>创建定时任务自动执行脚本清理工作
> minute hour day month dayofweek command
  minute - 从0到59的整数
  hour - 从0到23的整数
  day - 从1到31的整数 (必须是指定月份的有效日期)
  month - 从1到12的整数 (或如Jan或Feb简写的月份)
  dayofweek - 从0到6的整数，0或6用来描述周日 (或用Sun或Mon简写来表示)
  command - 需要执行的命令(可用as ls /proc >> /tmp/proc或 执行自定义脚本的命令)

```shell
#查看以后的定时任务
$ crontab -l
#编辑一个定时任务
$ crontab -e
#每周日0点执行
0 0 * * 0 sh /home/roc/fabric/zookeeper-3.4.10/bin/zkCleanup.sh
```
