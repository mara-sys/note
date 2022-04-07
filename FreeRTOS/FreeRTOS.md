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


## 第四章 FreeRTOS 中断配置和临界段
## 第五章 FreeRTOS 任务基础知识
### 5.1 什么是多任务系统
&emsp;&emsp;在裸机中，通常用一个 while 循环来完成所有的处理，有时候也需要在中断中完成一些处理。相对于多任务系统而言，这个就是单任务系统，也称作前后台系统，终端服务函数作为前台程序，大循环 while 作为后台程序。
&emsp;&emsp;前后台系统的实时性差，各个任务都是排队等着轮流执行，相当于所有任务的优先级都是一样的。
&emsp;&emsp;在多任务系统中，高优先级的任务可以打断低优先级任务的运行而取得 CPU 的使用权，这样就保证了那些紧急任务的运行。高优先级的任务执行完成以后重新把 CPU 的使用权归还给低优先级的任务。

### 5.2 FreeRTOS 任务与协程
&emsp;&emsp;在 FreeRTOS 中应用既可以使用任务，也可以使用协程（Co-Routine），或者两者混合使用。但是任务和协程使用不同的 API 函数，因此不能通过队列（或信号量）将数据从任务发送给协程，反之亦然。
#### 5.2.1 任务（Task）的特性
&emsp;&emsp;在使用 RTOS 的时候每个任务都有自己的运行环境，不依赖于系统中其他的任务或者 RTOS 调度器。任何一个时间点只能由一个任务运行。RTOS 的职责是确保当一个任务开始执行的时候其上下文环境（寄存器值，堆栈内容等）和任务上一次退出的时候相同。为了做到这一点，每个任务都必须有个堆栈，当任务切换的时候将上下文环境保存在堆栈中，这样当任务再次执行的时候就可以从堆栈中取出上下文环境，任务恢复运行。
### 5.3 任务状态
&emsp;&emsp;FreeRTOS 中的任务永远处于下面几个状态中的某一个：
* 运行态：处于运行态的任务就是当前正在使用处理器的任务。如果使用的是单核处理器的话那么不管在任何时刻永远都只有一个任务处于运行态。
* 就绪态：处于就绪态的任务就是那些准备就绪（这些任务没有被阻塞或者挂起），可以运行的任务，但是处于就绪态的任务还没有运行，因为有一个同优先级或者更高优先级的任务正在运行。
* 阻塞态：如果一个任务当前正在等待某个外部事件的话就说它处于阻塞态，比如说如果某个任务调用了函数`vTaskDelay()`的话就会进入阻塞态，直到延时周期完成。任务在等待队列、信号量、事件组、通知或互斥信号量的时候也会进入阻塞态。任务进入阻塞态会有一个超时时间，当超过这个超时时间任务就会退出阻塞态，即使所等待的事件还没有来临。
* 挂起态：像阻塞态一样，任务进入挂起态以后也不能被调度器调用进入运行态，但是进入挂起态的任务没有超时时间。任务进入和退出挂起态通过调用函数`vTaskSuspend()`和`xTaskResume()`。

&emsp;&emsp;任务状态之间的转换如图所示：  
![任务状态转换](./FreeRTOS_images/050401_任务状态转换.png)  

### 5.4 任务优先级
&emsp;&emsp;每个任务都可以分配一个从 0~（configMAX_PRIORITIES-1）的优先级。
&emsp;&emsp;如果硬件平台支持类似计算前导零这样的指令，并且宏 configUSE_PORT_OPTIMISED_TASK_SELECTION 也设置为了 1，那么宏 configMAX_PRIORITIES 不能超过 32.
&emsp;&emsp;其他情况下 configMAX_PRIORITIES 可以为任意值。
&emsp;&emsp;优先级数字越低标识任务的优先级越低，0 的优先级最低，空闲任务的优先级最低，为 0。
&emsp;&emsp;当宏 configUSE_TIME_SLICING 为 1 的时候，多个任务可以共用同一个优先级，数量不限。此时处于就绪态的优先级相同的任务就会使用时间片轮转调度器获取运行时间。

```c
BaseType_t xTaskCreate(	TaskFunction_t pxTaskCode,
                        const char * const pcName,
                        const uint16_t usStackDepth,
                        void * const pvParameters,
                        UBaseType_t uxPriority,
                        TaskHandle_t * const pxCreatedTask ) 

TaskHandle_t xTaskCreateStatic(	TaskFunction_t pxTaskCode,
                                const char * const pcName,
                                const uint32_t ulStackDepth,
                                void * const pvParameters,
                                UBaseType_t uxPriority,
                                StackType_t * const puxStackBuffer,
                                StaticTask_t * const pxTaskBuffer )
```

### 5.5 任务实现
&emsp;&emsp;在使用Fre





















