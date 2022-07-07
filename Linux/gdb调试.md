# T-HEAD CPU 调试技巧
[toc]
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

#### 2.7.1 clear
1. 如果没有参数，删除当前行所有的断点。
2. 如果后面加行数，则删除改行所有的断点
3. 如果后面加函数名，则删除在函数首定义的所有断点。
```shell
delete [breakpoints] [range…]
```
&emsp;&emsp;删除指定的断点，breakpoints 为断点号。如果不指定断点号，则表示删除所有的断点。range 表示断点号的范围（如： 3-7 或者 3 4 5 6 7 用空格分隔）。其简写命令为 d。
&emsp;&emsp;比删除更好的一种方法是 disable 停止点，disable 了的停止点，T-HEAD 调试器不会删除，当你还需要时，enable 即可，就好像回收站一样。
```shell
disable [breakpoints] [range…]
```
&emsp;&emsp;disable 所指定的停止点， breakpoints 为停止点号。如果什么都不指定，表示 disable 所有的停止点。简写命令是 dis.
```shell
enable [breakpoints] [range…]
```
&emsp;&emsp;enable 所指定的停止点， breakpoints 为停止点号。简写为 en。

### 2.8 恢复程序运行和单步调试
```shell
continue 或 c
```
&emsp;&emsp;命令恢复程序的运行直到程序结束，或下一个断点到来。
```shell
step <count>
```
&emsp;&emsp;单步跟踪，如果有函数调用，他会进入该函数。进入函数的前提是，**此函数被编译有 debug 信息**。
```shell
next <count> 或 n <count>
```
&emsp;&emsp;同样单步跟踪，如果有函数调用，他不会进入该函数。
```shell
stepi 或 si
nexti 或 ni
```
&emsp;&emsp;单步跟踪一条机器指令。一条程序代码有可能由数条机器指令完成，stepi 和 nexti 可以单步执行机器指令。

### 2.9 查看栈信息
&emsp;&emsp;当程序被停住了，你需要做的第一件事就是查看程序是在哪里停住的。当你的程序调用了一个函数，函数的地址，函数参数，函数内的局部变量都会被压入“栈”（Stack）中。你可以用 T-HEAD 调试器命令来查看当前的栈中的信息。
&emsp;&emsp;下面是一些查看函数调用栈信息的 T-HEAD 调试器命令：
```shell
backtrace 或 bt
```
&emsp;&emsp;打印当前的函数调用栈的所有信息。如：
```shell
(cskygdb) bt
#0 func (n=250) at tst.c:6
#1 0x08048524 in main (argc=1, argv=0xbffff674) at tst.c:30
#2 0x400409ed in __libc_start_main () from /lib/libc.so.6
```
&emsp;&emsp;根据以上输出可以看出函数的调用栈信息： __libc_start_main –> main() –> func()
&emsp;&emsp;GDB 跳转到相应的堆栈 Frame 命令：
```shell
frame Id
(cskygdb) frame 2 ffff 跳到 frame 2，即 __libc_start_main 的 frame
```
&emsp;&emsp;用户可以查看当前堆栈 frame 的一些局部变量等信息。

### 2.10 查看源程序
&emsp;&emsp;显示源代码 list 或 l
&emsp;&emsp;T-HEAD 调试器可以打印出所调试程序的源代码，当然，在程序编译时一定要加上-g 的参数，把源程序信息编译到执行文件中。不然就看不到源程序了。当程序停下来以后， T-HEAD 调试器会报告程序停在了那个文件的第几行上。你可以用 list 命令来打印程序的源代码。
```shell
list 或 l
```
&emsp;&emsp;显示当前行后面的源程序。一般是打印当前行的上 5 行和下 5 行。
```shell
1.disassemble
```
&emsp;&emsp;查看源程序的当前执行时的机器码，这个命令会把目前内存中的指令 dump 出来。如下面的示例表示查看函数 func 的汇编代码。可以如下执行命令：
```shell
disassemble
disassemble 地址值, 地址值 2
disassemble $pc,$pc+offset
disassemble func
```

### 2.11 查看运行时数据
&emsp;&emsp;可以使用 printf 命令（p），或是同义命令 inspect 来查看当前程序的运行数据。printf 命令的格式是：
```shell
print <expr>
print /<f> <expr>
```
&emsp;&emsp;`<expr>`是表达式，是你所调试的程序的语言的表达式，`<f>`是输出的格式，比如，如果要把表达式按 16 进制的格式输出，那么就是 /x。
&emsp;&emsp;一般来说，T-HEAD 调试器会根据变量的类型输出变量的值。但也可以自定义 T-HEAD 调试器的输出格式。例如，想输出一个整数的十六进制，或是二进制来查看这个整型变量的中的位的情况。要做到这样，你可以使用 T-HEAD 调试器的数据显示格式：
* x 按十六进制格式显示变量。
* d 按十进制格式显示变量。
* u 按十六进制格式显示无符号整型。
* o 按八进制格式显示变量。
* t 按二进制格式显示变量。
* a 按十六进制格式显示变量。
* c 按字符格式显示变量。
* f 按浮点数格式显示变量。

### 2.12 查看内存
&emsp;&emsp;可以使用 examine 命令（x）来查看内存地址中的值。x 命令的语法如下所示：
```shell
x/<n/f/u> <addr>
```
&emsp;&emsp;n、f、u 是可选的参数。
* n 是一个正整数，表示显示内存的长度，也就是说从当前地址向后显示几个地址的内容。
* f 表示显示的格式，参见上面。如果地址所指的是字符串，那么格式可以是 s，如果地址是指令地址，那么格式可以是 i。
* u 表示从当前地址往后请求的字节数，如果不指定的话， T-HEAD 调试器默认是 4 个 bytes。 u 参数可以用下面的字符来代替， b 表示单字节， h 表示双字节， w 表示四字节， g 表示八字节。当我们指定了字节长度后， T-HEAD 调试器会从指内存定的内存地址开始，读写指定字节，并把其当作一个值取出来。

&emsp;&emsp;`<addr>`表示一个内存地址。
&emsp;&emsp;n/f/u 三个参数可以一起使用。例如：
&emsp;&emsp;命令： x/3uh 0x54320 表示，从内存地址 0x54320 读取内容，h 表示以双字节为一个单位，3 表示三个单位，u 表示按十六进制显示。

### 2.13 自动显示
&emsp;&emsp;可以设置一些自动显示的变量，当程序停住时，或是在单步跟踪时，这些变量会自动显示。相关的 T-HEAD 调试器命令是 display。
```shell
display <expr>
display/<fmt> <expr>
display/<fmt> <addr>
```
&emsp;&emsp;expr 是一个表达式，fmt 表示显示的格式，addr 表示内存地址，当你用 display 设定好了一个或多个表达式后，只要你的程序被停下来，T-HEAD 调试器会自动显示你所设置的这些表达式的值。
&emsp;&emsp;格式 i 和 s 同样被 display 支持，一个非常有用的命令是：
```shell
display/i $pc
```
&emsp;&emsp;$pc 是 T-HEAD 调试器的环境变量，表示着指令的地址， /i 则表示输出格式为机器指令码，也就是汇编。于是当程序停下后，就会出现源代码和机器指令码相对应的情形，这是一个很有意思的功能。

### 2.14 维护自动显示
&emsp;&emsp;在 T-HEAD 调试器中，如果觉得已定义好的自动显示点没有用了，可以使用 delete、disable、enable 这几个命令来维护。
```shell
delete display [display points]
```
&emsp;&emsp;删除指定的断点， display points 为自动显示号。如果不指定断点号，则表示删除所有的断点。 range 表示断点号的范围（如 3 4 5 6 7 用空格分隔）。其简写命令为 d display。
&emsp;&emsp;另外一种方法是 disable 自动显示点，disable 了的自动显示点，T-HEAD 调试器不会删除，当你还需要时，enable 即可。
```shell
disable display [display points]
```
&emsp;&emsp;disable 所指定的停止点，display points 为停止点号。如果什么都不指定，表示 disable 所有的停止点。简写命令是 dis display。
```shell
enable display [display points]
```
&emsp;&emsp;enable 所指定的停止点，display points 为停止点号。简写为 en display。

###### 注意：
&emsp;&emsp;与维护停止点有所不同。比如 d 3 是删除 3 号断点。 d display 3 是删除 3 号自动显示点。可以写 d 3-5 但是不能写 d display 3-5 只能是 d display 3 4 5。

### 2.15 查看寄存器
&emsp;&emsp;要查看寄存器，很简单，可以使用如下命令：
```shell
info registers
```
&emsp;&emsp;查看 CPU 通用寄存器和 psr, epsr, pc, epc。
```shell
info registers <regname…>
```
&emsp;&emsp;查看具体的某个寄存器， regname 表示具体寄存器名字，可以是多个。
&emsp;&emsp;寄存器中放置了运行时数据，比如程序当前运行的指令地址（pc），程序的当前堆栈指针（sp）等等。同样可以使用 print 命令来访问寄存器的情况，只需要在寄存器名字前加一个 $ 符号就可以了。如：p $pc。
&emsp;&emsp;由于 CSKY CPU 内部寄存器较多，csky-abiv2-elf-gdb 内部对所有 CPU 对应的寄存器进行了分组， csky-abiv2-elf-gdb 支持寄存器的分组查看功能，只需要在普通查看下加上寄存器组名即可。
```shell
info registers general
```
&emsp;&emsp;查看 CPU 通用寄存器。
```shell
info registers cr
```
&emsp;&emsp;查看 CPU 全部控制寄存器。
```shell
info registers fr
```
&emsp;&emsp;查看 CPU 全部浮点寄存器。
```shell
info registers vr
```
&emsp;&emsp;查看 CPU 全部向量寄存器，仅存在与 CK810 处理器中。
```shell
info registers mmu
```
&emsp;&emsp;查看 CPU 全部内存管理相关的控制寄存器。
```shell
info registers profiling
```
&emsp;&emsp;查看 CPU 全部 profiling 相关寄存器，仅存在于 CK810 处理器中。

### 2.16 修改变量值
&emsp;&emsp;修改被调试程序运行时的变量值，在 T-HEAD 调试器中很容易实现，使用 T-HEAD 调试器的 set 命令即可完成。如：
```shell
(cskygdb) set x=4 （比如 set endcommand=7）
```
### 2.17 修改寄存器值
&emsp;&emsp;使用 set 命令即可完成。
```shell
(cskygdb) set $r0=4
```

### 2.18 restore 命令
&emsp;&emsp;调试器可以在动态调试中，随时将目标文件中的程序加载到目标板的相应的内存中。
```shell
restore <filename> <offset> [ <startaddr> <endaddr>]
```
&emsp;&emsp;其中 filename 是需要加载的文件名，可以使 elf 文件，也可以是 bin 文件或者 hex 文件， offset 是程序在文件中指定的地址与即将加载地址之间的偏移；可选参数 startaddr、 endaddr 如果给出，则表示仅向目标板加载这个地址区间的程序，超过这个地址范围的程序则会忽略。
&emsp;&emsp;我没看懂这个 offset 是什么意思，用的时候会出错，我实际使用的命令是下面这样子的：
```shell
restore filename binary startaddress
```
&emsp;&emsp;需要指定 binary，否则会报错`is not a recognized file format`。

### 2.19 dump/append 命令
&emsp;&emsp;调试器不仅支持动态的加载文件到内存，还支持实时的将目标板内存中的信息抽取出来，并以文件的形式保存下来，这个功能由 dump/append 命令来实现。
```shell
dump/append binary
```
&emsp;&emsp;将目标板的指令和数据以二进制流保存到文件。


### 2.20 call function 命令
&emsp;&emsp;在调试程序的时候，调试器允许用户自由的调用程序中的任意函数，用户可以通过自己输入函数的参数，然后观察函数返回值。
```shell
call <function(arg1,arg2…)>
```
&emsp;&emsp;其中，function 是函数名，arg 是参数列表，当命令完成后，调试器会输出函数执行的返回结果。

### 2.21 source 命令
&emsp;&emsp;调试器在开启后的任何时候，都可以通过 source 命令来执行脚本。
```shell
source <scriptfilepath>
```
&emsp;&emsp;其中`<scriptfilepath>`是脚本名称。

## 第三章 T-HEAD 拓展调试命令
### 3.1 pctrace 命令
&emsp;&emsp;CK-KPU 硬件上提供了一个记录 PC 跳转轨迹的单元，可以记录最近 8 次跳转指令的地址的值，可以通过 T-HEAD 调试器的 pctrace 命令来查看程序运行时的 PC 跳转轨迹。直接在命令栏输入 pctrace，或者其简写命令 pc 即可。
```shell
pctrace
```
&emsp;&emsp;对于结果，标号靠前的地址是最近的跳转轨迹。


















































