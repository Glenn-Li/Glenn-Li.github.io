>1. [0 前提](#0 前提 "0 前提")
1. [1 设置环境参数](#1 设置环境参数 "1 设置环境参数")
1. [2 修改Makefile文件](#2 修改Makefile文件 "2 修改Makefile文件")
1. [3 添加config文件](#3 添加config文件 "3 添加config文件")
1. [4 添加board下的板级配置](#4 添加board下的板级配置 "4 添加board下的板级配置")
1. [5 u-boot 下的驱动移植](#5 u-boot 下的驱动移植 "5 u-boot 下的驱动移植")
	1. [5.1 DDR参数调试](#5.1 DDR参数调试 "5.1 DDR参数调试")
	1. [5.2 Flash驱动调试](#5.2 Flash驱动调试 "5.2 Flash驱动调试")
	1. [5.3 串口驱动调试](#5.3 串口驱动调试 "5.3 串口驱动调试")
	1. [5.4 I2C驱动移植](#5.4 I2C驱动移植 "5.4 I2C驱动移植")
	1. [5.5 ZL32067 时钟芯片驱动开发](#5.5 ZL32067 时钟芯片驱动开发 "5.5 ZL32067 时钟芯片驱动开发")
	1. [5.6 FPGA加载](#5.6 FPGA加载 "5.6 FPGA加载")

# Porting U-boot-2010-06 for svr2730(his3536)

## 0 前提
u-boot-2010.06版u-boot使用移植过后的代码，里面已经包含了必须的驱动以及命令代码。比如sfc350控制器驱动、ddr配置和基本的u-boot命令。

## 1 设置环境参数
	export ARCH=arm
	export CROSS_COMPILE=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-

## 2 修改Makefile文件
在Makefile文件里增加svr2730的编译配置：

![Makefile](assets/200/500-cb310e95.png "Makefile")

## 3 添加config文件
在/include/configs目录下添加svr2730配置文件：

	include/configs/svr2730.h
	include/configs/svr2730_master.h
	include/configs/svr2730_slave.h
主要关注svr2730.h文件，另外两个文件用在主从CPU启动的时候，目前没有项目在使用。
svr2730.h文件是根据His3536 sdk中的默认配置文件修改而来的，在原先的基础上增加公司的定制选线和修改设备名称为svr2730，根据硬件配置增删驱动支持。

## 4 添加board下的板级配置
在/board/目录下增加板级启动初始化代码和flash分区配置，该代码提供：

	board_init()
	misc_init_r()
	dram_init()
	board_late_init()
	serial_txrx_switch()

以上函数实现。init()这些函数在u-boot启动时由start_armboot()调用。

```
svr2730/
├── board.c
├── board_make.mk
├── config.mk
├── DMEB1-uboot-Hi3536-DDR400M-2GB-A17-1200M-A7-900M-hw-forCustomer.xls
├── DMEB1-uboot-Hi3536-DDR400M-2GB-A17-1200M-A7-900M-hw-RnD-fix-20150330.xls
├── DMEB1-uboot-Hi3536-DDR466M-2GB-A17-1400M-A7-900M-hw.xls
├── Makefile
├── reg_ddr400_2g
```

## 5 u-boot 下的驱动移植
svr2730 需要注意调试DDR参数、Spi Nor Flash 驱动支持、串口驱动调试，需要移植i2c控制器驱动，时钟芯片Zl32067 i2c控制配置驱动和FPGA加载
### 5.1 DDR参数调试
使用Hisi提供的配置表格，使用之前使用的register 配置bin文件，暂时不管（后面补说明）。
### 5.2 Flash驱动调试
仔细分析过驱动，看起来是不用修改的（后面补一个Flash驱动调试说明）。
### 5.3 串口驱动调试
我认为不会有问题（后面补说明）。
### 5.4 I2C驱动移植
添加控制器驱动代码：drivers/i2c/hi35xx_i2c.c

修改i2c驱动编译Makefile：drivers/i2c/Makefile

![I2C Makefile](assets/200/500-1aaf2894.png "i2c Makefile")

板级配置中添加i2c支持：include/configs/svr2730.h
```java
#define CONFIG_I2C
#ifdef CONFIG_I2C
	#define CONFIG_HI35XX_I2C
	#ifdef CONFIG_HI35XX_I2C
		#define CONFIG_HARD_I2C
		#define CONFIG_CMD_I2C
		#define CONFIG_SYS_I2C_SPEED  (100000)
		#define CONFIG_HI_I2C_TX_FIFO 0x8
		#define CONFIG_HI_I2C_RX_FIFO 0x8

		#define TCA9548A_I2C_ADDR 0x70
		//#define SI5341B_I2C_ADDR 0x75
		#define ZL32067_I2C_ADDR 0x74
	#endif
#endif
```
### 5.5 ZL32067 时钟芯片驱动开发
实际就是完成ZL32067的I2C控制，在u-boot启动时通过I2C把ZL32067的寄存器配置好，使其能够产生正确的时钟供DSP使用。
因为不是每个板子都需要这个芯片，而且每个板子上的配置要求也不一样，所以把这个时钟芯片的配置驱动放在板级目录下。
添加ZL32067配置驱动：board/svr2730/si5341b.c 和 board/svr2730/si5341b.h
修改板级目录下相应Makefile。
u-boot启动初始化过程中调用定义的setup接口：

	start_armboot() -> misc_init_r() -> si5341b_setup()

### 5.6 FPGA加载
添加fpgaload命令：board/svr2730/fpga.c
添加编译fpga.c：board/svr2730/Makefile

![Fpga Makefile](assets/200/500-2cf958a8.png "Fpga Makefile")

划分一个flash分区用来存放fpga固件：board/svr2730/board.c
在pinfo分区下面划分一个640KB大小的firmware0分区。
增加初始化函数调用：board/svr2730/board.c

![Call Fpga init](assets/200/500-056f7123.png "Call Fpga init")

升级时获取firmware分区信息：common/kedacom/cmd_update.c

![Modify cmd_update](assets/200/500-9588dbbd.png "Modify cmd_update.c")

板级配置中增加配置，支持fpga加载：include/configs/svr2730.h

![Fpga config](assets/200/500-069339bc.png "FPGA config")

板级配置中增加配置，使系统能调用fpga初始化：include/configs/svr2730.h

![FPGA board_late_init](assets/200/500-fb8d8a94.png "FPGA board_late_init")

	start_armboot() -> board_late_init () -> brd_fpga_init()
在brd_fpga_init()里面要要进行gpio引脚初始化gpio_init()，这些引脚的配置是板级相关的，要根据硬件原理图来设置（有时间的时候重新整理一下结构吧）。根据硬件引脚配置，需要设备两个结构体数组和一组宏定义：fpga_gpio_cfg、fpga_gpio_pinctl 和“FPGA ctrl pin”

	fpga_gpio_cfg：gpio管脚方向和初始值
	fpga_gpio_pinctl：配置相应管脚为gpio模式
	FPGA ctrl pin：方便gpio_read 和 gpio_write 操作而定义的一组管脚
