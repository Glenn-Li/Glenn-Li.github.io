```txt
The Linux Kernel Device Model
Linux 内核设备模型

Patrick Mochel	<mochel@digitalimplant.org>
Patrick Mochel	<mochel@digitalimplant.org>

Drafted 26 August 2002
Updated 31 January 2006
2002年8月26日起草
2006年1月31日更新


Overview
概述
~~~~~~~~

The Linux Kernel Driver Model is a unification of all the disparate driver
models that were previously used in the kernel. It is intended to augment the
bus-specific drivers for bridges and devices by consolidating a set of data
and operations into globally accessible data structures.
Linux 内核设备模式是以前在内核中用过的所有不同的驱动模型的统一。它的目的是在全
局可访问的数据结构中强化一组数据和操作从而增强桥接器和设备的总线特定的驱动。

Traditional driver models implemented some sort of tree-like structure
(sometimes just a list) for the devices they control. There wasn't any
uniformity across the different bus types.
传统驱动模型为他们控制的设备实现一些树状结构(有时只是一个列表)。总线类型之间没有
任何统一性。

The current driver model provides a common, uniform data model for describing
a bus and the devices that can appear under the bus. The unified bus
model includes a set of common attributes which all busses carry, and a set
of common callbacks, such as device discovery during bus probing, bus
shutdown, bus power management, etc.
现在的驱动模型提供通用的、统一的数据模型用来描述一条总线和挂载在它下面的设备。
通用总线模型包括一组所有总线都有的通用属性、回调，例如总线探测、关闭、电源管理
期间的设备发现。

The common device and bridge interface reflects the goals of the modern
computer: namely the ability to do seamless device "plug and play", power
management, and hot plug. In particular, the model dictated by Intel and
Microsoft (namely ACPI) ensures that almost every device on almost any bus
on an x86-compatible system can work within this paradigm.  Of course,
not every bus is able to support all such operations, although most
buses support most of those operations.
通用设备和桥接器接口反映了现代计算机的目标，即能够让设备无缝即插即用、电源管
理和热插拔。特别是由Intel和Microsoft提出的模型（即ACPI）确保了几乎所有总线上
的所有设备可以以这种规范在X86兼容系统上工作。当然，即使大多数总线支持大多数的
这些操作，但并不是每种总线都能全部支持这些操纵。

Downstream Access
低层访问
~~~~~~~~~~~~~~~~~

Common data fields have been moved out of individual bus layers into a common
data structure. These fields must still be accessed by the bus layers,
and sometimes by the device-specific drivers.
通用数据字段已经从总线私有层移到了通用数据结构。这些字段必须依旧可以被总线层
访问，有时还要能被特定设备驱动程序访问。

Other bus layers are encouraged to do what has been done for the PCI layer.
struct pci_dev now looks like this:
鼓励其他总线层像PCI层一样处理。现在结构体 pci_dev 想这样：

struct pci_dev {
	...

	struct device dev;     /* Generic device interface */
			       /* 通用设备接口 */
	...
};

Note first that the struct device dev within the struct pci_dev is
statically allocated. This means only one allocation on device discovery.
首先要注意是在结构 pci_dev 里的设备结构 dev 是静态分配的。这意味着在设备发
现时只需要一次分配。

Note also that that struct device dev is not necessarily defined at the
front of the pci_dev structure.  This is to make people think about what
they're doing when switching between the bus driver and the global driver,
and to discourage meaningless and incorrect casts between the two.
还要注意的是 dev 结构体不需要定义在 pci_dev 结构体的前面。这会让人去思考当
总线驱动和全局驱动切换时它们在做什么和阻止它们之间无意义和不正确的转换。

The PCI bus layer freely accesses the fields of struct device. It knows about
the structure of struct pci_dev, and it should know the structure of struct
device. Individual PCI device drivers that have been converted to the current
driver model generally do not and should not touch the fields of struct device,
unless there is a compelling reason to do so.
PCI 总线层可以自由访问 device 结构的字段。它知道 pci_dev 结构体，也就能知道
device 结构体。个别已经转换成当前驱动模型的PCI设备驱动通常不会也不该接触
device 结构的字段，这么做除非有令人信服的理由。


The above abstraction prevents unnecessary pain during transitional phases.
If it were not done this way, then when a field was renamed or removed, every
downstream driver would break.  On the other hand, if only the bus layer
(and not the device layer) accesses the struct device, it is only the bus
layer that needs to change.
以上抽象是为了防止在过度阶段产生不必要的麻烦。如果不这么做，那么当一个字段被
重命名或者删除时，底层所有驱动就会崩溃。另一方面，如果只让总线层（不是设备层）
访问 device 结构，则仅需要修改总线层。


User Interface
用户接口
~~~~~~~~~~~~~~

By virtue of having a complete hierarchical view of all the devices in the
system, exporting a complete hierarchical view to userspace becomes relatively
easy. This has been accomplished by implementing a special purpose virtual
file system named sysfs.
凭借一个包含所有系统总设备的完成层级视图，导出一个完成的层级视图到用户空间变得
非常简单。这是通过实现叫sysfs的专用文件系统而完成的。

Almost all mainstream Linux distros mount this filesystem automatically; you
can see some variation of the following in the output of the "mount" command:
几乎所有的主流Linux发行版会自动挂载这个文件系统：可以在 mount 命令的输出中看到
一些差异：

$ mount
...
none on /sys type sysfs (rw,noexec,nosuid,nodev)
...
$

The auto-mounting of sysfs is typically accomplished by an entry similar to
the following in the /etc/fstab file:
sysfs的自动挂载通常是通过类似在 /etc/fstab 文件中的条目完成的：

none     	/sys	sysfs    defaults	  	0 0

or something similar in the /lib/init/fstab file on Debian-based systems:
或者在 Debian 基础系统中类似 /lib/init/fstab 文件中的条目：

none            /sys    sysfs    nodev,noexec,nosuid    0 0

If sysfs is not automatically mounted, you can always do it manually with:
如果 sysfs 没有被自动挂载，依旧可以手动的执行：

# mount -t sysfs sysfs /sys

Whenever a device is inserted into the tree, a directory is created for it.
This directory may be populated at each layer of discovery - the global layer,
the bus layer, or the device layer.
任何时候，一个设备被插入到树中都会为它创建一个目录。这个目录可能会在全局层、
总线层或者设备层中找到。

The global layer currently creates two files - 'name' and 'power'. The
former only reports the name of the device. The latter reports the
current power state of the device. It will also be used to set the current
power state.
在全局层中一般会创建 name 和 power 两个文件。前者仅说明了设备名称。后者说明当时
的设备电源状态，也一般会用来设置当前的电源状态。

The bus layer may also create files for the devices it finds while probing the
bus. For example, the PCI layer currently creates 'irq' and 'resource' files
for each PCI device.
总线层也可能会为在总线探测时找到的设备创建文件。例如，PCI 层一般会给每一个PCI
设备创建 irq 和 resource 文件。

A device-specific driver may also export files in its directory to expose
device-specific data or tunable interfaces.
具体的设备驱动也可能会为了暴露具体设备数据或者可控接口而导出文件。

More information about the sysfs directory layout can be found in
the other documents in this directory and in the file
Documentation/filesystems/sysfs.txt.
关于sysfs目录布局的更多信息可以查阅这个目录下的其他文件和
Documentation/filesystems/sysfs.txt 文件。
```
