---
layout: post
title:  "Linux 安装Shadowsocks"
date:   2018-05-03
categories: [linux]
---


最近需要科学上网查点资料，于是就在linode上买了一个vps，linode性价比不算高，但是为了图省事也就忍了。下面我记录一下搭建科学上网环境的具体步骤。

## ubuntu篇

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

## Centos 篇

前面记录了在Ubuntu下安装shadowsocks的方法。下面我们再记录一下在centos7下安装ss。

### 安装pip

与Ubuntu有些不一样，我们使用`curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py` 命令安装。

```shell
[root@ssserver ~]# curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1604k  100 1604k    0     0  11.1M      0 --:--:-- --:--:-- --:--:-- 11.2M
```

下载完成后再在终端执行：`python get-pip.py`，执行成功会看到类似如下信息：

```shell
Installing collected packages: pip, wheel
Successfully installed pip-18.0 wheel-0.31.1
```

### 安装shadowsocks

在终端输入`pip install shadowsocks`命令，成功会看到类似如下：

```shell
Successfully built shadowsocks
Installing collected packages: shadowsocks
Successfully installed shadowsocks-2.8.2
```
 
### 配置shadowsocks

输入编辑文件命令`vi /etc/shadowsocks.json`并回车，配置类似：

```shell
{
    "server":"0.0.0.0",
    "server_port":50013,  //服务器的端口
    "password":"1234567890",    //自己设定的密码
    "method":"aes-256-cfb"  //加密方法，推荐使用"aes-256-cfb"
}
```

若是多用户模式，将server_port和password合并为port_password：

```shell
"port_password": {
         "50013": " mypassword 1”,  //对应端口设定不同的密码
         "8888": " mypassword 2”
     },
```

### 将shadowsocks加入系统服务

输入编辑文件命令`vi /etc/systemd/system/shadowsocks.service`并回车，粘贴下面内容：

```shell
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target

```

保存并退出vi。

### 启动shadowsocks服务并设置开机自启

- 设置开机自启命令：`systemctl enable shadowsocks`

- 启动命令：`systemctl start shadowsocks`

- 查看状态命令：`systemctl status shadowsocks`

如果成功的话，会看到类似如下信息：

```shell
[root@ssserver ~]# systemctl start shadowsocks
[root@ssserver ~]# systemctl status shadowsocks
● shadowsocks.service - Shadowsocks
   Loaded: loaded (/etc/systemd/system/shadowsocks.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2018-08-28 13:27:53 UTC; 7s ago
 Main PID: 1259 (ssserver)
   CGroup: /system.slice/shadowsocks.service
           └─1259 /usr/bin/python /usr/bin/ssserver -c /etc/shadowsocks.json

Aug 28 13:27:53 ssserver systemd[1]: Started Shadowsocks.
Aug 28 13:27:53 ssserver systemd[1]: Starting Shadowsocks...
Aug 28 13:27:54 ssserver ssserver[1259]: INFO: loading config from /etc/shadowsocks.json
Aug 28 13:27:54 ssserver ssserver[1259]: 2018-08-28 13:27:54 INFO     loading libcrypto from libcrypto.so.10
Aug 28 13:27:54 ssserver ssserver[1259]: 2018-08-28 13:27:54 INFO     starting server at 0.0.0.0:50013
```

### 防火墙问题

有时候我们按照上面的方式配置了，也在ss客户端配置，但就是无法科学上网，这时问题可能出现在我们设置的端口被防火墙禁用了，在centos7下使用的filewalle防火墙，我们只需修改`/etc/firewalld/zones/public.xml`文件，在<zone>标签中添加如下：

```shell
<port protocol="tcp" port="8088"/>
<port protocol="udp" port="8088"/>
```
保存后，运行:

```shell
# firewall-cmd --complete-reload
```

### 性能提升

Gevent可以提高ss性能，由于gevent依赖于libevent和greenlet

```shell
$ yum install -y libevent
$ pip install greenlet
$ pip install gevent
```