<!-- TOC depthFrom:0 depthTo:4 withLinks:1 updateOnSave:1 orderedList:1 -->
# Linux Kernel Support Pcie Host Bridge Driver for His3536
>1. [前提](#前提 "前提")
1. [设置环境变量](#设置环境变量 "设置环境变量")
1. [內核配置](#內核配置 "內核配置")
1. [配置支持MCA(Multi-core Chip Architecture)](#配置支持MCA(Multi-core Chip Architecture) "配置支持MCA(Multi-core Chip Architecture)")
1. [编译](#编译 "编译")
<!-- /TOC -->

# Linux Kernel Support Pcie Host Bridge Driver for His3536

## 前提
Hisi提供的内核版本3.10.0, pcie主桥代码已经支持。

## 设置环境变量
	export ARCH=arm
	export CROSS_COMPILE=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-

## 內核配置

*进入到 [Bus Support]*
![Bus Support](assets/250/500-ea003581.png)

*选择hisi PCIE support*
![PCIE Support](assets/250/500-1bfe8ac2.png)
![Hisi PCIE-1](assets/250/500-e8411894.png)

*PCIE配置，一般是有默认配置，所以只查看一下就好了*
![PCIE config](assets/250/500-23ed2cfc.png)

保存退出。

## 配置支持MCA(Multi-core Chip Architecture)

进入到 [Device Drivers]  ---> [KEDACOM Linux Support Package(KLSP)]后选择：
![MCA Support](assets/250/500-ca134408.png)

再进入到v1.1项：
![MCA v1.1](assets/250/500-a592cc4a.png)

选上里面唯一的一项：
![Hi3536 mca-rc v1.1](assets/250/500-3b9d2f2d.png)

保存退出。

## 编译

make uImage

PCI Bus 模块编译后生成目标文件列表：

```tree
$drivers/pci$ tree -f | grep "\.o"
├── ./access.o
├── ./built-in.o
├── ./bus.o
│   ├── ./hipcie/built-in.o
│   ├── ./hipcie/hipcie.o
│   └── ./hipcie/pcie.o
├── ./host-bridge.o
├── ./irq.o
├── ./pci-driver.o
│   │   ├── ./pcie/aer/aerdriver.o
│   │   ├── ./pcie/aer/aerdrv_core.o
│   │   ├── ./pcie/aer/aerdrv_errprint.o
│   │   ├── ./pcie/aer/aerdrv.o
│   │   ├── ./pcie/aer/built-in.o
│   ├── ./pcie/aspm.o
│   ├── ./pcie/built-in.o
│   ├── ./pcie/pcieportdrv.o
│   ├── ./pcie/pme.o
│   ├── ./pcie/portdrv_bus.o
│   ├── ./pcie/portdrv_core.o
│   └── ./pcie/portdrv_pci.o
├── ./pci.o
├── ./pci-sysfs.o
├── ./probe.o
├── ./proc.o
├── ./quirks.o
├── ./remove.o
├── ./rom.o
├── ./search.o
├── ./setup-bus.o
├── ./setup-irq.o
├── ./setup-res.o
├── ./slot.o
├── ./syscall.o
├── ./vpd.o
```

MCA 模块编译后生成目标文件列表：

```tree
$drivers/klsp/drivers/mca/mca_v1.1$ ls *.o
built-in.o  hi3536_host.o  mca_host.o
```

未完待续...
