# FreeRTOS STM32F4
## 第一章 FreeRTOS简介
#### 1.3.2 FreeRTOS文件预览
&emsp;&emsp;FreeRTOS文件夹下有`FreeRTOS`和`FreeRTOS-Plus`两个文件夹，区别是Plus的功能比普通的功能强大一点。`FreeRTOS`文件夹下有三个文件夹`Demo`、`License`、`Source`，`Demo`中是针对不同芯片的例程，移植时可以参考，`Source`就是FreeRTOS的源码。  
&emsp;&emsp;`Source`文件夹下面包括两个文件夹`include`、`portable`和一些文件。**`include`文件夹是一些头文件，移植的时候是需要的**。`portable`文件夹里面是FreeRTOS系统和具体硬件之间的桥梁。不同的编译环境，不同的MCU，其桥梁应该是不同的，`portable`下面首先是不同的编译环境，例如GCC、Keil、IAR等。***其中`MemMang`是内存管理相关内容，移植的时候是必须的**。如下图所示：
![portable文件夹内容](./FreeRTOS_images/010302_portable文件夹内容.png)  
在每个编译环境目录下是针对不同架构的MCU的分类，如下图所示，在GCC下面，`ARM_CA9`是cortex-A9架构，`ARM_CM0`是cortex-M0架构相关代码，****其中的`prot.c`和`portmacro.h`是移植时必须的**。
## 第二章 FreeRTOS移植
### 2.2 FreeRTOS移植
&emsp;&emsp;在正点原子的历程中添加文件。添加的文件就是上面加粗的文件。包括，
* `Source`文件夹下的所有`.c`文件；
* `include`中的所有头文件；
* `portable`中的`MemMang`文件夹中任一个`.c`文件；
* `portable`中和编译器相关的具体架构的文件夹，例如：RVDS-->ARM_CM4F_MPU文件夹中的文件：`port.c`和`portmacro.h`
* 除此之外，还需要有`FreeRTOSConfig.h`，在官方demo文件中可以找到这个文件。  
&emsp;&emsp;移植完上面的文件以后，编译还会有许多错误。
* `SystemCoreClock`未定义的错误：在FreeRTOSConfig.h中使用它来标记MCU频率。
* 会有一些未定义的Hook函数，这是因为在FreeRTOSConfig.h中开启了这些钩子函数却没有定义导致的，关闭这些函数即可，将这些宏定义改为0即可。
* 在正点原子的例程中，FreeRTOS的心跳是由滴答定时器产生的，在滴答定时器中调用FreeRTOS的API函数`xPortSysTickHandler()`
## 第三章 FreeRTOS系统配置
&emsp;&emsp;实际使用FreeRTOS的时候需要根据自己的需求来配置FreeRTOS，不同架构的MCU在使用的时候配置也不同。FreeRTOS的系统配置文件为FreeRTOSConfig.h，在此配置文件中可以完成FreeRTOS的裁剪和配置。


























