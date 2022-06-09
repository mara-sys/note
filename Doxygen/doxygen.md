[官方文档](https://www.doxygen.nl/manual/index.html)
[参考链接](https://zhuanlan.zhihu.com/p/122523174)















# Doxygen 手册
## 2 安装
&emsp;&emsp;[windows 安装链接](https://www.doxygen.nl/files/doxygen-1.9.4-setup.exe)
&emsp;&emsp;安装完成以后将 doxygen 安装路径`C:\softinstall\doxygen\doxygen\bin`添加到环境变量中。
&emsp;&emsp;在 windows 中的命令行中输入 doxygen 会出现版本信息和帮助手册。

## 3 开始
### 3.1 创建配置文件
&emsp;&emsp;Doxygen 使用配置文件来确定其所有设置。每个项目都应获取自己的配置文件。项目可以由单个源文件组成，也可以递归扫描整个源码树。
&emsp;&emsp;为了简化配置文件的创建，doxygen 可以为您创建一个模板配置文件。要从命令行执行此操作，请使用`-g`选项：
```shell
doxygen -g <config-file>
```
&emsp;&emsp;其中 \<config-file> 是配置文件的名称。如果省略文件名，将创建一个名为 Doxyfile 的文件。如果名称为 \<config-file> 的文件已存在，则在生成配置模板之前，doxygen 会将其重命名为 \<config-file>.bak。如果您使用`-`作为文件名，则 doxygen 将尝试从标准输入（stdin）读取配置文件，这对于脚本编写非常有用。
&emsp;&emsp;配置文件的格式类似于简单的 Makefile 的格式。它由许多形式的 tags 组成：
```shell
TAGNAME = VALUE or
TAGNAME = VALUE1 VALUE2 ...
```
&emsp;&emsp;如果您不希望使用文本编辑器编辑配置文件，可以看`doxywizard`的使用，这是一个GUI前端，可以创建，读取和写入 doxygen 配置文件，并允许通过对话框输入配置选项来设置配置选项。
&emsp;&emsp;对于由几个 C 和/或 C++ 源文件和头文件组成的小型项目，您可以将 INPUT 标记留空，doxygen 将在当前目录中搜索源。
&emsp;&emsp;如果您有一个由源目录或树组成的较大项目，则应将一个或多个根目录分配给 INPUT 标记，并将一个或多个文件 patterns 添加到 FILE_PATTERNS 标记（例如）。只有与其中一种模式匹配的文件才会被解析（如果省略了模式，则对 doxygen 支持的文件类型使用典型模式列表）。要对源树进行递归解析，必须将 RECURSIVE tag 设置为 YES。要进一步微调解析的文件列表，可以使用 EXCLUDE 和 EXCLUDE_PATTERNS tags。例如，要从源代码树中省略所有 test 目录，可以使用：
```shell
EXCLUDE_PATTERNS = */test/*
```

### 3.2 运行 doxygen 






















