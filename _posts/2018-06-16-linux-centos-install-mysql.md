---
layout: post
title:  "在CentOS上安装MySQL并初始化"
date:   2018-06-16
categories: [linux]
---

首先，你可能需要查看一下是否已经安装MySQL了。

```shell
yum list installed | grep mysql
```

如果，你需要卸载？

```shell
yum -y remove + 数据库名称
```

那么，接下来就是下载安装了，我使用wget工具来下载MySQL，如果没有安装wget工具可以参考如下安装方式：

```shell
yum install wget
```

成功安装好wget后，我们就可以使用wget去下载MySQL啦， 例如下载`http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm`：

```shell
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
```

添加MySQL Yum Repository 到你的系统respository列表中，

```shell
yum localinstall mysql-community-release-el7-5.noarch.rpm
```

验证一下是否添加成功

```shell
yum repolist enabled | grep "mysql.*-community.*"
```

 **选择要启用的版本**

查看MySQL版本

```shell
yum repolist all | grep mysql
```

可以看到5.5，5.7版本是默认禁用的，因为现在最新的稳定版本是5.6，查看当前的启动的MySQL版本

```shell
yum repolist enabled | grep mysql
```

执行如下命令进行安装

```shell
yum install mysql-community-server
```

以上，就是我们在centos上安装mysql的详细步骤，但是作为一个基础服务工具来说这个并没有结束，所以

- - - - - 

启动MySQL服务

```shell
systemctl start mysqld
```

查看MySQL服务状态

```shell
systemctl status mysqld
```

关闭MySQL服务

```shell
systemctl stop mysqld
```

安装完成并且启动服务后，我们需要对MySQL进行安全设置，也就是对MySQL进行初始化，比如，设置root用户的密码，设置root用户的是否具有远程访问的权限等等。

```shell
mysql_secure_installation
```

至此，安装并初始化MySQL基本结束。

