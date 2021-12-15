# I.MX6U嵌入式Linux驱动开发指南

# 第四篇 ARM Linux 驱动开发篇
&emsp;&emsp;查看设备状态总结
1. 查看设备号
```shell
cat /proc/devices
```
## 驱动相关知识总结
### 1 Linux 设备号
#### 1.1 设备号的组成
&emsp;&emsp;Linux 中每个设备都有一个设备号，设备号由主设备号和次设备号两部分组成，**主设备号表示某一个具体的驱动，次设备号表示使用这个驱动的各个设备。**
&emsp;&emsp;Linux 提供了一个名为`dev_t`的数据类型表示设备号，其定义如下：
```c
typedef unsigned int __u32;
typedef __u32 __kernel_dev_t;
typedef __kernel_dev_t dev_t;
```
&emsp;&emsp;即`dev_t`是一个 unsigned int 类型。其中高12位为主设备号，低20位为次设备号。
&emsp;&emsp;几个操作设备号的操作函数（本质上就是宏，掩码和移位）：
```c
#define MINORBITS 20
#define MINORMASK ((1U << MINORBITS) - 1)

#define MAJOR(dev) ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev) ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi) (((ma) << MINORBITS) | (mi))
```
* MAJOR：从`dev_t`中获取主设备号。
* MINOR：从`dev_t`中获取次设备号。
* MKDEV：将给定的主设备号和次设备号组成 dev_t 类型的设备号。
#### 1.2 旧设备号的分配
&emsp;&emsp;见 40.2.2 节。
#### 1.3 新设备号的分配
1. 静态分配设备号
&emsp;&emsp;静态分配设备号需要我们检查当前系统中所有被使用了的设备号，然后挑选一个没有使用的。
2. 动态分配设备号
&emsp;&emsp;在注册字符设备之前先申请一个设备号，系统自动分配一个没有被使用的设备号，卸载驱动时释放掉这个设备号即可。
```c
/* 
 * 静态申请设备号
 * from: 要申请的起始设备号，也就是给定的设备号。
 * count: 要申请的数量
 * name: 设备名字
 */
int register_chrdev_region(dev_t from, unsigned count, const char *name)

/* 
 * 动态申请设备号
 * dev: 保存申请到的设备号
 * baseminor: 次设备号起始地址，此函数可以申请一段连续的多个次设备号，
 *            这些设备号的主设备号一样，但是次设备号不同，次设备号以
 *            baseminor为起始地址开始递增。一般为0。
 * count: 要申请的设备号的数量。
 * name: 设备名字。
 */
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, 
        unsigned count, const char *name)

/* 
 * 上面两个函数申请的设备号都通过此函数注销。
 * from: 要释放的设备号。
 * count: 表示从 from 开始，要释放的设备号的数量。
 */
void unregister_chrdev_region(dev_t from, unsigned count)
```

### 2 printk 函数消息级别
&emsp;&emsp;printk 可以根据日志级别对消息进行分类，一共有8个消息级别，0的优先级最高，7的优先级最低。定义在文件`include/linux/kern_level.h`中
```c
#define KERN_SOH "\001"
#define KERN_EMERG KERN_SOH "0"   /* 紧急事件，一般是内核崩溃 */
#define KERN_ALERT KERN_SOH "1"   /* 必须立即采取行动 */
#define KERN_CRIT KERN_SOH "2"    /* 临界条件，比如严重的软件或硬件错误*/
#define KERN_ERR KERN_SOH "3"     /* 错误状态，一般设备驱动程序中使用
                                     KERN_ERR 报告硬件错误 */
#define KERN_WARNING KERN_SOH "4" /* 警告信息，不会对系统造成严重影响 */
#define KERN_NOTICE KERN_SOH "5"  /* 有必要进行提示的一些信息 */
#define KERN_INFO KERN_SOH "6"    /* 提示性的信息 */
#define KERN_DEBUG KERN_SOH "7"   /* 调试信息 */
```
&emsp;&emsp;如果要设置消息级别，示例如下：
```c
printk(KERN_EMERG "gsmi: Log Shutdown Reason\n");
```
&emsp;&emsp;如果使用 printk 的时候不显示的设置消息级别，默认采用`MESSAGE_LOGLEVEL_DEFAULT`，默认为4。
&emsp;&emsp;在`include/linux/printk.h`中有个宏`CONSOLE_LOGLEVEL_DEFAULT`，定义如下：
```c
#define CONSOLE_LOGLEVEL_DEFAULT 7
```
&emsp;&emsp;此宏默认为 7，意味着只有优先级高于 7 的消息才能显示在控制台上。


## 第四十章 字符设备驱动开发
### 40.1 字符设备驱动简介
&emsp;&emsp;字符设备就是一个一个字节，按照字节流进行读写操作的设备，读写数据是分先后顺序的。
### 40.2 字符设备驱动开发步骤
#### 40.2.1 驱动模块的加载和卸载
&emsp;&emsp;可以使用`insmod`和`modprobe`来加载驱动模块。
* insmod：不能解决模块的依赖关系
* modprobe：会分析模块的依赖关系，然后会将所有依赖的模块都加载到内核中。modprobe 默认会去`lib/modules/<kernel-version>`目录中查找模块。一般自己制作的根文件系统中时不会有这个目录的，需要自己手动创建。  

&emsp;&emsp;模块的加载和卸载动作在驱动中体现为两个函数：
```c
module_init(xxx_init);      //注册模块加载函数
module_exit(xxx_exit);      //注册模块卸载函数
```
&emsp;&emsp;加载函数和卸载函数模板：
```c
static int __init xxx_init(void)
{
    /* 入口函数具体内容 */
    return 0;
}

static void __exit xxx_exit(void)
{
    /* 出口函数具体内容 */
}
/* 将上面两个函数指定为驱动的入口和出口函数 */
module_init(xxx_init);
module_exit(xxx_exit);
```
#### 40.2.2 字符设备注册与注销
&emsp;&emsp;驱动模块加载成功以后需要注册字符设备，卸载驱动模块的时候需要注销字符设备。
```c
static inline int register_chrdev(unsigned int major, 
    const char *name, const struct file_operations *fops)
static inline void unregister_chrdev(unsigned int major, 
    const char *name)
```
&emsp;&emsp;这两个函数的参数含义如下：
* major：主设备号。
* name：设备名字，指向一串字符串。
* fops：结构体 file_operations 类型的指针，指向设备的操作函数集合变量。


#### 40.2.4 添加 LICENSE 和作者信息
```c
MODULE_LICENSE() //添加模块 LICENSE 信息，必须添加
MODULE_AUTHOR()  //添加模块作者信息，可以不添加
```


&emsp;&emsp;总结起来，字符设备的驱动编写模板如下：
```c
static struct file_operations test_fops;

/* 下面这些是根据自己需求需要实现的回调函数 */
static struct file_operations test_fops = {
    .owner = THIS_MODULE,
    .open = chrtest_open,
    .read = chrtest_read,
    .write = chrtest_write,
    .release = chrtest_release,
};

/* 驱动入口函数 */
static int __init xxx_init(void)
{
    /* 入口函数具体内容 */
    int retvalue = 0;

    /* 注册字符设备驱动 */
    retvalue = register_chrdev(200, "chrtest", &test_fops);
    if(retvalue < 0){
        /* 字符设备注册失败,自行处理 */
    }
    return 0;
}

/* 驱动出口函数 */
static void __exit xxx_exit(void)
{
    /* 注销字符设备驱动 */
    unregister_chrdev(200, "chrtest");
}

/* 将上面两个函数指定为驱动的入口和出口函数 */
module_init(xxx_init);
module_exit(xxx_exit);
```
&emsp;&emsp;驱动加载成功以后需要在 /dev 下面创建一个与之对应的设备节点文件，应用程序通过此设备节点来操作具体的设备。
```shell
mknod /dev/chrdevbase c 200 0
```
&emsp;&emsp;然后操作此文件即可。
&emsp;&emsp;即，通过`module_init`，`module_exit`指定驱动出口入口函数，通过`register_chrdev`，`unregister_chrdev`注册注销字符设备，使用`insmode`，`rmmode`加载卸载驱动，通过`mknod`创建设备节点。

### 40.4 
#### 40.4.3 编译驱动程序和测试APP
1. 编译驱动程序（编译成 .ko 文件，insmod）
```Makefile
KERNELDIR := /home/zuozhongkai/linux/IMX6ULL/linux/temp/linux-imxrel_imx_4.1.15_2.1.0_ga_alientek
CURRENT_PATH := $(shell pwd)
obj-m := chrdevbase.o

build: kernel_modules

kernel_modules:
    $(MAKE) -C $(KERNELDIR) M=$(CURRENT_PATH) modules
clean:
    $(MAKE) -C $(KERNELDIR) M=$(CURRENT_PATH) clean
```
2. 编译测试APP
```Makefile
arm-linux-gnueabihf-gcc chrdevbaseApp.c -o chrdevbaseApp
```

## 第四十一章 （物理地址和虚拟地址映射函数）
&emsp;&emsp;物理地址和虚拟地址映射函数：`ioremap`和`iounmap`。
```c
/* 
 * cookie: 要映射的物理起始地址 
 * size: 要映射的内存空间大小
 * 返回值：__iomem 类型的指针，指向映射后的虚拟空间首地址
 */
ioremap(cookie,size)

/* 
 * addr: 要取消映射的虚拟地址空间首地址
 */
void iounmap (volatile void __iomem *addr)

//示例
#define SW_MUX_GPIO1_IO03_BASE (0X020E0068)
static void __iomem* SW_MUX_GPIO1_IO03;
SW_MUX_GPIO1_IO03 = ioremap(SW_MUX_GPIO1_IO03_BASE, 4);

iounmap(SW_MUX_GPIO1_IO03);
```

&emsp;&emsp;I/O内存访问函数，读操作函数和写操作函数，分别是 8 位，16 位，32 位：
```c
u8 readb(const volatile void __iomem *addr)
u16 readw(const volatile void __iomem *addr)
u32 readl(const volatile void __iomem *addr)
void writeb(u8 value, volatile void __iomem *addr)
void writew(u16 value, volatile void __iomem *addr)
void writel(u32 value, volatile void __iomem *addr)
```

## 第四十二章 新字符设备驱动实验
&emsp;&emsp;第四十章使用的函数`register_chrdev`会将一个主设备号下的所有次设备号都使用掉，新的字符设备驱动已经不再使用这两个函数。
### 42.1 新字符设备驱动原理
#### 42.1.1 分配和释放设备号
&emsp;&emsp;参见驱动相关知识总结，设备号一节。
#### 42.1.2 新的字符设备注册方法
&emsp;&emsp;使用一个结构体`cdev`来抽象表示一个字符设备，其定义如下：
```c
struct cdev {
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops;
    struct list_head list;
    dev_t dev;
    unsigned int count;
};
```
&emsp;&emsp;成员变量`ops`时字符设备文件操作集合，`dev_t`是设备号，`owner`通常是`THIS_MODULE`。需要使用此结构体定义的变量来注册其代表的字符设备。注册注销函数流程如下：
```c
struct cdev testcdev;

/* 设备操作函数 */
static struct file_operations test_fops = {
    .owner = THIS_MODULE,
    /* 其他具体的初始项 */
};

testcdev.owner = THIS_MODULE;
cdev_init(&testcdev, &test_fops); /* 初始化 cdev 结构体变量 */
cdev_add(&testcdev, devid, 1); /* 添加字符设备 */

cdev_del(&testcdev); /* 删除 cdev */
```
&emsp;&emsp;各函数定义如下：
```c
/* 
 * cdev: 要注册的字符设备
 * fops: 对应的操作函数集合
 */
void cdev_init(struct cdev *cdev, const struct file_operations *fops)

/* 
 * cdev: 要添加的字符设备
 * dev: 要添加的设备的起始设备号
 * count: 此设备对应的连续的次设备号的数量
 */
int cdev_add(struct cdev *p, dev_t dev, unsigned count)

/* 
 * p: 要删除的字符设备
 */
void cdev_del(struct cdev *p)
```
### 42.2 自动创建设备节点
&emsp;&emsp;在驱动中实现自动创建设备节点的功能以后，使用 modprobe 加载驱动模块成功的话就会自动在 /dev 目录下创建对应的设备文件。
&emsp;&emsp;自动创建设备节点，需要在驱动入口函数中先创建一个类，再在这个类下创建一个设备。卸载驱动的时候需要删除掉创建的设备，先删除掉设备，再删除对应的类。流程如下：
```c
struct class *class; /* 类 */
struct device *device; /* 设备 */
dev_t devid; /* 设备号 */

/* 驱动入口函数 */
static int __init xxx_init(void)
{
    /* 创建类 */
    class = class_create(THIS_MODULE, "xxx");
    /* 创建设备 */
    device = device_create(class, NULL, devid, NULL, "xxx");
    return 0;
}

/* 驱动出口函数 */
static void __exit led_exit(void)
{
    /* 删除设备 */
    device_destroy(newchrled.class, newchrled.devid);
    /* 删除类 */
    class_destroy(newchrled.class);
}

module_init(led_init);
module_exit(led_exit);
```
&emsp;&emsp;各函数原型如下：
```c
/* 
 * class_create是一个宏，展开后如下所示
 * owner: 一般为 THIS_MODULE
 * name: 类名字
 * 返回值：指向结构体 class 的指针，也就是创建的类
 */
struct class *class_create (struct module *owner, const char *name)

/* 
 * cls: 要删除的类
 */
void class_destroy(struct class *cls);

/* 
 * class: 要创建在哪个类下面
 * parent: 父设备，如果没有，设为 NULL
 * devt: 设备号
 * drvdata: 设备可能会使用的私有数据，如果没有，设为 NULL
 * fmt: 设备名字
 * 返回值：创建好的设备
 */
struct device *device_create(struct class *class,
                             struct device *parent,
                             dev_t devt,
                             void *drvdata,
                             const char *fmt, ...)

/* 
 * class: 要删除的设备所处的类
 * devt: 要删除的设备号
 */
void device_destroy(struct class *class, dev_t devt)
```
### 42.4 总结
&emsp;&emsp;总的来说，新的字符设备注册以及自动创建设备节点示例如下：
```c
/* newchrled 设备结构体 */
struct newchrled_dev{
    dev_t devid; /* 设备号 */
    struct cdev cdev; /* cdev */
    struct class *class; /* 类 */
    struct device *device; /* 设备 */
};

struct newchrled_dev newchrled; /* led 设备 */

/* 设备操作函数 */
static struct file_operations newchrled_fops = {
    .owner = THIS_MODULE,
    .open = led_open,
    .read = led_read,
    .write = led_write,
    .release = led_release,
};

static int __init led_init(void)
{
    /* 对硬件设备的初始化配置 */

    /* 注册字符设备驱动，下面两个函数根据需要选择 */
    /* 1、设备号 */
    register_chrdev_region(newchrled.devid, NEWCHRLED_CNT,
        NEWCHRLED_NAME);
    alloc_chrdev_region(&newchrled.devid, 0, NEWCHRLED_CNT,
        NEWCHRLED_NAME); 

    /* 2、初始化抽象设备的结构体 cdev */
    newchrled.cdev.owner = THIS_MODULE;
    cdev_init(&newchrled.cdev, &newchrled_fops);

    /* 3、添加一个 cdev */
    cdev_add(&newchrled.cdev, newchrled.devid, NEWCHRLED_CNT);

    /* 上面已经完成了字符设备的注册，下面是自动创建设备节点 */
    /* 1、创建类 */
    newchrled.class = class_create(THIS_MODULE, NEWCHRLED_NAME);

    /* 2、创建设备 */
    newchrled.device = device_create(newchrled.class, NULL,
        newchrled.devid, NULL, NEWCHRLED_NAME);
}

static void __exit led_exit(void)
{
    /* 对硬件设备的操作 */

    /* 注销字符设备 */
    cdev_del(&newchrled.cdev);/* 删除 cdev */
    unregister_chrdev_region(newchrled.devid, NEWCHRLED_CNT);

    device_destroy(newchrled.class, newchrled.devid);
    class_destroy(newchrled.class);
}

module_init(led_init);
module_exit(led_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("zuozhongkai");
```

## 第四十三章 设备树
### 43.2 DTS、DTB 和 DTC
&emsp;&emsp;要编译 DTS 文件的话只需要进入到 Linux 源码根目录下，然后执行 `make dtbs` 即可。
&emsp;&emsp;在`arch/arm/boot/dts/Makefile`中（对于arm来说，是在arm架构下，别的架构去对应的路径下找即可，例如`arch/riscv/boot/dts/Makefile`）有如下内容：
```c
dtb-$(CONFIG_SOC_IMX6UL) += \
    imx6ul-14x14-ddr3-arm2.dtb \
......
dtb-$(CONFIG_SOC_IMX6ULL) += \
......
    imx6ull-14x14-evk-usb-certi.dtb \
    imx6ull-alientek-emmc.dtb \
    imx6ull-alientek-nand.dtb \
......
dtb-$(CONFIG_SOC_IMX6SLL) += \
    imx6sll-lpddr2-arm2.dtb \
```
&emsp;&emsp;当选中 I.MX6ULL 这个SOC以后（CONFIG_SOC_IMX6ULL=y），所有使用到 I.MX6ULL 这个 SOC 的板子对应的 .dts 文件都会被编译为 .dtb 如果我们使用 I.MX6ULL 新做了一个板子，只需要新建一个此板子对应的 .dts 文件，然后将对应的 .dtb 文件名添加到 dtb-$(CONFIG_SOC_IMX6ULL)下，这样在编译设备树的时候就会将对应的 .dts 编译为二进制的 .dtb 文件。

### 43.5 设备树在系统中的体现
&emsp;&emsp;Linux 内核启动的时候会解析设备树中各个节点的信息，并且在根文件系统的`/proc/device-tree`目录下根据节点名字创建不同的文件夹。其目录结构和设备树是一致的。
### 43.9 设备树常用 OF 操作函数
&emsp;&emsp;Linux 内核提供了一系列函数来获取设备树中的节点和节点中的属性信息。这一系列函数都有一个统一的前缀“of_”，这些函数原型定义在`include/linux/of.h`文件中。
#### 43.9.1 查找节点的 OF 函数
&emsp;&emsp;设备都是以节点的形式“挂”到设备树上的，因此要想获取这个设备的其他属性信息，必须先获取到这个设备节点。Linux 内核使用`device_node`来抽象一个节点，定义如下：
```c
struct device_node {
    const char *name; /* 节点名字 */
    const char *type; /* 设备类型 */
    phandle phandle;
    const char *full_name; /* 节点全名 */
    struct fwnode_handle fwnode;

    struct property *properties; /* 属性 */
    struct property *deadprops; /* removed 属性 */
    struct device_node *parent; /* 父节点 */
    struct device_node *child; /* 子节点 */
    struct device_node *sibling;
    struct kobject kobj;
    unsigned long _flags;
    void *data;
#if defined(CONFIG_SPARC)
    const char *path_component_name;
    unsigned int unique_id;
    struct of_irq_controller *irq_trans;
#endif
};
```
&emsp;&emsp;查找节点的 OF 函数共有 5 个：
```c
/* 
 * from: 开始查找的节点，如果为 NULL 表示从根节点开始查找整个设备树
 * name: 要查找的节点名字
 * 返回值：找到的节点，如果为 NULL 表示查找失败
 */
struct device_node *of_find_node_by_name(struct device_node *from,
                const char *name);

/* 
 * from: 开始查找的节点，如果为 NULL 表示从根节点开始查找整个设备树
 * type: 要查找的节点对应的 type 字符串，也就是 device_type 属性值
 */
struct device_node *of_find_node_by_type(struct device_node *from,
                const char *type)

/* 
 * type: 可以为 NULL，表示忽略掉 device_type 属性
 * compatible：要查找的节点所对应的 compatible 属性列表
 */
struct device_node *of_find_compatible_node(struct device_node *from,
                const char *type,
                const char *compatible)

/* 
 * matches：of_device_id 匹配表，也就是在此匹配表里面查找节点。
 * match：找到的匹配的 of_device_id。
 */
struct device_node *of_find_matching_node_and_match(struct device_node *from,
                const struct of_device_id *matches,
                const struct of_device_id **match)

/* 
 * path：带有全路径的节点名，可以使用节点的别名，
 * 比如“/backlight”就是 backlight 这个节点的全路径。
 */
inline struct device_node *of_find_node_by_path(const char *path)
```
&emsp;&emsp;查找父节点或子节点的 OF 函数
```c
/* 
 * node：要查找的父节点的节点
 * 返回值：找到的父节点
 */
struct device_node *of_get_parent(const struct device_node *node)

/* 
 * node：父节点
 * prev：前一个子节点，也就是从哪一个子节点开始迭代的查找下一个子节点。
 *       可以设置为 NULL，表示从第一个子节点开始。
 * 返回值：找到的父节点
 */
struct device_node *of_get_next_child(const struct device_node *node,
                            struct device_node *prev)
```
#### 43.9.3 查找属性值的 OF 函数
&emsp;&emsp;Linux 内核使用结构体 property 抽象属性，定义如下：
```c
struct property {
    char *name; /* 属性名字 */
    int length; /* 属性长度 */
    void *value; /* 属性值 */
    struct property *next; /* 下一个属性 */
    unsigned long _flags;
    unsigned int unique_id;
    struct bin_attribute attr;
};
```
&emsp;&emsp;让我们来看看从驱动程序代码启用这个时钟： 
```c
	clk_enable(clk);
		clk->ops->enable(clk->hw);
		[resolves to...]
			clk_gate_enable(hw);
			[resolves struct clk gate with to_clk_gate(hw)]
				clk_gate_set_bit(gate);
```



































## 第五十一章 Linux中断实验


## 第五十八章 Linux INPUT子系统实验
