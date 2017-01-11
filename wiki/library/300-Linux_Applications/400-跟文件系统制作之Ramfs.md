

# 嵌入式linux跟文件系统制作之Ramfs

## 以制作hi3519的内存文件系统为例

```sh
set -x
TMP_RELEASE_DIR=`pwd`
TARGET_WORKSPACE=`pwd`
SRC_DIR=${TARGET_WORKSPACE}/rootfs_3519
TARGET=ramfs_3519

APP_ARM_TOOLCHAIN=/opt/hisi-linux/x86-arm/arm-hisiv500-linux/bin/arm-hisiv500-linux-uclibcgnueabi-

find ${SRC_DIR}/bin -type f -executable | xargs ${APP_ARM_TOOLCHAIN}strip --strip-unneeded
find ${SRC_DIR}/lib -type f -name *.so | xargs ${APP_ARM_TOOLCHAIN}strip --strip-unneeded
find ${SRC_DIR}/lib -type f -name *.so.[0-9]* | xargs ${APP_ARM_TOOLCHAIN}strip --strip-unneeded

cd ${SRC_DIR} && find . | cpio -H newc -o > ${TMP_RELEASE_DIR}/initramfs.img
cd ..
gzip -9 ${TMP_RELEASE_DIR}/initramfs.img && mv ${TMP_RELEASE_DIR}/initramfs.img.gz ${TMP_RELEASE_DIR}/initramfs.img
pwd
mkimage -A arm -O linux -T ramdisk -C none -a 0x82000000 -e 0x82000000 -n cpioInitramfs -d initramfs.img uInitramfs
rm initramfs.img && chmod 777 uInitramfs && mv uInitramfs ${TARGET}
```

## 使用用例

进入到u-boot，设置bootargs中的 "root=/dev/ram" ,在bootcmd中增加tftp 0x82000000 的命令把文件系统下载到内存中（内核也要以这种方式来下载），之后用bootm命令加上内核镜像的内存地址和文件系统镜像的内存地址，就可以把内核启动起来了：

    bootargs=mca_mem=72M console=ttyAMA0,115200n8 rdinit=/linuxrc root=/dev/ram vmalloc=700M host=svr2730 printk.time=y
    bootcmd=tftp 41000000 uImage-hi3536;tftp 42000000 ramfs_3536;bootm 41000000 42000000

## 小结

有时候公司网络环境不好的时候，或者电脑配置低，跑不起虚拟机的时候，就果断切换到内存文件系统上开发吧，整个人的状态都会编号。
