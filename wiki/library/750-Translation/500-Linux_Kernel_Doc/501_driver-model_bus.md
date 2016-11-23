```txt
Bus Types
总线类型

Definition
定义
~~~~~~~~~~
See the kerneldoc for the struct bus_type.
查看内核文档了解 bus_type 结构。

/* 总线注册接口 */
int bus_register(struct bus_type * bus);

Declaration
声明
~~~~~~~~~~~

Each bus type in the kernel (PCI, USB, etc) should declare one static
object of this type. They must initialize the name field, and may
optionally initialize the match callback.
每个内核中的总线类型（PCI、USB等）应该声明一个静态对象。它们必须初始化 name 字段，
可能会选择性的初始化配对回调。

struct bus_type pci_bus_type = {
       .name	= "pci",
       .match	= pci_bus_match,
};

The structure should be exported to drivers in a header file:
定义的结构体应该导出到驱动头文件中：

extern struct bus_type pci_bus_type;


Registration
注册
~~~~~~~~~~~~

When a bus driver is initialized, it calls bus_register. This
initializes the rest of the fields in the bus object and inserts it
into a global list of bus types. Once the bus object is registered,
the fields in it are usable by the bus driver.
当完成 bus 驱动的初始化就可以调用 bus_register 了。注册函数会初始化 bus 对象中
其余的字段并且把 bus 对象插入到全局 bus 类型列表。只要 bus 对象被注册，bus 驱动
就可以使用它定义的字段。

Callbacks
回调
~~~~~~~~~

match(): Attaching Drivers to Devices
match()：连接驱动和设备
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The format of device ID structures and the semantics for comparing
them are inherently bus-specific. Drivers typically declare an array
of device IDs of devices they support that reside in a bus-specific
driver structure.
设备 ID 结构格式和比较的方法都是特定总线固有的。驱动通常定义一组可以驻留在特定
总线驱动结构体里的设备的设备 ID 号。

The purpose of the match callback is provide the bus an opportunity to
determine if a particular driver supports a particular device by
comparing the device IDs the driver supports with the device ID of a
particular device, without sacrificing bus-specific functionality or
type-safety.
配对回调的目的是在不牺牲总线功能或者类型安全的情况下，让总线有机会通过对比驱动
支持的设备 ID 号组和特定设备的设备 ID 号来确定是否有驱动能支持该设备。

When a driver is registered with the bus, the bus's list of devices is
iterated over, and the match callback is called for each device that
does not have a driver associated with it.
当一个驱动被注册到总线时，总线上的设备列表会被遍历，每一个没有被驱动关联的设备
都会被执行配对回调。


Device and Driver Lists
设备和驱动列表
~~~~~~~~~~~~~~~~~~~~~~~

The lists of devices and drivers are intended to replace the local
lists that many buses keep. They are lists of struct devices and
struct device_drivers, respectively. Bus drivers are free to use the
lists as they please, but conversion to the bus-specific type may be
necessary.
这个设备、驱动列表是为了替换很多总线持有的表。它罗列出 devices 结构和
devices_drivers 结构。总线驱动可以为所欲为的自由使用这些列表，但是可能需要转换成
特定的总线类型。

The LDM core provides helper functions for iterating over each list.
LDM核心提供辅助函数用来遍历每一个表。

int bus_for_each_dev(struct bus_type * bus, struct device * start, void * data,
		     int (*fn)(struct device *, void *));

int bus_for_each_drv(struct bus_type * bus, struct device_driver * start,
		     void * data, int (*fn)(struct device_driver *, void *));

These helpers iterate over the respective list, and call the callback
for each device or driver in the list. All list accesses are
synchronized by taking the bus's lock (read currently). The reference
count on each object in the list is incremented before the callback is
called; it is decremented after the next object has been obtained. The
lock is not held when calling the callback.
这些辅助函数遍历各自的列表，并且为列表里的每个设备或者驱动调用回调函数。所有的
列表访问都通过获得总线锁来同步（当前为读）。列表中每个对象的引用计数在回调函数
调用前递增；在取的下一个对象的时候递减。在调用回调过程中锁不被持有。


sysfs
~~~~~~~~
There is a top-level directory named 'bus'.
在顶层目录有一个名为 bus 的目录。

Each bus gets a directory in the bus directory, along with two default
directories:
在 bus 目录下每条总线有一个带有两个默认目录的目录：

	/sys/bus/pci/
	|-- devices
	`-- drivers

Drivers registered with the bus get a directory in the bus's drivers
directory:
注册在总线上的驱动在总线的 drivers 目录下拥有一个目录。

	/sys/bus/pci/
	|-- devices
	`-- drivers
	    |-- Intel ICH
	    |-- Intel ICH Joystick
	    |-- agpgart
	    `-- e100

Each device that is discovered on a bus of that type gets a symlink in
the bus's devices directory to the device's directory in the physical
hierarchy:
每个在该总线上发现的设备在该总线目录下拥有一个链接到在硬件层里的设备目录
的链接文件：

	/sys/bus/pci/
	|-- devices
	|   |-- 00:00.0 -> ../../../root/pci0/00:00.0
	|   |-- 00:01.0 -> ../../../root/pci0/00:01.0
	|   `-- 00:02.0 -> ../../../root/pci0/00:02.0
	`-- drivers


Exporting Attributes
导出属性
~~~~~~~~~~~~~~~~~~~~
struct bus_attribute {
	struct attribute	attr;
	ssize_t (*show)(struct bus_type *, char * buf);
	ssize_t (*store)(struct bus_type *, const char * buf, size_t count);
};

Bus drivers can export attributes using the BUS_ATTR macro that works
similarly to the DEVICE_ATTR macro for devices. For example, a definition
like this:
总线驱动可以用 BUS_ATTR 宏来导出属性，就像 DEVICE_ATTR 宏用在设备上一样。例如，
可以这样定义：

static BUS_ATTR(debug,0644,show_debug,store_debug);

is equivalent to declaring:
等价于以下声明：

static bus_attribute bus_attr_debug;

This can then be used to add and remove the attribute from the bus's
sysfs directory using:
通常从sysfs目录增加和删除属性是用以下函数：

int bus_create_file(struct bus_type *, struct bus_attribute *);
void bus_remove_file(struct bus_type *, struct bus_attribute *);

```
