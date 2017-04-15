---
layout: post
title:  "Kafka原理认知"
date:   2017-04-05 15:18:23 +0700
categories: [kafka]
---
* content
{:toc}

# Kafka消息组织原理

#### 磁盘重认识

* 确定磁道和扇区	寻道时间
* 目标扇区旋转	旋转时间
* 磁盘到内存到网络	数据传输时间

#### 提高读取速度

* 预读数据
* 合并写-多逻辑写操作合并成一个大的物理写操作,顺序写的速度远远大于随机写.

####  写入原理

* 生产生: 网络 --> pagecache --> 磁盘
* 消费者: 磁盘 --> 网络
* 目的:一次生产多次消费


#### 删除原理

* 从最久的日志开始删除逐渐向前推进,直到日志段不满足条件为止.
* 满足给定条件predicte,配置项log.retention.{ms, minutes, hours}和log.retention.bytes指定
* 无法删除当前激活日志段,即当前使用日志文件
* 删除间隔设置 `log.retention.check.interval.ms`
* 刷盘间隔设置 `log.flush.scheduler.interval.ms`
* 记录检查点间隔 `log.flush.offset.checkpoint.interval.ms`
* 压缩 `log.cleaner.enable`


#### 检索原理

| 字段 | 内容 |
| --- | --- |
| 8字节偏移量 | 表示有多少个message |
| 4字节消息量 | 表示message的大小  |
| 4字节CRC32 | CRC32校验消息完整度 |
| 1字节`magic` | 本次kafka服务程序协议版本号 |
| 1字节`attributes` | 独立版本或标识压缩类型或编码类型 |
| 4字节key length | 当key长度为-1,k byte key不填 |
| k byte key | 可选 |
| value byte length | 实际消息数据  |
| payload | 实际消息 |

* 通过偏移量二分查找可快速定位文件

#### 集群维护

##### 实时查看和修改

>topic工具,也可以通过配置文件修改达到同样的效果

* 列出集群所有可用的topic

```shell
$ bin/kafka-topics.sh --list -zookeeper zookeeper_address

```

* 查看进群特定topic信息

```shell
$ bin/kafka-topics.sh --describle -zookeeper zookeeper_address --topic topic_name

```

* 创建topic

```shell
#replication-factor为副本数量
$ bin/kafka-topics.sh --create -zookeeper zookeeper_address --replication-factor 1 --partitions 1 --topic topic_name

```

* 增减以后topic的partition已达到扩容增强并发

```shell
#改变分区数量为4,注意无法减少分区
$ bin/kafka-topics.sh --zookeeper zookeeper_address --alter --topic topic_name --partitions 4

```

##### Leader平衡机制

* 重新分配偏爱分区,对频繁重启服务有效

```shell

$ bin/kafka-preffered-replica-election.sh -zookeeper zookeeper_address

```
* `auto.leader.rebalance.enable=true`

##### 集群分区日志迁移

* 迁移全部数据

>创建文件topics-to-move.json

```js
{"topics": [
 {"topic": "fortest1"},
 {"topic": "fortest2"},
 {"topic": "fortest3"}
 ],
 "version":1
}
```

```shell
#3,4 为broker节点id,需要做好备份,以便失败后从新迁移
$ bin/kafka-reassign-partitions.sh --zookeeper 192.168.103.47:2181 --topics-to-move-json-file topics-to-move.json --broker-list "3,4" --generate
```

>创建文件reassignment-node.json 写入上面命令得到的json数据

```shell
#开始迁移
$ bin/kafka-reassign-partitions.sh --zookeeper 192.168.103.47:2181 --reassignment-json-file reassignment-node.json --execute

#查看运行结果
$ bin/kafka-reassign-partitions.sh --zookeeper 192.168.103.47:2181 --reassignment-json-file reassignment-node.json --verify
```

* 迁移分区数据

>恩,觉得不是很实用就不下拉

##### 集群监控

* kafka offset monitor 主要监控存活机器

	* 存活的broker集合
	* topic集合
	* 消费组李彪
	* 未消费信息

* kafka manager 监控多集群

	* 多集群管理
	* 检查集群状态,可以查看副本和分区的分布情况
	* 选择副本
	* 分配分区
