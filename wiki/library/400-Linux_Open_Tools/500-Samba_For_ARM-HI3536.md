
# ARM(HI3536) 系统支持samba服务

## 源码来源

1. 从samba开源项目提供的网站上下载最新源码：**https://download.samba.org/pub/samba/** samba-4.5.1.tar.gz

2. 使用公司已经提交在库的代码：**172.16.8.9:29418/sysdev/packages/toolslib** samba-3.0.33

## 配置HI3536的编译环境

HI3536用的交叉编译工具链最新的是： **/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-**
在交叉编译时需要指定 **--cross-compile** 选项为这个交叉工具命令。

## 编译
samba 4.0以后使用了新的基于 **waf** 的编译系统。想要了解更多关于 **waf** 的信息请访问： **https://wiki.samba.org/index.php/Waf**
**http://www.freehackers.org/~tnagy/wafbook/single.html** 这个网站介绍waf的用法，但是经常打不开，就是说明waf是用python脚本写的，放在google code上的，怎么下载安装怎么的。
所以接下来也会分别介绍一下4.0以后和4.0以前版本各自的编译方法：


### 交叉编译samba-3.6.25

```sh
cd source3/
./autogen.sh
./configure --without-krb5 --without-ldap --without-ads \
  --disable-cups --enable-swat=no --with-winbind=no \
  --target=arm-linux-gnueabi --host=arm-linux \
  CC=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-gcc \
  --with-configdir=/etc \
  samba_cv_CC_NEGATIVE_ENUM_VALUES=yes \
  libreplace_cv_HAVE_GETADDRINFO=no \
  ac_cv_file__proc_sys_kernel_core_pattern=yes
make
make install
```

编译完成之后在samba-3.6.25/install目录下会生成一下文件：

	bin  include  lib  private  sbin  share  var

在sbin目录下的 **nmbd smbd** 即是最终要得到的服务程序。

## 配置smb.conf
编译时指定--with-configdir=/etc，那么配置文件就可以放到/etc下了。

## 把smb服务必须的程序放到设备上

把install目录下的所有文件都拷贝到设备的/usr/local下

	tar -czvf install.tar.gz install
	把install.tar.gz上传到设备上/usr目录下
	mkdir -p /usr/local/samba
	tar -xf install.tar.gz
	mv install/* local/samba

## 启动smb服务

	/usr/local/samba/sbin/smbd -D
	/usr/local/mamba/sbin/nmbd -D

查看启动是否成功： ps | grep mbd

如果没有成功可以执行： /usr/local/sbin/smbd -i -d 8  将会打印出相应的错误信息

## 访问测试

	ifconfig 查看网络ip

	然后

## 相关问题

启动服务时报库不存在

	/usr/local/sbin # ./smbd -D
	./smbd: can't load library 'libtalloc.so.2'

	解决：export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/samba/lib

启动服务是报一下错误，导致服务无法启动

* PANIC (pid 848): Could not init smbd's messaging context.


* lp_servicenumber: couldn't find printers


* ERROR: failed to setup guest info.

* ERROR: Failed to initialise messages database: No such file or directory

![Failed to initialise messages database](assets/400/500-0b24fa88.png "Failed to initialise messages database")


### 交叉编译samba-4.5.1(失败)

waf编译工具路径：**samba-4.5.1/buildtools/bin/waf**

	export PATH=$HOME/6-laboratory/samba-4.5.1/buildtools/bin:$PATH

因为没有目标机虚拟工具，所以只能使用--cross-answers方式来编译，遇到问题后 **我并不知道该怎么来该对，所以我选择暂时放弃使用waf来编译最新的samba系统，有时间再搞**

以下命令仅留作参考，并没有意义：

	CC=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-gcc /waf configure --cross-compile --cross-answers=hi3536.txt --prefix=$HOME/6-laboratory/samba-4.5.1/install

	cd samba-4.5.1/
	mkdir install
	./configure configure \
	--cross-compile=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi- \
	--hostcc=/opt/arm-hisiv300-linux/bin/arm-hisiv300-linux-uclibcgnueabi-gcc \
	--prefix=$HOME/6-laboratory/samba-4.5.1/install
