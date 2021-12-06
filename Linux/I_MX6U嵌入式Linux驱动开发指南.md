# I.MX6U嵌入式Linux驱动开发指南

# 第四篇 ARM Linux 驱动开发篇
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



## 第五十一章 Linux中断实验


## 第五十八章 Linux INPUT子系统实验
