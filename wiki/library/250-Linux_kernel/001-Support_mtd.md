>1. [0 前提](#0前提 "0 前提")
1. [1 设置环境变量](#1设置环境变量 "1 设置环境变量")
1. [2 内核配置](#2内核配置 "2 内核配置")
1. [3 保存退出](#3保存退出 "3 保存退出")

# Linux Kernel Support Mtd and Support Spi Nor Flash for His3536(sfc350)

## 0 前提
Hisi提供的内核版本3.10.0, sfc350控制器代码已经支持。

## 1 设置环境变量
	export ARCH=arm
	export CROSS_COMPILE=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-

## 2 内核配置
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

## 3 保存退出  
保存配置：  
 	![Save](assets/002/001-b20a6d59.png "Save")  
