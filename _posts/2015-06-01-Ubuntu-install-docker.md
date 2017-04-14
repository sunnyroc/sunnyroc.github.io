---
layout: post
title:  "Ubuntu安装Docker"
categories: Ubuntu Docker
tags: Guide
author: Roc
---

* content
{:toc}


# Ubuntu安装Docker
容器技术现在火的不得了,可移植性强,docker库中有很多成熟的官方容器可以拿来就用,对硬件条件有限切网架设多种服务的人来说,非常吸引人.

#### 安装最新稳定版
```shell
#安装依赖包
sudo apt-get -y install apt-transport-https  ca-certificates curl

#下面两行为一个命令
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update

sudo apt-get -y install docker-ce

#测试是否安装成功
sudo docker run hello-world
```
如果上面命令无效,请参考[官网最新命令](https://store.docker.com/editions/community/docker-ce-server-ubuntu?tab=description)安装CE版本

#### 开机启动
```shell
sudo systemctl enable docker
#关闭
sudo systemctl disable docker

#切换手动启动
echo manual | sudo tee /etc/init/docker.override
#切换自动自动
echo upstart | sudo tee /etc/init/docker.override
```
#### 常用命令

```bash
#查看容器日志
docker logs -f <容器名orID>
#查看正在运行的容器
docker ps
docker ps -a为查看所有的容器，包括已经停止的
#删除所有容器
docker rm $(docker ps -a -q)
#删除单个容器
docker rm <容器名orID>
#停止、启动、杀死一个容器
docker stop <容器名orID>
docker start <容器名orID>
docker kill <容器名orID>
#查看所有镜像
docker images
#删除所有镜像
docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)
#拉取镜像
docker pull <镜像名:tag>
#镜像迁移
#机器A
docker save busybox-1 > /home/save.tar
#机器B
docker load < /home/save.tar
```
