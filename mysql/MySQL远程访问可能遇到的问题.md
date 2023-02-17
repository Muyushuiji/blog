---
title: MySQL远程访问可能遇到的问题
date: 2023-02-17 17:41:00
categories:
- MySQL
tags:
- mysql
- 部署
---

#### 前景

在ubuntu搭建完mysql后，本地连接出现`10061(Unknow Server) error code`。

#### 分析

1. mysql 配置文件未开启远程访问

修改mysql的配置文件

```bash
#ubuntu默认安装的配置文件在  /etc/mysql/mysql.conf.d/mysqld.cnf
vim  /etc/mysql/mysql.conf.d/mysqld.cnf

将下面这句注释掉
bind-address           = 127.0.0.1
#bind-address           = 127.0.0.1

#授权给root用户
grant all privileges  on *.* to root@"%" identified by "password";
flush privileges;
```



2. linux服务器未开放3306端口

```bash
#查看端口开放情况
sudo ufw status numbered

#若未启动，则启动防火墙
sudo systemctl enable firewalld
sudo systemctl start firewalld

#开放xx端口指定的协议
sudo ufw allow [port]/[protocol] 
sudo ufw allow 3306/tcp

#重新载入
sudo ufw reload
```

如果是red-hat发行的linux版本，则使用firewalld

```bash
#启动和关闭
systemctl start firewalld
systemctl stop firewalld

#查看状态
systemctl status firewalld

#开启自启动和禁用
systemctl disable firewalld
systemctl enable firewalld
#显示状态
firewall-cmd --state

#查看所有打开的端口
firewall-cmd --zone=public --list-ports

#重新载入防火墙规则
firewall-cmd --reload

#开发和删除端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```



2. 云服务器防火墙3306规则没开放

若购买的是腾讯云轻量应用服务器，则无需配置安全组，详情见[轻量应用服务器防火墙与操作系统防火墙有什么区别](https://cloud.tencent.com/document/product/1207/44569#.E8.BD.BB.E9.87.8F.E5.BA.94.E7.94.A8.E6.9C.8D.E5.8A.A1.E5.99.A8.E9.98.B2.E7.81.AB.E5.A2.99.E4.B8.8E.E6.93.8D.E4.BD.9C.E7.B3.BB.E7.BB.9F.E9.98.B2.E7.81.AB.E5.A2.99.E6.9C.89.E4.BB.80.E4.B9.88.E5.8C.BA.E5.88.AB.EF.BC.9F)

点击云服务器的实例 - > 详情 -> 防火墙 -> 添加规则

```txt
参数选择
自定义 、TCP 、3306 、允许。
备注可不填
```

