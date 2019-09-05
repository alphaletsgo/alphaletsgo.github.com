---
layout: post
title:  "ubuntu 安装nginx"
date:   2018-06-02
categories: [linux]
---


### 安装gcc g++的依赖库

```shell
apt-get install build-essential
apt-get install libtool
```

### 安装pcre依赖库

```shell
sudo apt-get update
sudo apt-get install libpcre3 libpcre3-dev
```

### 安装 zlib依赖库

```shell
apt-get install zlib1g-dev
```

### 安装ssl依赖库

```shell
apt-get install openssl
```

### 安装Nginx

```shell
#载最新版本：
wget http://nginx.org/download/nginx-1.11.3.tar.gz
#解压：
tar -zxvf nginx-1.11.3.tar.gz
#进入解压目录：
cd nginx-1.11.3
#配置：
./configure --prefix=/usr/local/nginx 
#注意：在Ubuntu中这里可以出现‘ checking for C compiler ... not found’问题，这里应该是上面的gcc g++库没有安装成功，可是执行：apt-get install gcc gcc-c++ kernel-devel
#编辑nginx：
make
#注意：这里可能会报错，提示“pcre.h No such file or directory”,具体详见：http://stackoverflow.com/questions/22555561/error-building-fatal-error-pcre-h-no-such-file-or-directory
#需要安装 libpcre3-dev,命令为：sudo apt-get install libpcre3-dev
#安装nginx：
sudo make install
#启动nginx：
sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
注意：-c 指定配置文件的路径，不加的话，nginx会自动加载默认路径的配置文件，可以通过 -h查看帮助命令。
#查看nginx进程：
ps -ef|grep nginx
```

特别注意在上面执行make命令的时候可能会出现‘The program 'make' can be found in the following packages:’，这说明make没有安装，运行下面命令安装：

```shell
sudo apt-get install build-essential 
```

### 启动Nginx

```shell
#-s都是采用向 Nginx 发送信号的方式。
./sbin/nginx -s stop
./sbin/nginx -s quit
```

### Nginx重新加载配置

```shell
./sbin/nginx -s reload
```

### 指定配置文件

```shell
./sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

[更多参考](http://www.cnblogs.com/piscesLoveCc/p/5794926.html)