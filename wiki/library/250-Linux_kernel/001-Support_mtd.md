<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
>1. [1 前提](#1 前提 "1 前提")
1. [2 设置环境变量](#2 设置环境变量 "2 设置环境变量")
1. [3 内核配置](#3 内核配置 "3 内核配置")
1. [4 保存退出](#4 保存退出 "4 保存退出")
1. [5 编译](#5 编译 "5 编译")
1. [6 启动内核加载驱动的打印信息](#6 启动内核加载驱动的打印信息 "6 启动内核加载驱动的打印信息")
1. [7 proc和sys文件系统下的信息](#7 proc和sys文件系统下的信息 "7 proc和sys文件系统下的信息")
	1. [proc fs](#proc fs "proc fs")
	1. [sysfs](#sysfs "sysfs")
1. [8 hisi3536 flash挂载信息](#8 hisi3536 flash挂载信息 "8 hisi3536 flash挂载信息")
1. [9 设备上默认有的flash操纵相关命令工具](#9 设备上默认有的flash操纵相关命令工具 "9 设备上默认有的flash操纵相关命令工具")
1. [10 常见问题](#10 常见问题 "10 常见问题")
	1. [jffs2文件系统挂载异常报错](#jffs2文件系统挂载异常报错 "jffs2文件系统挂载异常报错")
<!-- /TOC -->

# Linux Kernel Support Mtd and Support Spi Nor Flash for His3536(sfc350)

## 1 前提
Hisi提供的内核版本3.10.0, sfc350控制器代码已经支持。

## 2 设置环境变量
	export ARCH=arm
	export CROSS_COMPILE=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-

## 3 内核配置
执行 make menuconfig 进入到配置界面，选择 Device Drivers：

![Device Drivers](assets/002/001-93328f84.png "Device Drivers")

选中 MTD 驱动支持：

![Memory Technology Device (MTD) support](assets/002/001-93328f84.png "Memory Technology Device (MTD) support")

选中以下选项：

![Select all](assets/002/001-7141440e.png "Select All")

进入到Self-contained MTD device drivers:

![Self-contained MTD device drivers](assets/002/001-e926d0d5.png "Self-contained MTD device drivers")

选中以下选项：

![Select sfc350](assets/002/001-023bde1f.png "Select sfc350")

正确设置基址和片选：

![Config sfc350](assets/002/001-1511e7b9.png "Config sfc350")

## 4 保存退出
保存配置：

![Save](assets/002/001-b20a6d59.png "Save")

## 5 编译
执行 make uImage 编译生成uImage内核镜像。
在 *linux-3.10.0/drivers/mtd/devices/hisfc350* 目录下会生成三个.o文件，built-in.o  hisfc350.o  hisfc350_spi_ids.o

## 6 启动内核加载驱动的打印信息
内核启动时的mtd驱动加载信息如下：

![Probe sfc350](assets/250/001-2be0e503.png "Probe sfc350")

## 7 proc和sys文件系统下的信息

### proc fs
/proc/mtd 文件说明分区信息：

![/proc/mtd文件](assets/250/001-3d5d8238.png "proc/mtd文件")

/proc/partitions 文件中也会列出相应的分区信息：

![/proc/partitions文件](assets/250/001-8ef95614.png "proc partitions 文件")

### sysfs
/sys/devices/platform/hi_sfc：sfc350 平台设备目录，sfc350把自身注册为平台设备

![platform device](assets/250/001-9521d3e3.png "platform device")

/sys/bus/platform/drivers/hi_sfc: sfc350 平台设备驱动目录，sfc350相应的平台驱动

![platform Driver](assets/250/001-31d0265a.png "platform Driver")

/sys/block：显示设备中全部的块设备，spi nor flash也属于块设备，所以在这里也会生成相应的链接文件

![block device infomations](assets/250/001-959c637d.png "block device infomations")

## 8 hisi3536 flash挂载信息
挂载命领调用流程如下：  文件名函数
	/etc/init.d/rcS(. /etc/init.d/common.sh) -> /etc/init.d/common.sh(mount_mtd) -> /etc/conf/env.conf(mount_mtd)

/etc/conf/env.conf: 会把业务用到的单个分区挂载到/usr,/usr/bin和/usr/config三个目录
```sh
#!/bin/sh

KERNEL_VERSION=`uname -r`

WORK_PATH=/usr
WORK_BIN_PATH=${WORK_PATH}/bin
WORK_LIB_PATH=${WORK_PATH}/lib
WORK_LOG_PATH=${WORK_PATH}/log
WORK_CONFIG_PATH=${WORK_PATH}/config
WORK_CONFIG_SYSTEM_PATH=${WORK_CONFIG_PATH}/system

###########Customized the common.sh############
HOSTNAME=KEDACOM
USR_MANAGE_ENABLE=y
SYSLOGD_ENABLE=y
SYSLOGD_LOGFILE=/var/log/message
SYSLOGD_MAXSIZE=128
KLOGD_ENABLE=y
LDCONFIG_ENABLE=n
###########Do Early Init#######################
early_init()
{
	#make dev nodes
	echo "early init done"
	#[ ! -c /dev/ppp ] && mknod /dev/ppp c 108 0
}

#$1: partition name
#$2: mount point
mount_jffs2_partition()
{
	PART_NAME=$1
	MNT_POINT=$2

	[ ! -d "$MNT_POINT" ] && mkdir -p "$MNT_POINT"

	MTD_NR=`cat /proc/mtd | grep "$PART_NAME" | cut -c4`

	[ -z "$MTD_NR" ] && exit

	mount -t jffs2 /dev/mtdblock$MTD_NR $MNT_POINT >/dev/null 1>&2
	if [ "$?" -ne 0 ];then
		flash_eraseall -j /dev/mtd$MTD_NR
		mount -t jffs2 /dev/mtdblock$MTD_NR $MNT_POINT >/dev/null 1>&2
	fi
}

cmdline_fetch_member()
{
	local find

	for member in `cat /proc/cmdline`
	do
		has=`echo $member | grep "$1\>"`
			if [ -n "$has" ];then
				 find=`echo $member | cut -f2 -d "=" $member - 2>/dev/null`
				echo $find
			fi
	done
}

mount_mtd()
{
    #MUST MOUNT MTD PARTITION HERE!!
	#MUST MOUNT /usr /usr/bin
	pidx=$(cmdline_fetch_member "partindex")
	[ -z $pidx ] && exit
	if [ "$pidx" -eq "0" ];then
		mount_jffs2_partition usr /usr
	fi
	mount_jffs2_partition conf"$pidx" /usr/config
	mount_jffs2_partition app"$pidx" /usr/bin
}

setup_netdevice()
{
	NET_DEV_CONF_DIR=$1

	[ -z "$NET_DEV_CONF_DIR" ] && exit 0
	IFUP_BIN=`which ifup`
	#bring up Network
	if [ ! -d "$NET_DEV_CONF_DIR" ];then
		 mkdir -p "$NET_DEV_CONF_DIR"
	fi

	if [ ! -f $NET_DEV_CONF_DIR/interfaces ];then
		 cp /etc/network/interfaces $NET_DEV_CONF_DIR
	fi

	#config  netdevice ethxx
	NFS_FIND=`cat /proc/cmdline | grep "root=/dev/nfs"`
	if [ -z "$NFS_FIND" ];then
		$IFUP_BIN -a -i $NET_DEV_CONF_DIR/interfaces
	fi
}
```

## 9 设备上默认有的flash操纵相关命令工具
flash_eraseall： 分区擦除，只能操作整个分区，不支持address定位擦除

![flash_eraseall](assets/250/001-6c0ca85f.png "flash_eraseall")

dd： 可以直接与flash进行读写操作

	dd if=/dev/mtdblkck5 of=/tmp/mtd5   # 把第六个分区完整的读取出来，如果分区已经被格式成了jffs2，那么mtd5也是拥有格式的
	dd if=/tmp/mtd5 of=/dev/mtdblkck5	# 直接写分区，升级app时，可以直接把app.img想这样写入，前提是要先erase分区

## 10 常见问题

### jffs2文件系统挂载异常报错
有时候挂载Jffs2分区会有类似以下提醒或者错误：

![no data nodes found](assets/250/001-94cd2bd4.png "no data nodes found")

一般是因为系统异常断电，分区内有数据不完整，将不需要写入数据的分区挂载成只读模式可以，需要写入数据的分区在写完数据后及时sync。
