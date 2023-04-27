# 一、gcc 简介和命令行参数简介
[原文链接](https://blog.csdn.net/yueguangmuyu/article/details/116703618)
```MERMAID
graph TD
    c_file(hello.c)
    i_file[hello.i]
    s_file[hello.s]
    o_file[hello.o]
    out_file(a.out)
    c_file -->|gcc -E 预处理\n 展开宏和头文件\n替换条件编译\n删除注释,空行,空白| i_file 
    i_file -->|gcc -S 编译\n检查语法规范\n 消耗时间,系统资源最多| s_file 
    s_file -->|gcc -c 汇编\n将汇编指令翻译成\n机器指令| o_file 
    o_file-->|无参数\n链接\n数据段合并地址回填| out_file
```
## 1.1 gcc 基本用法
&emsp;&emsp;使用gcc编译器时，必须给出一系列必要的调用参数和文件名称。不同参数的先后顺序对执行结果没有影响，只有在使用同类参数时的先后顺序才需要考虑。 如果使用了多个 -L 的参数来定义库目录，gcc会根据多个 -L 参数的先后顺序来执行相应的库目录。
&emsp;&emsp;因为很多gcc参数都由多个字母组成，所以gcc参数不支持单字母的组合，Linux中常被叫短参数（short options），如 -dr 与 -d -r 的含义不一样。gcc编译器的调用参数大约有100多个，其中多数参数我们可能根本就用不到，这里只介绍其中最基本、最常用的参数。
&emsp;&emsp;gcc最基本的用法是：
```shell
gcc [options] [filenames]
```
&emsp;&emsp;其中，options就是编译器所需要的参数，filenames给出相关的文件名称，最常用的有以下参数：
* -c：只编译，不链接成为可执行文件。 编译器只是由输入的 .c 等源代码文件生成 .o 为后缀的目标文件，通常用于编译不包含主程序的子程序文件。
* -o output_filename：只编译，不链接成为可执行文件。 编译器只是由输入的 .c 等源代码文件生成 .o 为后缀的目标文件，通常用于编译不包含主程序的子程序文件。
* -g：产生符号调试工具（GNU的 gdb）所必要的符号信息。 想要对源代码进行调试，就必须加入这个选项。
* -O：对程序进行优化编译、链接。 采用这个选项，整个源代码会在编译、链接过程中进行优化处理，这样产生的可执行文件的执行效率可以提高，但是编译、链接的速度就相应地要慢一些，而且对执行文件的调试会产生一定的影响，造成一些执行效果与对应源文件代码不一致等一些令人“困惑”的情况。因此，一般在编译输出软件发行版时使用此选项。
* -O2：比 -O 更好的优化编译、链接。当然整个编译链接过程会更慢。
* −Idir：即 Include dir，将 dirname 所指出的目录加入到程序头文件目录列表中，是在预编译过程中使用的参数。
说明：C程序中的头文件包含两种情况：
```c
#include <stdio.h>
#include "stdio.h"
```
&emsp;&emsp;其中，使用尖括号（<>），预处理程序 cpp 在系统默认包含文件目录（如/usr/include）中搜索相应的文件；使用双引号，预处理程序 cpp 首先在当前目录中搜寻头文件，如果没有找到，就到指定的 dirname 目录中去寻找。在程序设计中，如果需要的这种包含文件分别分布在不同的目录中，就需要逐个使用 -I 选项给出搜索路径。
* −Ldir：即 Link dir，将dirname所指出的目录加入到程序函数库文件的目录列表中，是在链接过程中使用的参数。 在默认状态下，链接程序 ld 在系统默认路径中（如 /usr/lib）寻找所需要的库文件。这个选项告诉链接程序，首先到 -L 指定的目录中去寻找，然后到系统默认路径中寻找；如果函数库存放在多个目录下，就需要依次使用这个选项，给出相应的存放目录。
* −lname：链接时装载名为 libname.a 的函数库。 该函数库位于系统默认的目录或者由 -L 选项确定的目录下。 Linux下的库文件在命名时有一个约定，就是应该以 lib 这3个字母开头，由于所有的库文件都遵循了同样的规范，因此在用 -l 选项指定链接的库文件名时可以省去 lib 这3个字母。例如，gcc 在对 -lfoo 进行处理时，会自动去链接名为 libfoo.so 的文件，又如，-lm 表示链接名为 libm.a 的数学函数库。


# 二、跟我一起写 Makefile
## 2.2 关于程序的编译和链接
&emsp;&emsp;源文件首先会生成中间目标文件，再由中间目标文件生成执行文件。在编译时，编译器只检测程序语法，和函数、变量是否被声明无关。如果函数未被声明，编译器会给出一个警告，但可以生成中间目标文件。而在链接程序时，链接器会在所有的中间目标文件中找寻函数的实现，如果找不到，那就会报链接错误码。
## 2.3 Makefile 介绍
&emsp;&emsp;make 命令执行时，需要一个 Makefile 文件，以告诉 make 命令需要怎么样去编译和链接程序。
### 2.3.1 Makefile 的规则
&emsp;&emsp;先粗略的看一下 Makefile 的规则。
```shell
target ... : prerequisites ...
	command
    ...
    ...
```
* `target`也就是一个目标文件，可以是`Object File`，也可以是执行文件。还可以是一个标签（Label）。
* `prerequisites`就是要生成那个`target`所需要的文件或是目标。
* `command`也就是`make`需要执行的命令。（任意的 Shell 命令）

&emsp;&emsp;这是一个**文件的依赖关系**，也就是说，`target`这一个或多个的目标文件依赖于`prerequisites`中的文件，其生成规则定义在`command`中。说白一点就是说，`prerequisites`中如果有一个以上的文件比`target`文件要新的话，`command`所定义的命令就会被执行。这就是`Makefile`的规则。也就是`Makefile`中最核心的内容。

### 2.3.2 一个示例
&emsp;&emsp;正如前面所说的，如果一个工程有3个头文件，和8个C文件，我们为了完成前面所述的那三个规则，我们的Makefile应该是下面的这个样子的。
```Makefile
# 最终目标文件--可执行文件
edit : main.o kbd.o command.o display.o /
			insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o /
    insert.o search.o files.o utils.o
			
# 中间目标文件
main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
            
# 伪目标文件
clean :
rm edit main.o kbd.o command.o display.o /
	insert.o search.o files.o utils.o
```
&emsp;&emsp;`反斜杠（/）`是换行符的意思。这样比较便于`Makefile`的易读。我们可以把这个内容保存在文件为`Makefile`或`makefile`的文件中，然后在该目录下直接输入命令`make`就可以生成执行文件`edit`。如果要删除执行文件和所有的中间目标文件，那么，只要简单地执行一下`make clean`就可以了。
* 目标文件（target）包含：执行文件（edit）和中间目标文件（*.o）
* 依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h文件。

&emsp;&emsp;这里要说明一点的是，`clean`不是一个文件，它只不过是一个动作名字，有点像C语言中的`lable`一样，其冒号后什么也没有，那么，`make`就不会自动去找文件的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在`make`命令后明显得指出这个`lable`的名字。这样的方法非常有用，我们可以在一个`makefile`中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。

### 2.3.3 make 是如何工作的
&emsp;&emsp;在默认的方式下，也就是我们只输入make命令。那么：
1. make会在当前目录下找名字叫Makefile或makefile的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到edit这个并把这个文件作为最终的目标文件。
3. 如果edit文件不存在，或是edit所依赖的后面的 .o 文件的修改时间要比edit这个文件新，那么，他就会执行后面所定义的命令来生成edit这个文件。
4. 如果edit所依赖的.o文件不存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到，则根据对应的依赖规则生成.o文件。（这有点像一个堆栈的过程）
5. 当然，你的C文件和H文件是存在的啦，于是make会生成.o 文件，然后再用.o 文件生成make的终极任务，也就是执行文件edit了。

&emsp;&emsp;这就是整个make的依赖性，make会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在寻找的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。make只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。
&emsp;&emsp;通过上述分析，我们知道，像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显示要make执行。即执行命令make clean，以此来清除所有的目标文件，以便重编译。
&emsp;&emsp;于是在我们编程中，如果这个工程已被编译过了，当我们修改了其中一个源文件，比如file.c，那么根据我们的依赖性，我们的目标file.o会被重编译（也就是在这个依性关系后面所定义的命令），于是file.o的文件也是最新的啦，于是file.o的文件修改时间要比edit要新，所以edit也会被重新链接了（详见edit目标文件后定义的命令）。
&emsp;&emsp;而如果我们改变了command.h，那么，kdb.o、command.o和iles.o都会被重编译，并且，edit会被重链接。

### 2.3.4 makefile 中使用变量
&emsp;&emsp;在`makefile`中我们可以使用变量，理解成 C 语言中的宏可能会更好。
&emsp;&emsp;例如：
```Makefile
objects = main.o kbd.o command.o display.o /
	insert.o search.o files.o utils.o
```
&emsp;&emsp;我们就可以很方便地在我们的`makefile`中以`$(objects)`的方式来使用这个变量了。

### 2.3.5 让 make 自动推导
&emsp;&emsp;`GNU`的`make`很强大，它可以自动推导文件以及文件依赖关系后面的命令，于是我们就没必要去在每一个`.o`文件后都写上类似的命令，因为，我们的`make`会自动识别，并自己推导命令。
&emsp;&emsp;只要`make`看到一个`.o`文件，它就会自动的把`.c`文件加在依赖关系中，如果`make`找到一个`whatever.o`，那么`whatever.c`，就会是`whatever.o`的依赖文件。并且`cc -c whatever.c`也会被推导出来，于是，我们的`makefile`再也不用写得这么复杂。我们的是新的`makefile`又出炉了。
```makefile
# 变量定义
objects = main.o kbd.o command.o display.o /
	insert.o search.o files.o utils.o
# 最终目标文件
edit : $(objects)
	cc -o edit $(objects)
# 中间目标文件
main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h
# 伪目标文件
.PHONY : clean
clean :
	rm edit $(objects)
```
&emsp;&emsp;这种方法，也就是`make`的隐晦规则。

### 2.3.6 另类风格的 makefile

### 2.3.7 清空目标文件的规则

## 2.4 Makefiel 总述
### 2.4.2 Makefile 的文件名
&emsp;&emsp;默认的情况下，`make`命令会在当前目录下按顺序找寻文件名为`GNUmakefile`、`makefile`、`Makefile`的文件，找到了解释这个文件。在这三个文件名中，最好使用`Makefile`这个文件名，因为，这个文件名第一个字符为大写，这样有一种显目的感觉。最好不要用`GNUmakefile`，这个文件是`GNU`的`make`识别的。有另外一些`make`只对全小写的`makefile`文件名敏感，但是基本上来说，大多数的`make`
都支持makefile和Makefile这两种默认文件名。
&emsp;&emsp;当然，你可以使用别的文件名来书写`Makefile`，比如：`Make.Linux`，`Make.Solaris`，`Make.AIX`等，如果要指定特定的`Makefile`，你可以使用`make`的`-f`和`--file`参数，如：
```shell
make -f Make.Linux
make --file Make.AIX
```

### 2.4.3 引用其它的 Makefile
&emsp;&emsp;在`Makefile`使用`include`关键字可以把别的`Makefile`包含进来，这很像 C 语言的`#include`，被包含的文件会原模原样的放在当前文件的包含位置。`include`的语法是：
```Makefile
include <filename>
```
filename可以是当前操作系统Shell的文件模式（可以包含路径和通配符）

在include前面可以有一些空字符，但是绝不能是[Tab]键开始。include和<filename>可以用一个或多个空格隔开。举个例子，你有这样几个Makefile：a.mk、b.mk、c.mk，还有一个文件叫foo.make，以及一个变量$(bar)，其包含了e.mk和f.mk，那么，下面的语句：
























































