# T-HEAD CPU 调试技巧
## 第一章 使用介绍
&emsp;&emsp;一般来说 T-HEAD 调试器主要调试的是 C/C++ 程序和汇编语言。要进行程序的调试，首先在编译时，我们必须要把调试信息加到可执行文件中。使用编译器的 -g 参数可以做到这一点。如：
```shell
csky-abiv2-elf-gcc -g hello.c -o hello
csky-abiv2-elf-g++ -g hello.cpp -o hello
```
&emsp;&emsp;如果没有 -g，你将看不见程序的变量名，所代替的全是运行时的内存地址。

1. T-HEAD 调试器中运行 UNIX 的 shell 程序
&emsp;&emsp;在 gdb 环境中，可以执行 UNIX 的 shell 的命令，使用 T-HEAD 调试器的 shell 命令来完成：
```shell
shell <command string>
```
&emsp;&emsp;调用 UNIX 的 shell 来执行 `command string`，环境变量 SHELL 中定义的 UNIX 的 shell 将会被用来执行 `command string`，如果 SHELL 没有定义，那就使用 UNIX 的标准 shell： /bin/sh。（在 Windows 中使用 Command.com 或 cmd.exe）
&emsp;&emsp;还有一个 T-HEAD 调试器命令是 make：
```shell
make <make-args>
```
&emsp;&emsp;可以在 T-HEAD 调试器中执行 make 命令来重新 build 自己的程序。这个命令等价于 `shell make <make-args>`。

###### 注意：
&emsp;&emsp;注意: 由于 T-HEAD 调试器是用于调试 T-HEAD 体系结构的应用程序，那么如果用户需要在硬件目标板调试程序，则还需要先启动调试代理服务程序 CSkyDebugServer，具体请参考 CSkyDebugServer 的用户手册。

## 第二章 GDB 标准调试命令
### 2.1 File 命令
&emsp;&emsp;用于打开被调试的程序，并根据文件基本信息，更新调试器内部状态，如大小端等。不管启动 T-HEAD 调试器是否已经打开被调试的程序文件，您都可以使用该命令来指定或重新指定被调试程序。
```shell
file 被调试文件名
```

### 2.2 Target 命令
&emsp;&emsp;


### 2.3 Directory 命令



### 2.4 Run 命令
&emsp;&emsp;当程序被 T-HEAD 调试器停住时，可以使用 info program 来查看程序被暂停的原因。
&emsp;&emsp;在 T-HEAD 调试器中，我们可以有以下几种暂停方式：
&emsp;&emsp;断点（BreakPoint）、（WatchPoint）停止程序运行；CTRL+C 停止运行的程序。
&emsp;&emsp;如果要恢复程序运行，可以使用 c 或是 continue 命令。

### 2.5 设置断点（BreakPoint）
&emsp;&emsp;我们用`break`或`b`命令来设置断点，例如：
```shell
b * 程序代码断地址
b 函数名
b 文件名：行号
b 当前程序 pc 指针所在文件行号
break +offset 或 b +offset
break -offset 或 b –offset
```
&emsp;&emsp;在当前行号的前面或后面的 offset 行停住。offset 为自然数。

###### 注解：
offset 的范围为：只要存在（当前行数 +offset）这一行就可以。

&emsp;&emsp;我们用 hb 来设置硬件断点。







&emsp;&emsp;在嵌入式系统中，如果想调试的程序不是位于内存中，而是位于像闪存这样的存储器中，此时就无法使用软件程序断点了，因为闪存中的内容并不像内存那样方便更改。
&emsp;&emsp;此时只能使用硬件程序断点来调试程序。硬件程序断点的实现原理与软件程序断点完全不同，断点时通过配置处理器的断点寄存器的方式来实现的。
&emsp;&emsp;当处理器运行到断点寄存器所指示位置的指令时就会产生中断，调试工具通过该中断使我们获得干预的机会。处理器所能设置的硬件程序断点数量是有限的，可能最多也就4个。

### 2.6 设置观察点（WatchPoint）
&emsp;&emsp;用`watch`命令来设置观察点。
&emsp;&emsp;观察点一般来观察某个表达式（变量也是一种表达式）的值是否有变化了，如果有变化，马上停住程序。我们有下面的几种方法来设置观察点：
```shell
watch 表达式
```
&emsp;&emsp;查看观察点时，可使用`info`命令。
```shell
info watchpoints
```
&emsp;&emsp;列出当前所设置的所有观察点。

### 2.7 维护停止点（断点或观察点）
&emsp;&emsp;在 T-HEAD 调试器中，如果已经定义好的停止点没有用了，可以使用 delete、clear、disable、enable 这几个命令来维护。































































































