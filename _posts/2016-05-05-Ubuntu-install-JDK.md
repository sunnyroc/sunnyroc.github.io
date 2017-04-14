---
layout: post
title:  "Ubuntu安装JDK"
categories: Ubuntu java
tags: Guide
author: Roc
---

* content
{:toc}


# Ubuntu安装OpenJDK 或 OralceJDK

>打开终端后查看是否安装
java -version

##### 安装OpenJDK

```shell
#使用下面命令行安装,不需要额外配置环境变量
sudo apt-get install default-jre
sudo apt-get install default-jdk
```
>更多软件版本用下列查询命令

```shell
sudo apt-cache search jre
sudo apt-cache search jdk
```

##### 安装Oracle JDK


>如果想安装java7,替换掉命令中java8即可
>命令行安装依赖网络环境

```shell
#使用下面命令行安装,不需要额外配置环境变量
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default

```


##### 通过bin包安装
>这种方式需要配置环境变量
[从官网下载对应tar包](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

如下载后文件为`jdk-8u101-linux-x64.tar.gz`
使用如下命令解压:


```shell
sudo tar zxvf ./jdk-8u101-linux-x64.tar.gz
```
为了方便管理,请将解压后的文件夹短命名然后移至到你的统一包管理文件夹内
例如:`/usr/java/jdk1.8/`

编辑文件 `~/.bashrc` 配置java环境变量


```shell
JAVA_HOME=/usr/java/jdk1.8/
JRE_HOME=$JAVA_HOME/jre
JAVA_BIN=$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

如果使用命令行源安装,默认安装路径为`/usr/lib/jvm/xxxxx`
修改JAVA_HOME路径为对应路径即可

使用下面命令使配置及时生效


```shell
source ~/.bashrc
```
最后再次使用下面命令验证配置是否生效

```shell
java -version
```
