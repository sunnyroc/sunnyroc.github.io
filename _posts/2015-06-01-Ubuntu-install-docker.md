---
layout: post
title:  "Ubuntu install docker"
date:   2015-06-01 15:18:23 +0700
categories: [linux]
---

* content
{:toc}


# Ubuntu安装Docker
容器技术现在火的不得了,可移植性强,docker库中有很多成熟的官方容器可以拿来就用,对硬件条件有限切网架设多种服务的人来说,非常吸引人.

#### 安装最新稳定版
{% highlight shell %}
#安装依赖包
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#Verify that the key fingerprint is 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88.
$ sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
{% endhighlight %}
`amd64:`
{% highlight shell %}
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
{% endhighlight %}
`armhf:`
{% highlight shell %}
$ sudo add-apt-repository \
   "deb [arch=armhf] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
{% endhighlight %}
卸载Docker

{% highlight shell %}
sudo apt-get purge docker-ce

#Images, containers, volumes, or customized configuration files on your host are not automatically removed. To delete all images, containers, and volumes:
$ sudo rm -rf /var/lib/docker
{% endhighlight %}

如果上面命令无效,请参考[官网最新命令](https://store.docker.com/editions/community/docker-ce-server-ubuntu?tab=description)安装CE版本

#### 开机启动
{% highlight shell %}
sudo systemctl enable docker
#关闭
sudo systemctl disable docker

#切换手动启动
echo manual | sudo tee /etc/init/docker.override
#切换自动自动
echo upstart | sudo tee /etc/init/docker.override
{% endhighlight %}
#### 常用命令

{% highlight shell %}
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
{% endhighlight %}

#### 安装Docker Compose
[官网地址](https://docs.docker.com/compose/install/)

{% highlight shell %}
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ docker-compose --version

docker-compose version 1.11.2, build dfed245

#移除
$ rm /usr/local/bin/docker-compose

#升级到指定版本替换下面migrate-to-labels
$ docker-compose migrate-to-labels
{% endhighlight %}

pip方式安装

{% highlight shell %}
$ sudo pip install -U docker-compose
{% endhighlight %}
