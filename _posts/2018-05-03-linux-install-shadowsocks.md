---
layout: post
title:  "ubuntu 安装Shadowsocks"
date:   2018-05-03
categories: [linux]
---


最近需要科学上网查点资料，于是就在linode上买了一个vps，linode性价比不算高，但是为了图省事也就忍了。下面我记录一下搭建科学上网环境的具体步骤。


### 第一步 安装Shadowsocks

```shell
#更新软件源
sudo apt-get update
#安装pip环境
sudo apt-get install python-pip
#安装shadowsock客户端
sudo pip install shadowsocks
#安装加密库
sudo apt–get install python–m2crypto
```

### 第二步 新建配置文件

在任意目录下新建一个json文件，命名无所谓，我的全路径：`/home/shadowsocks.json`，添加按如下内容进行编辑（正式的内容没有后面#）：

```json
{
"server":"67.209.xxx.xxx",      #你买的vps的ip
"server_port":443,              #vps端口号
"local_address":"127.0.0.1",    #客户机本机环回地址 ,可以不写
"local_port":1080,              #客户机端口号,可以自己设置
"password":"xxxxxxxx",          #vps密码
"timeout":300,                  #连接超时时间,可以不写
"method":"aes-256-cfb"          #加密方式
}
```

### 最后一步 启动
 
```shell
ssserver -c /home/shadowsocks.json -d start
```

### 结束

好了，至此科学上网环境已经搭好了。
 
 
 