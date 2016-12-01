<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
# Klsp中串口初始化配置
>1. [内核配置串口控制器驱动](#内核配置串口控制器驱动 "内核配置串口控制器驱动")
1. [SVR2730 还需要配置USB转串口驱动](#SVR2730 还需要配置USB转串口驱动 "SVR2730 还需要配置USB转串口驱动")
1. [KLSP 配置](#KLSP 配置 "KLSP 配置")
	1. [配置板级串口数量](#配置板级串口数量 "配置板级串口数量")
	1. [配置board_serials](#配置board_serials "配置board_serials")
	1. [注册RS485方向控制接口](#注册RS485方向控制接口 "注册RS485方向控制接口")
	1. [代码实现](#代码实现 "代码实现")
1. [小结](#小结 "小结")
<!-- /TOC -->

# Klsp中串口初始化配置

主要完成驱动配置支持，和在KLSP中填充board_serials数组， 如果是RS485还要注册读写方向控制接口。工作将分成以下几个步骤：

## 内核配置串口控制器驱动

His3536 用的串口控制器是 amba-pl011 (drivers/tty/serial)

**确认配置上已经选中了该驱动**

1. 从配置首页进入驱动配置页：

![从配置首页进入驱动配置页](assets/250/200-3f8b9f6b.png)

2. 选择字符设备：

![选择字符设备](assets/250/200-4aebb829.png)

3. 选择serial设备：

![选择serial设备](assets/250/200-87868413.png)

4. 选中如下配置：

![选中如下配置](assets/250/200-66fad40c.png)

以上选中就说明已经配置好了串口驱动支持。保存退出，编译内核，好了之后可以在 **drivers/tty/serial** 目录下看到生成的 .o 文件。

## SVR2730 还需要配置USB转串口驱动

该转换芯片的驱动是放在了KLSP下，所以按照以下步骤配置：

1. 同样进入到驱动配置页

2. 选择KLSP

![选择KLSP](assets/250/200-d4b4e300.png)

3. 选择USB Serial support

![选择USB Serial support](assets/250/200-6fc8bb10.png)

4. 选中对应的转换芯片驱动

![选中对应的转换芯片驱动](assets/250/200-aaac7562.png)

## KLSP 配置

配置好了驱动之后，还要在KLSP中进行板级串口配置，有以下几个地方要配置：

### 配置板级串口数量

进入到目录 **drivers/klsp/boards/kdm/svr2730**

修改 **brd_cfg.h** 中的串口数量：

```c
/* SERIAL Number */
#define CFG_SERIAL_NUM          10		/* Svr2730是有两个RS485（原生）和8个RS232（其中两个是原生，6个是USB转的） */
```

### 配置board_serials

以下代码配置都是在 **drivers/klsp/boards/kdm/svr2730/brd_cfg.c** 文件中完成。

```c
......
/* 定义board_serials结构体 */
#if defined(CFG_SERIAL_NUM) && (CFG_SERIAL_NUM > 0)
struct serial_info board_serials[CFG_SERIAL_NUM];
#endif
......
/* board config Init, later brd_update will be called to update some params */
static int __init board_init(void)
{
	.....
	board_cfg.serial_num = CFG_SERIAL_NUM;
	.....
	/* init serial info */
#if defined(CFG_SERIAL_NUM) && (CFG_SERIAL_NUM > 0)
	memset(&board_serials, 0, sizeof(board_serials));
	for (i = 0; i < CFG_SERIAL_NUM; i++) {
		board_serials[i].no = i;
		board_serials[i].type = SERIAL_RS232;
	}

	/* index max is 9 */
	board_serials[index].usage = SERIAL_CONSOLE;	/* Svr2730把UART2用做了调试串口 *//**/;
	strncpy(board_serials[index++].name, "/dev/ttyAMA0", MOST_STR_MAX_LEN - 1);

#ifndef CONFIG_ARCH_HI3536_SLAVE
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyAMA1", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyAMA2", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyAMA3", MOST_STR_MAX_LEN - 1);
#endif

	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyXRUSB0", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyXRUSB1", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyXRUSB2", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyXRUSB3", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyXRUSB4", MOST_STR_MAX_LEN - 1);
	board_serials[index].usage = SERIAL_COMMU;
	strncpy(board_serials[index++].name, "/dev/ttyXRUSB5", MOST_STR_MAX_LEN - 1);

#endif
}
```

以上就是基本的串口配置。其中的UART0 用作调试串口，但是有一个问题是：在原理图上没有找到UART0的直接引出引脚，只有RS232的引出。

### 注册RS485方向控制接口

其中UART2、3使用的RS485，他们需要注册相应的方向控制接口，在 **drivers/klsp/boards/kdm/svr2730/brd_gpio.c** 文件中配置。

配置 RS485 方向控制接口的基本步骤如下(所有的操作都是在)：

1. 根据原理图确定哪些UART用作RS485，是否需要方向切换，阅读SVR2730的原理图可知 UART2 和 UART3 用作了RS485，并且都需要方向控制。

![SVR2730 RS485原理图](assets/250/200-00350e20.png)

2. 确定方向控制GPIO及其相应的GPIO复用寄存器，方向寄存器，功能寄存器，并且初始化，代码由uart_txrx_swi_init()实现。也是通过原理可知GPIO7_2作为UART2的方向控制，GPIO7_3给UART3。

确定GPIO引脚：

*各个UART的方向控制GPIO*

![各个UART的方向控制GPIO](assets/250/200-85140d1d.png)

确定GPIO引脚复用寄存器，阅读HI3536芯片手册，找到 **2.5 管脚复用寄存器** 节，首先设置UART的RXD和TXD引脚复用：

*UART2的RXD和TXD引脚复用*

![UART2的RXD和TXD引脚复用](assets/250/200-18a6cf41.png)

*UART3的RXD和TXD引脚复用*

![UART3的RXD和TXD引脚复用](assets/250/200-d1d4dca4.png)

设置方向控制GPIO引脚复用为GPIO(**2.5 管脚复用寄存器** 节)：

*GPIO7_2.3复用寄存器*

![GPIO7_2.3复用寄存器](assets/250/200-f5bc7183.png)

*配置为0作为GPIO用*

![配置为0作为GPIO用](assets/250/200-84e3f070.png)

设置GPIO为方向为输出，默认低电平(**13.5.4 GPIO 寄存器概览** 节)：

*设置GPIO方向为输出模式*

![GPIO基址](assets/250/200-0f1c5f7d.png)

![GPIO7基地址](assets/250/200-03cc6187.png)

*基地址加上该偏移*

![方向控制寄存器偏移](assets/250/200-6b92d343.png)

*设置为output模式*

![设置为输出模式](assets/250/200-2969970c.png)

最后就是一个写GPIO数据寄存器，使GPIO默认为低电平，表示UART默认是接收模式，之后的RS485写操作时，需要先切换成Tx模式（拉高），写完后切换成Rx模式（拉低）

*HI3536的数据寄存器读写很有意思，需要先设置掩码，在设置值，而且低2 bits 不使用*

![GPIO数据寄存器偏移](assets/250/200-b81c53f8.png)

3. 注册控制接口到串口驱动

注册相应UART号的控制接口，RS485写操作（默认是读模式，所以驱动中读不需要操作）时会调用这个接口来设置方向切换GPIO的电平。
```c
serial_his_dir_register(2, p2_uart_txrx_switch );
serial_his_dir_register(3, p3_uart_txrx_switch );
```
### 代码实现

*引用头文件*
```c
#include <mach/io.h>
#include <linux/serial_core.h>
```

*接口定义以及注册*
```c
#ifndef CONFIG_ARCH_HI3536_SLAVE
#define P2_SWI_GPIO_BANK 		7
#define P2_SWI_GPIO_INDEX		2
#define P2_GPIO_PINMUX_INDEX		0x138
#define P2_GPIO_PINMUX_VAL		0

#define P3_SWI_GPIO_BANK 		7
#define P3_SWI_GPIO_INDEX		3
#define P3_GPIO_PINMUX_INDEX		0x13C
#define P3_GPIO_PINMUX_VAL		0


#define GPIO_BASE 			0x12150000
#define GPIO_PINMUX_BASE 		0x120F0000

/* port 2 Rx/Tx switch */
static void p2_uart_txrx_switch(int mode)
{
	/*master or slave */

	volatile unsigned char *gpio_bank_base = \
		(volatile unsigned char *)IO_ADDRESS(GPIO_BASE + \
		0x10000 * P2_SWI_GPIO_BANK +(1 << (P2_SWI_GPIO_INDEX + 2)));
	if(mode == 1)
		writeb(0xff, gpio_bank_base);
	else
		writeb(0, gpio_bank_base);

}

/* port 0 Rx/Tx switch */
static void p3_uart_txrx_switch(int mode)
{
	/*master or slave */
	volatile unsigned char *gpio_bank_base = \
		(volatile unsigned char *)IO_ADDRESS(GPIO_BASE + \
		0x10000 * P3_SWI_GPIO_BANK +(1 << (P3_SWI_GPIO_INDEX + 2)));
	if(mode == 1)
		writeb(0xff, gpio_bank_base);
	else
		writeb(0, gpio_bank_base);

}

static void uart_txrx_swi_init(void)
{
	volatile unsigned int *addr_base = NULL;
	unsigned int reg = 0;
	volatile unsigned char *gpio_bank_base= NULL;

	/*set uart2 tx,rx pinmux master,slave,and sigle*/
	addr_base = (volatile unsigned int *)IO_ADDRESS(GPIO_PINMUX_BASE + 0x140);
	writel(1, addr_base);
	addr_base = (volatile unsigned int *)IO_ADDRESS(GPIO_PINMUX_BASE + 0x144);
	writel(1, addr_base);

	/*set uart3 tx,rx pinmux master,slave,and sigle*/
	addr_base = (volatile unsigned int *)IO_ADDRESS(GPIO_PINMUX_BASE + 0x164);
	writel(1, addr_base);
	addr_base = (volatile unsigned int *)IO_ADDRESS(GPIO_PINMUX_BASE + 0x148);
	writel(1, addr_base);

	/*set gpio pinmux for P2_GPIO*/
	addr_base = (volatile unsigned int *)\
		    IO_ADDRESS(GPIO_PINMUX_BASE + P2_GPIO_PINMUX_INDEX);
	writel(P2_GPIO_PINMUX_VAL, addr_base);
	/*set gpio dir output for P2_GPIO*/
	addr_base = (volatile unsigned int *)\
		    IO_ADDRESS(GPIO_BASE + P2_SWI_GPIO_BANK * 0x10000 + 0x400);
	reg = readb((volatile unsigned char *)addr_base);
	reg |= (1 << P2_SWI_GPIO_INDEX);
	writeb(reg, (volatile unsigned char *)addr_base);

	/*set gpio low to read default*/
	gpio_bank_base = \
		(volatile unsigned char *)IO_ADDRESS(GPIO_BASE + \
		0x10000 * P2_SWI_GPIO_BANK +(1 << (P2_SWI_GPIO_INDEX + 2)));
		writeb(0, gpio_bank_base);


	/*set gpio pinmux for P3_GPIO*/
	addr_base = (volatile unsigned int *)\
		    IO_ADDRESS(GPIO_PINMUX_BASE + P3_GPIO_PINMUX_INDEX);
	writel(P3_GPIO_PINMUX_VAL, addr_base);

	/*set gpio dir output for P3_GPIO*/
	addr_base = (volatile unsigned int *)\
		    IO_ADDRESS(GPIO_BASE + P3_SWI_GPIO_BANK * 0x10000 + 0x400);
	reg = readb((volatile unsigned char *)addr_base);
	reg |= (1 << P3_SWI_GPIO_INDEX);
	writeb(reg, (volatile unsigned char *)addr_base);

	/*set gpio low to read default*/
	gpio_bank_base = \
		(volatile unsigned char *)IO_ADDRESS(GPIO_BASE + \
		0x10000 * P3_SWI_GPIO_BANK +(1 << (P3_SWI_GPIO_INDEX + 2)));
		writeb(0, gpio_bank_base);
}

static void board_init(void)
{
	printk("RS485 SERIAL CONTROL SWITCH\n");
	/*init serial switch*/
	uart_txrx_swi_init();
	/*sigle or master*/
	/*when slave close the console,you can operate the ttyAMA2*/
	serial_his_dir_register(2, p2_uart_txrx_switch );
	serial_his_dir_register(3, p3_uart_txrx_switch );

}
#endif
```

*加入到执行列表中*
```c
#ifndef CONFIG_ARCH_HI3536_SLAVE
core_initcall(board_init);
#endif
```

## 小结

没啥好总结的，还不懂的话，多看几遍文档，把其中的谬误纠正就好了。
