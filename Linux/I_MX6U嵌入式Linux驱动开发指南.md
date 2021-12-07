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
#### 1.2 设备号的分配
1. 静态分配设备号
&emsp;&emsp;静态分配设备号需要我们检查当前系统中所有被使用了的设备号，然后挑选一个没有使用的。
2. 动态分配设备号
&emsp;&emsp;在注册字符设备之前先申请一个设备号，系统自动分配一个没有被使用的设备号，卸载驱动时释放掉这个设备号即可。
```c
/* 
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
#### 40.4.4 运行测试
&emsp;&emsp;驱动加载成功以后需要在 /dev 下面创建一个与之对应的设备节点文件，应用程序通过此设备节点来操作具体的设备。
```shell
mknod /dev/chrdevbase c 200 0
```
&emsp;&emsp;然后操作此文件即可。

## 第四十一章 嵌入式 Linux LED 驱动开发实验



## 第五十一章 Linux中断实验


## 第五十八章 Linux INPUT子系统实验
