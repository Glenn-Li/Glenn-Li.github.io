<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
# Linux Kernel Reseverd Memory
>1. [第一种方法调用Reserve方法](#第一种方法调用Reserve方法 "第一种方法调用Reserve方法")
1. [第二种方法 RESERVEDMEM_OF_DECLARE](#第二种方法 RESERVEDMEM_OF_DECLARE "第二种方法 RESERVEDMEM_OF_DECLARE")
	1. [在 DTS 中设置 reserved-memory 节点](#在 DTS 中设置 reserved-memory 节点 "在 DTS 中设置 reserved-memory 节点")
	1. [在驱动中使用](#在驱动中使用 "在驱动中使用")
1. [小结](#小结 "小结")
<!-- /TOC -->

# Linux Kernel Reseverd Memory

有的时候我们需要保留一块连续的内存给驱动用，比如DMA，CMA和公司产品上在用的pcie mca，此时我们就需要在内核启动过程中的较早时候先预留出这么一块内存。

保留一块内存可以用两种方法，第一种方式针对较老的内核版本，在arm_machine_init的时候调用reserve方法，驱动中直接extern预留内存的地址和大小变量；第二种方式针对较新内核版本，在DeviceTree中先预留好一块内存，在驱动中使用 **RESERVEDMEM_OF_DECLARE** 宏设置回调函数获取设置的预留内存地址和大小。

两种方法有个优缺点：

第一种方法预留的内存是动态分配的，所以大小可以方便修改，甚至可以从u-boot bootargs参数中传入，但是第二种方法我暂时只会设置固定位置内存，能不能设置成动态的分配我需要在研究研究；
第二种方法使用 **RESERVEDMEM_OF_DECLARE** 宏来获取预留内存地址和大小，相比第一种直接extern传递参数的方式使用上更加规范，而且降低了模块耦合性。


## 第一种方法调用Reserve方法

*以后再说吧*

## 第二种方法 RESERVEDMEM_OF_DECLARE

有关DeviceTree Reserved memory的文档可以查阅：Documentation/devicetree/bindings/reserved-memory/reserved-memory.txt

### 在 DTS 中设置 reserved-memory 节点

在跟节点中增加reserved-memory子节点：

```sh
/ {
    .....
    reserved-memory {
        #address-cells = <1>;
        #size-cells = <1>;
        ranges;            
        mca_reserved: mca@b8000000 {
                compatible = "hisi,mca";
                reg = <0xb8000000 0x4000000>;
        };  
    };
};
```

### 在驱动中使用

定义用 **RESERVEDMEM_OF_DECLARE** 宏，设置回调函数

```c
static unsigned int mca_mem_start;
static unsigned int mca_mem_size;

static int __init rmem_mca_setup(struct reserved_mem *rmem)
{                   
        /*unsigned long node = rmem->fdt_node;*/

        /*rmem->ops = &rmem_dma_ops;*/
        printk(KERN_ERR "yqli-- rmem_mca_setup\n");
        mca_debug("Reserved memory: created MCA reserved memory at %pa, size %ld MiB\n",
                &rmem->base, (unsigned long)rmem->size / SZ_1M);

        mca_mem_start = rmem->base;
        mca_mem_size = rmem->size;

        return 0;
}                   
RESERVEDMEM_OF_DECLARE(mca, "hisi,mca", rmem_mca_setup);
```

## 小结

桔梗？
