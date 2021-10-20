
- [一、reboot](#一reboot)
  - [1.1 系统调用](#11-系统调用)
  - [1.2 复位函数的实现](#12-复位函数的实现)
    - [1.2.1 实例](#121-实例)
    - [1.2.2 注册与注销函数](#122-注册与注销函数)
    - [1.3 通知链表](#13-通知链表)
- [八、 零散的宏定义](#八-零散的宏定义)
  - [8.1 DEVICE_ATTR](#81-device_attr)
    - [8.1.1 介绍](#811-介绍)
    - [8.1.2 DEVICE_ATTR举例](#812-device_attr举例)
    - [8.1.3 DEVICE_ATTR分析](#813-device_attr分析)
    - [8.1.4 DEVICE_ATTR_RW、DEVICE_ATTR_RO、DEVICE_ATTR_WO分析](#814-device_attr_rwdevice_attr_rodevice_attr_wo分析)
    - [8.1.5 权限标识方法](#815-权限标识方法)
    - [8.1.6 pwm.c中宏定义的展开](#816-pwmc中宏定义的展开)
    - [8.1.7 device_attribute结构体的定义](#817-device_attribute结构体的定义)
    - [8.1.8 将属性公开到文件系统中](#818-将属性公开到文件系统中)

# 一、reboot
[参考的帖子链接](https://blog.csdn.net/renlonggg/article/details/78204305)
## 1.1 系统调用
&emsp;&emsp;命令行输入reboot到busybox中的那部分内容没理解。直接从busybox结束开始调用标准C函数reboot开始。
&emsp;&emsp;reboot函数直接进行系统调用进入内核。
&emsp;&emsp;内核系统调用，位于/linux/kernel/reboot.c
```c
SYSCALL_DEFINE4(reboot, int, magic1, int, magic2, unsigned int, cmd,
        void __user *, arg)
{
    ......

    mutex_lock(&reboot_mutex);
    switch (cmd) {
    case LINUX_REBOOT_CMD_RESTART:
        kernel_restart(NULL);
        break;

    case LINUX_REBOOT_CMD_CAD_ON:
        C_A_D = 1;
        break;

    case LINUX_REBOOT_CMD_CAD_OFF:
        C_A_D = 0;
        break;

    case LINUX_REBOOT_CMD_HALT:
        kernel_halt();
        do_exit(0);
        panic("cannot halt");

    case LINUX_REBOOT_CMD_POWER_OFF:
        kernel_power_off();
        do_exit(0);
        break;

    ......

    default:
        ret = -EINVAL;
        break;
    }
    mutex_unlock(&reboot_mutex);
    return ret;
}
```
进入kernel_restart(NULL)函数
```c
void kernel_restart(char *cmd)
{   
    kernel_restart_prepare(cmd);
    migrate_to_reboot_cpu();
    syscore_shutdown();
    if (!cmd)
        pr_emerg("Restarting system\n");
    else
        pr_emerg("Restarting system with command '%s'\n", cmd);
    kmsg_dump(KMSG_DUMP_RESTART);
    machine_restart(cmd);
}
EXPORT_SYMBOL_GPL(kernel_restart);
```
&emsp;&emsp;进入machine_restart(cmd)函数。
&emsp;&emsp;machine_restart函数在arch目录下，不同的架构会定义不同的machine_restart函数，和链接中的不太一样，我调试的machine_restart函数位于/linux/arch/riscv/kernel/reset.c
```c
void machine_restart(char *cmd)
 {   
     do_kernel_restart(cmd);
     while (1);
 }   
 
 void machine_halt(void)
 {
     machine_power_off();
 }   
 
 void machine_power_off(void)
 {
     sbi_shutdown();
     while (1);
 }   
```
&emsp;&emsp;调用了do_kernel_restart函数，此函数位于/linux/kernel/reboot.c中，
```c
void do_kernel_restart(char *cmd)
{
    atomic_notifier_call_chain(&restart_handler_list, reboot_mode, cmd);
}
```
&emsp;&emsp;atomic_notifier_call_chain此函数会调用restart_handler_list此链表中注册了的复位函数，如果注册了多个复位函数，则根据设置的结构体的优先级判定执行顺序，priority值越大，优先级越高。
&emsp;&emsp;此链表的初始化位于reboot.c中
```c
static ATOMIC_NOTIFIER_HEAD(restart_handler_list);
```
&emsp;&emsp;就是说，只要实现了复位函数，并向此链表注册复位函数，在命令行输入reboot后，就会调用到复位函数。
## 1.2 复位函数的实现
### 1.2.1 实例
&emsp;&emsp;复位函数可以依赖于某个具有复位功能的模块，例如可以通过看门狗复位，那就可以在看门狗中模块中实现复位功能并注册对应的函数。其实只要实现复位函数并注册进restart_handler_list链表即可。例如：
```c
#define SYSCTL_BOOT_BASE_ADDR   0x97000000U
#define SOC_GLB_RST             0x60

static void __iomem *MMAP_ADDR;

//复位功能实现函数
static int k510_restart(struct notifier_block *this, unsigned long mode, void *cmd)
{
    MMAP_ADDR = ioremap(SYSCTL_BOOT_BASE_ADDR + SOC_GLB_RST, 4);
    
    writel(((1 << 0) | (1 << 16)), MMAP_ADDR);
    while(1);
}   

static int k510_restart_register(void)
{
    static struct notifier_block restart_handler;

    restart_handler.notifier_call = k510_restart;
    restart_handler.priority = 128; 
    
    //向链表注册
    return register_restart_handler(&restart_handler);
}   
#endif
```
&emsp;&emsp;然后，调用此函数即可，我把这段代码写在了linux/arch/riscv/kernel/setup.c中，在setup_arch函数最后调用了此函数。
### 1.2.2 注册与注销函数
```c
int register_restart_handler(struct notifier_block *nb)
{
    return atomic_notifier_chain_register(&restart_handler_list, nb);
}
EXPORT_SYMBOL(register_restart_handler);


int unregister_restart_handler(struct notifier_block *nb)
{
    return atomic_notifier_chain_unregister(&restart_handler_list, nb);
}
EXPORT_SYMBOL(unregister_restart_handler);
```
### 1.3 通知链表
[参考链接](http://bbs.chinaunix.net/thread-2011776-1-1.html)


# 八、 零散的宏定义
## 8.1 DEVICE_ATTR
### 8.1.1 介绍
&emsp;&emsp;使用DEVICE_ATTR，可以实现驱动在sys目录自动创建文件，我们只需要实现show和store函数即可。然后在应用层就能通过cat和echo命令来对sys创建出来的文件进行读写驱动设备，实现交互。
### 8.1.2 DEVICE_ATTR举例
以pwm中的sysfs.c为例。文件位置位于/drivers/pwm/sysfs.c
```c
static ssize_t polarity_show(struct device *child,
                 struct device_attribute *attr,
                 char *buf)
{
    const struct pwm_device *pwm = child_to_pwm_device(child);
    /* 省略具体的处理内容 */

    return sprintf(buf, "%s\n", polarity);
}

static ssize_t polarity_store(struct device *child,
                  struct device_attribute *attr,
                  const char *buf, size_t size)
{
    struct pwm_export *export = child_to_pwm_export(child);
    /* 省略具体的处理内容 */

    return ret ? : size;
}

static DEVICE_ATTR_RW(polarity);
```
在`/sys/class/pwm`下可以看到polarity属性

![polarity属性](Linux_module_images/080102_polarity_example.png)

通过
```shell
echo 0 > polarity
cat polarity
```
来对该属性进行读写操作。
### 8.1.3 DEVICE_ATTR分析
DEVICE_ATTR定义在/include/linux/device.h中
```c
#define DEVICE_ATTR(_name, _mode, _show, _store) \
    struct device_attribute dev_attr_##_name = __ATTR(_name, _mode, _show, _store)
```
__ATTR定义在/include/linux/sysfs.h中
```c
#define __ATTR(_name, _mode, _show, _store) {               \
    .attr = {.name = __stringify(_name),                \
         .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },     \
    .show   = _show,                        \
    .store  = _store,                       \
}
```
### 8.1.4 DEVICE_ATTR_RW、DEVICE_ATTR_RO、DEVICE_ATTR_WO分析
```c
#define DEVICE_ATTR_RW(_name) \
    struct device_attribute dev_attr_##_name = __ATTR_RW(_name)
#define DEVICE_ATTR_RO(_name) \
    struct device_attribute dev_attr_##_name = __ATTR_RO(_name)
#define DEVICE_ATTR_WO(_name) \
    struct device_attribute dev_attr_##_name = __ATTR_WO(_name)
```
__ATTR_RW、__ATTR_RO、__ATTR_WO和__ATTR类似，定义在/include/linux/sysfs.h中
```c
#define __ATTR_RO(_name) {                      \
    .attr   = { .name = __stringify(_name), .mode = 0444 },     \
    .show   = _name##_show,                     \
}

#define __ATTR_WO(_name) {                      \
    .attr   = { .name = __stringify(_name), .mode = 0200 },     \
    .store  = _name##_store,                    \
}

#define __ATTR_RW(_name) __ATTR(_name, 0644, _name##_show, _name##_store)
```
&emsp;&emsp;__ATTR_RO只定义了show函数，函数名字是“_name_show”，类似的__ATTR_WO只定义了store函数，函数名字是”_name_store”，__ATTR_RW同时定义了show、store函数。这三个宏定义是DEVICE_ATTR的简化使用，默认对权限进行了配置。
### 8.1.5 权限标识方法
umask变量表示方法
|加权数值|第1位            |第2位          |第3位              |
|:---:  |:---             |:---           |:---              |
|100    |所有者拥有读权限  |群组拥有读权限  |其他用户拥有读权限  |
|010    |所有者拥有写权限  |群组拥有写权限  |其他用户拥有写权限  |
|001    |所有者拥有执行权限|群组拥有执行权限|其他用户拥有执行权限 |

与上面权限对应的宏定义是

|参数	|说明	|参数	|说明	|参数	|说明|
|---    |---   |---    |---    |------|----|
|S_IRUSR	|所有者拥有读权限	|S_IRGRP	|群组拥有读权限	|S_IROTH	|其他用户拥有读权限|
|S_IWUSR	|所有者拥有写权限	|S_IWGRP	|群组拥有写权限	|S_IWOTH	|其他用户拥有写权限|
|S_IXUSR	|所有者拥有执行权限	|S_IXGRP	|群组拥有执行权限	|S_IXOTH	|其他用户拥有执行权限|
### 8.1.6 pwm.c中宏定义的展开
因此
```c
static DEVICE_ATTR_RW(polarity);
```
展开以后就是
```c
struct device_attribute dev_attr_polarity = {                
    .attr = {.name = __stringify(polarity),                
         .mode = VERIFY_OCTAL_PERMISSIONS(0644) },     
    .show   = polarity_show,                        
    .store  = polarity_store,                       
}
```
其中polarity_show和polarity_store已经在上面实现了。
### 8.1.7 device_attribute结构体的定义
attribute结构体定义在include/linux/sysfs.h
```c
struct attribute {
    const char      *name;
    umode_t         mode;
#ifdef CONFIG_DEBUG_LOCK_ALLOC
    bool            ignore_lockdep:1;
    struct lock_class_key   *key;
    struct lock_class_key   skey;
#endif
};
```
&emsp;&emsp;此结构体中name代表属性名称，一般表示为文件名，mode代表该属性的读写权限。device_attribute定义在/include/linux/device.h中
```c
struct device_attribute {
    struct attribute    attr;
    ssize_t (*show)(struct device *dev, struct device_attribute *attr, char *buf);
    ssize_t (*store)(struct device *dev, struct device_attribute *attr, const char *buf, size_t count);
};
```
&emsp;&emsp;该结构体是对attribute结构体的进一步封装，并提供了两个函数指针，show函数用于读取设备的属性文件，而store则是用于写设备的属性文件，**当我们在linux驱动程序中实现了这两个函数后，便可以使用cat和echo命令对设备属性文件进行读写操作。**
### 8.1.8 将属性公开到文件系统中
&emsp;&emsp;可以通过ATTRIBUTE_GROUPS宏定义来实现，在pwm的sysfs.c中可以看到，其中的dev_attr_name就是在DEVICE_ATTR中定义的结构体。
```c
static struct attribute *pwm_attrs[] = {
    &dev_attr_period.attr,
    &dev_attr_duty_cycle.attr,
    &dev_attr_enable.attr,
    &dev_attr_polarity.attr,
    &dev_attr_capture.attr,
    NULL
};
ATTRIBUTE_GROUPS(pwm);
```