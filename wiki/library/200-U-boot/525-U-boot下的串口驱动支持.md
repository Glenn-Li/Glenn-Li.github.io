<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
# U-boot 串口驱动支持并分析
>1. [U-boot 配置支持串口驱动](#U-boot 配置支持串口驱动 "U-boot 配置支持串口驱动")
1. [串口初始化流程](#串口初始化流程 "串口初始化流程")
	1. [serial_init() 函数基本逻辑](#serial_init() 函数基本逻辑 "serial_init() 函数基本逻辑")
1. [printf函数把日志打印到console口上的基本逻辑](#printf函数把日志打印到console口上的基本逻辑 "printf函数把日志打印到console口上的基本逻辑")
1. [小结](#小结 "小结")
<!-- /TOC -->

# U-boot 串口驱动支持并分析

## U-boot 配置支持串口驱动

HI3536 用的串口控制器是serial_pl01x，需要支持这个驱动，需要在 **include/configs/svr2730.h** 文件中定义配置宏：

```java
#define CONFIG_BAUDRATE         115200
.....
/* serial driver */
#define CONFIG_PL011_SERIAL 1
/* If we use RS485, should consider whether need support RS485 Rx/Tx switch */
#define CONFIG_SERIAL_485_RXTX_SWITCH 1
#define CONFIG_SWITCH_GPIO		54 // gpio6-6:6*8+6

/* due to bootrom, hi3536 use APB bus clock
 * UART is on APB, and PERI_AXI : APB is 2 : 1, so we set
 * UART clock 1/2 PERI_AXI bus_clk, but default is 2MHz.
 */
#define CONFIG_PL011_CLOCK	(CFG_CLK_BUS / 2)

#define CONFIG_PL01x_PORTS	\
{(void *)UART0_REG_BASE, (void *)UART1_REG_BASE, \
	(void *)UART2_REG_BASE, (void *)UART3_REG_BASE}

/* Console port number */
/* #define CONSOLE_PORT CONFIG_CONS_INDEX in drivers/serial/serial_pl01x.c */
#define CONFIG_CONS_INDEX	0

#define CFG_SERIAL0                  UART0_REG_BASE

/* accoding to CONFIG_CONS_INDEX */
#define CONFIG_CUR_UART_BASE          CFG_SERIAL0
```

**serial_pl01x驱动代码路径：drivers/serial/serial_pl01x.c**

*待会分析它。*

## 串口初始化流程

start_armboot() -> Call init_sequense -> serial_init(): *drivers/serial/serial_pl01x.c* 	

	265:		serial_init,		/* serial communications setup */

### serial_init() 函数基本逻辑

1. First, disable everything.
2. Set baud rate(115200)
3. Set the UART to be 8 bits, 1 stop bit, no parity, fifo enabled.
4. Finally, enable the UART

*源码如下*
```c
int serial_init (void)
{
	unsigned int temp;
	unsigned int divider;
	unsigned int remainder;
	unsigned int fraction;

	/*
	 ** First, disable everything.
	 */
	IO_WRITE (port[CONSOLE_PORT] + UART_PL011_CR, 0x0);

	/*
	 ** Set baud rate
	 **
	 ** IBRD = UART_CLK / (16 * BAUD_RATE)
	 ** FBRD = ROUND((64 * MOD(UART_CLK,(16 * BAUD_RATE))) / (16 * BAUD_RATE))
	 */
	temp = 16 * baudRate;
	divider = CONFIG_PL011_CLOCK / temp;
	remainder = CONFIG_PL011_CLOCK % temp;
	temp = (8 * remainder) / baudRate;
	fraction = (temp >> 1) + (temp & 1);

	IO_WRITE (port[CONSOLE_PORT] + UART_PL011_IBRD, divider);
	IO_WRITE (port[CONSOLE_PORT] + UART_PL011_FBRD, fraction);

	/*
	 ** Set the UART to be 8 bits, 1 stop bit, no parity, fifo enabled.
	 */
	IO_WRITE (port[CONSOLE_PORT] + UART_PL011_LCRH,
		  (UART_PL011_LCRH_WLEN_8 | UART_PL011_LCRH_FEN));

	/*
	 ** Finally, enable the UART
	 */
	IO_WRITE (port[CONSOLE_PORT] + UART_PL011_CR,
		  (UART_PL011_CR_UARTEN | UART_PL011_CR_TXE |
		   UART_PL011_CR_RXE));

	return 0;
}
```

## printf函数把日志打印到console口上的基本逻辑

printf()函数定义在 common/console.c 文件中，从名字看就知道这是个console打印的功能，那还有什么好解释的，但是本着打破沙锅问到底的原则，我觉得我还是要看看，万一这个console名字只是个幌子呢。

真点进去看了，我X，是不是有点太简单了, printf 分析了一下参数，组织成字符串，调用 serial_pl01x.c 里定义的 puts()函数，收工，其中的一下判断就不要在乎了，详细代码如下：

```c
void puts(const char *s)
{
#ifdef CONFIG_SILENT_CONSOLE
	if (gd->flags & GD_FLG_SILENT)
		return;
#endif

#ifdef CONFIG_DISABLE_CONSOLE
	if (gd->flags & GD_FLG_DISABLE_CONSOLE)
		return;
#endif

	if (gd->flags & GD_FLG_DEVINIT) {
		/* Send to the standard output */
		fputs(stdout, s);
	} else {
		/* Send directly to the handler */
		serial_puts(s);
	}
}

void printf(const char *fmt, ...)
{
	va_list args;
	char printbuffer[CONFIG_SYS_PBSIZE];

	va_start(args, fmt);

	/* For this to work, printbuffer must be larger than
	 * anything we ever want to print.
	 */
	vsprintf(printbuffer, fmt, args);
	va_end(args);

	/* Print the string */
	puts(printbuffer);
}
```

## 小结

每次都要写这个小结，但是知识点都在前面说了，这里真要写的话就是重新抄一遍前面的内容咯，如果想增加篇幅的话倒是个不错的选择。。。阿西吧
