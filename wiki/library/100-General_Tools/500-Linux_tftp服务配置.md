<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
# 在虚拟机Linux上安装配置tftp服务
>1. [安装tftp](#安装tftp "安装tftp")
1. [使用rpm -ivh xxx.rpm 命令安装以上三个rpm包。](#使用rpm -ivh xxx.rpm 命令安装以上三个rpm包。 "使用rpm -ivh xxx.rpm 命令安装以上三个rpm包。")
1. [打开 /etc/xinted.d/tftp ，修改tftp文件为以下内容：](#打开 /etc/xinted.d/tftp ，修改tftp文件为以下内容： "打开 /etc/xinted.d/tftp ，修改tftp文件为以下内容：")
1. [启动tftp服务](#启动tftp服务 "启动tftp服务")
1. [测试命令](#测试命令 "测试命令")
1. [小结](#小结 "小结")
<!-- /TOC -->

# 在虚拟机Linux上安装配置tftp服务

在Linux内核开发过程中，经常要到tftp启动内核和使用NFS文件系统挂载跟文件系统来提高开发效率，同时也能给开发者带来更好的开发环境。
本文档以在Centos 6 上安装tftp服务为例。

## 安装tftp

从Centos的安装镜像中的packages中找到以下RPM安装包：

	tftp-0.42-3.1.el5.cento.i386.rpm
	tftp-server-0.42-3.1.el5.centos.i386.rpm
	xinetd-2.3.14-10.el5.i386.rpm

## 使用rpm -ivh xxx.rpm 命令安装以上三个rpm包。
rpm命令可以一次安装多个包。当rpm安装rpm包时发生互相依赖的情况可以执行： rpm -ivh *.rpm *.rpm

安装完成后使用 rpm 查看安装结果，如下所示：

```txt
# rpm -qa | grep tftp
224:tftp-0.42-3.1.el5.centos
674:tftp-server-0.42-3.1.el5.centos
# rpm -qa | grep xinetd
448:xinetd-2.3.14-10.el5
```

## 打开 /etc/xinted.d/tftp ，修改tftp文件为以下内容：

```txt
# default: off
# description: The tftp server serves files using the trivial file transfer \
#	protocol.  The tftp protocol is often used to boot diskless \
#	workstations, download configuration files to network-aware printers, \
#	and to start the installation process for some operating systems.
service tftp
{
	socket_type		= dgram
	protocol		= udp
	wait			= yes
	user			= root
	server			= /usr/sbin/in.tftpd
	server_args		= -s /home/glenn/tftpboot -c
	disable			= no
	per_source		= 11
	cps			= 100 2
	flags			= IPv4
}
```

## 启动tftp服务

	service xinted start
	service xinted restart

检查tftp服务是否启动：

	netstat -a | grep tftp

## 测试命令

PC-LINUX系统的操作命令 **tftp [-v][-m mode] [host [port]] [-c command]**

	tftp 172.16.104.191 -c get test.txt

PC-windows系统操作命令 **TFTP [-i] host [GET | PUT] source [destination]**

	tftp 172.16.104.191 get test.txt

U-boot命令 **tftp [address] filename**，默认是写到内核启动的位置

	tftp text.txt

ARM-linux busybox 上的tftp命令 **tftp [OPTIONS] HOST [PORT]**

	tftp -rg 172.16.104.191 text.txt
	tftp -lp 172.16.104.191 text.txt

## 小结

没啥好说的。
