---
title: 服务器上搭建shadowsocks
date: 2018-06-04 23:07:40
tags: 服务器
---


## Ubuntu 16.版本
#### 虽然有的国外服务器是能够一键安装shadowsocks，例如bwh，但是他只能在centos 6（还是5）之前的Linux版本上安装，如果想使用别的就需要自己配置，这里以Ubuntu 16.版本为例
``` bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt-get install python-pip
$ sudo apt-get install python-m2crypto
$ pip install shadowsocks
$ sudo vim /etc/shadowsocks.json
```

### 在创建的shadowsocks.json文件里面写入
``` bash
{
"server":"0.0.0.0",   #填入服务器的ip地址
"server_port":8388,   #服务器提供服务端口
"local_port":1080,    #1080是本地代理端口
"password":"123456",  #123456为vps密码  
"timeout":600,
"method":"aes-256-cfb"  #加密方式
}
```

### 修改好相关参数后，使用命令

``` bash
$ sudo ssserver -c /etc/shadowsocks.json -d start
```

### 自此服务器配置成功，接下来在本地安装shadowsocks填入相关信息即可。

