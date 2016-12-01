
# Hello World

基本是套路了，开始接触一个新技术、新变成语言的时候，都要来先说一次“Hello World!”

## Hello world 内核模块源代码

```c
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");

static int __init hello_init(void)
{
	printk(KERN_ALERT "Hello, world!\n");
	return 0;
}

static void __exit hello_exit(void)
{
	printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(hello_init)
module_exit(hello_exit)
```

## Hello world 编译 Makefile

### 交叉编译-Makefile，生成ARM target设备模块代码

```shell
ifeq ($(KERNELRELEASE),)
	KERNELDIR ?= /usr/src/linux-2.6.37
	PWD := $(shell pwd)

modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions *.order *.symvers

.PHONY: modules modules_install clean

else
	obj-m := hello_world.o
endif
```
查看编译生成的hello_world.ko文件属性：

	$ file hello_world.ko
	hello_world.ko: ELF 32-bit LSB relocatable, ARM , version 1 (SYSV), not stripped

### 编译X86的模块的Makefile

```sh
ifeq ($(KERNELRELEASE),)
	KERNELDIR ?= /lib/modules/$(shell uname -r)/build
	PWD := $(shell pwd)

modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions *.order *.symvers

.PHONY: modules modules_install clean

else
	obj-m := hello_world.o
endif
```

查看编译生成的hello_world.ko文件属性：

	$file hello_world.ko
	hello_world.ko: ELF 32-bit LSB relocatable, Intel 80386, version 1 (SYSV), not stripped

## 小结

小结是谁？

Who is XiaoJie?
