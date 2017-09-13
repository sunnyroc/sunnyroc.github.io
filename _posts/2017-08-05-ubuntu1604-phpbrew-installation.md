---
layout: post
title:  "Phpbrew installation in Ubuntu16.04"
date:   2017-08-05 15:18:23 +0700
categories: [php]
---
* content
{:toc}

# 简介
针对Ubuntu16.04版本的安装尝试,自由切换php小版本.

# 准备工作
phpbrew的官方地址:https://github.com/phpbrew/phpbrew
在安装phpbrew之前，需要预先安装一些库.

```bash
apt-get install -y php \
	php-pear \
	autoconf \
	automake \
	curl \ 
	libcurl3-openssl-dev \
	build-essential \
	libxslt1-dev \
	re2c \
	libxml2 \
	libxml2-dev \
	bison \
	libbz2-dev \
	libreadline-dev\
	libfreetype6 \
	libfreetype6-dev \
	libpng12-0 \
	libpng12-dev \
	libjpeg-dev \
	libjpeg8-dev \
	libjpeg8  \
	libgd-dev \
	libgd3 \
	libxpm4 \
	libltdl7 \
	libltdl-dev \
	libssl-dev \
	openssl \
	gettext \
	libgettextpo-dev \
	libgettextpo0 \
	libicu-dev \
	libmhash-dev \
	libmhash2 \
	libmcrypt-dev \
	libmcrypt4
```

## 安装phpbrew

```bash
curl -L -O https://github.com/phpbrew/phpbrew/raw/master/phpbrew
chmod +x phpbrew

# Move phpbrew to somewhere can be found by your $PATH
sudo mv phpbrew /usr/local/bin/phpbrew
```

## 出初始化phpbrew

```bash
phpbrew init
```

## 配置环境变量

```bash
# 在 .bashrc中加入如下的代码
[[ -e ~/.phpbrew/bashrc ]] && source ~/.phpbrew/bashrc

# 更新bashrc
source .bashrc
```

# 安装LAMP环境

[LAMP的安装教程](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-in-ubuntu-16-04)
在Ubuntu16.04的环境下，默认安装的是php7.
安装php的命令如下：
```bash
phpbrew --debug install --stdout 5.6.16  +default +mysql +gd +fpm
```

其中 --debug --stdout意思将debug信息在控制台输出，同时安装了mysql和gd,fpm的扩展库。fpm库是一定要安装的库.
安装完毕之后，切换至5.6.16的版本，启动php-fpm

```bash
phpbrew use php-5.6.16
phpbrew fpm start
```

查看sock的权限

```bash
ls -ahl /home/spoock/.phpbrew/php/php-5.6.16/var/run/php-fpm.sock
```

对用户权限的修改就是在安装过程中遇到的最大的坑。
正常情况下一般都是root权限，将其修改为当前用户的权限，比如我当前的用户是spoock,则运行

```bash
chown -R spoock:spoock ~/.phpbrew/php/
```

修改完毕之后，对应的所有的配置文件都需要进行修改
包括/home/spoock/.phpbrew/php/php-5.5.38/etc/php-fpm.conf和/etc/nginx/nginx.conf
对php-fpm.conf的修改：

```bash
listen.owner = spoock
listen.group = spoock
user = spoock
group = spoock
```
对nginx.conf的修改：

```bash

user spoock;

```

nginx.conf的配置/etc/nginx/sites-available/default

```bash
location ~ \.php$ {
#       # With php7.0-fpm:
        fastcgi_index index.php;
        fastcgi_pass unix:/home/spoock/.phpbrew/php/php-5.5.38/var/run/php-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
}
```

至此，所有的配置均已完成。选择需要使用的php版本，运行下面的命令:

```bash
phpbrew use 5.6.16
phpbrew fpm start
sudo service nginx start
```

就开启开启php-5.6.16的测试了。