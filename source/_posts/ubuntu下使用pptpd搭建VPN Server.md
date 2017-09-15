---
title: ubuntu下使用pptpd搭建VPN Server
date: 2016-06-29 14:04:21
tags: "零碎记录"
---
由于之前用ss搭建的出校器没有预期的好用，主要还是ss不方便配置全局，路由器上的ss又不走udp，导致这几天一直在折腾实验室的网络
今天尝试了下用pptpd搭VPN Server，搭VPN的方法很多，据说pptpd比较简单，所以选择了它
网上的资料比较多，基本按网上的步骤操作，主要只是为了记录一下过程
<!-- more -->
安装pptpd：
```bash
sudo apt-get install pptpd
```
配置虚拟ip，编辑 /etc/pptpd.conf：
```bash
localip 10.20.41.50  #本机ip
remoteip 10.100.123.2-100  #分配的ip段
```
在/etc/ppp/pptpd-options中设置dns：
```bash
#根据实际情况设置dns
ms-dns 202.193.52.229  #自己内网的dns服务器
ms-dns 8.8.4.4
```
在/etc/ppp/chap-secrets中配置VPN帐号：
```bash
"user" pptpd "user" * #星号是不限制IP的意思
```
重启pptpd服务：
```bash
sudo /etc/init.d/pptpd restart
```
在/etc/sysctl.conf中配置ip转发（取消该行注释）：
```bash
net.ipv4.ip_forward=1
```
使配置立即生效：
```bash
sudo sysctl -p
```
安装iptables，这个是用于配置NAT映射的：
```bash
sudo apt-get install iptables
```
建立一个NAT：
```bash
sudo iptables -t nat -A POSTROUTING -s 10.100.123.0/24 -o wlo1 -j MASQUERADE
```
其中-o参数配置的是流量的出口，也就是我这的外网出口。
配置哪些包走内网：
上一条命令相当于全局走外网的，为了让部分数据包走内网的线路，需要额外的配置。
当目的地址为某个内网地址时就走内网的线路。
-d 后面的为目标地址 -o为输出的线路，由于我的wlo1连着外网，eno1连着内网，这条命令可以使目的地址为202.193.0.0/16的都走eno1的内网线路。
```bash
sudo iptables -t nat -A POSTROUTING -d 202.193.0.0/16 -o eno1 -j MASQUERADE
```
设置MTU，防止包过大而丢包：
```bash
sudo iptables -A FORWARD -s 10.100.123.0/24-p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1200
```
保存规则（此处需要root权限）：
```bash
sudo iptables-save >/etc/iptables-rules
```
编辑/etc/network/interfaces，在末尾加一行，使网卡加载时自动加载规则：
```bash
pre-up iptables-restore </etc/iptables-rules
```
配置到此完成。
