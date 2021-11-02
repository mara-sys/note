# Unix高级环境编程
## 第11章 终端I/O
### 11.2 综述
&emsp;&emsp;终端I/O有两种不同的工作方式：  
1. 规范方式输入处理。在这种方式中，终端输入以行为单位进行处理。对于每个读要求，终端驱动程序最多返回一行。  
2. 非规范方式输入处理。输入字符不以行为单位进行装配。系统不会对特殊字符进行处理。
如果不做特殊处理，则默认方式是规范方式。  

&emsp;&emsp;POSIX.1定义了11个特殊字符，其中9个可以改变。  
&emsp;&emsp;大多数UNIX系统在一个称为终端行规程的模块中进行规范处理。它是位于内核类属读、写函数和实际设备驱动程序之间的模块。  
<div align=center>
<img src="Unix高级环境编程_images/1102_终端行规程.svg" width="300">
</div>
&emsp;&emsp;所有我们可以检测和更改的终端设备特性都包含在termios结构中。  

```c
typedef unsigned char   cc_t;
typedef unsigned int    speed_t;
typedef unsigned int    tcflag_t;

#define NCCS 19
struct termios {
    tcflag_t c_iflag;       /* input mode flags */
    tcflag_t c_oflag;       /* output mode flags */
    tcflag_t c_cflag;       /* control mode flags */
    tcflag_t c_lflag;       /* local mode flags */
    cc_t c_line;            /* line discipline */
    cc_t c_cc[NCCS];        /* control characters */
};

struct termios2 {
    tcflag_t c_iflag;       /* input mode flags */
    tcflag_t c_oflag;       /* output mode flags */
    tcflag_t c_cflag;       /* control mode flags */
    tcflag_t c_lflag;       /* local mode flags */
    cc_t c_line;            /* line discipline */
    cc_t c_cc[NCCS];        /* control characters */
    speed_t c_ispeed;       /* input speed */
    speed_t c_ospeed;       /* output speed */
};
```
* c_iflag：输入标志，由终端设备驱动程序用来控制输入特性（剥除输入字节的第8位，允许输入奇偶校验等）；
* c_oflag：输出标志，控制输出特性（执行输出处理，将新行映照为CR/LF等）；
* c_cflag：控制标志，影响到RS-232串行线（忽略调制解调器的状态线，每个字符的一个或两个停止位等）；
* c_lfalg：本地标志：影响驱动程序和用户之间的界面（回送的开或关，可视的擦除符，允许终端产生的信号，对后台作业输出的控制停止信号等）。  
  
&emsp;&emsp;下表列出了所有可以更改影响终端设备特性的终端标志。其中POSIX.1定义的是SVR4和4.3+BSD都支持的，但是他们还有自己的扩充部分。
