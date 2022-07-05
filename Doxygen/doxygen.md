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
&emsp;&emsp;要生成文档，可以输入：
```shell
doxygen <config-file>
```
&emsp;&emsp;根据您的设置，doxygen 将在输出目录中创建 html、rtf、latex、xml、man，和/或 docbook 目录。顾名思义，这些目录包含 HTML、RTF、LATEX、Unix-Man 页面和 DocBook 格式的生成文档。
&emsp;&emsp;默认输出目录是命令行启动 doxygen 的目录。可以使用`OUTPUT_DIRECTORY`更改输出的根目录。可以使用`HTML_OUTPUT`、`RTF_OUTPUT`、`LATEX_OUTPUT`、`XML_OUTPUT`、`MAN_OUTPUT`和`DOCBOOK_OUTPUT`选择输出目录中的特定格式目录。如果输出目录不存在，将尝试为您创建它（但它不会尝试以递归方式创建整个路径，像`mkdir -p`一样）。

#### 3.2.1 HTML 输出
&emsp;&emsp;可以用 HTML 浏览器来查看产生的 HTML 文档，文档是位于 html 路径下的`index.html`。

#### 3.2.2 LaTeX 输出


### 3.3 注释源码
&emsp;&emsp;虽然注释源码是作为步骤 3 呈现的，但在一个新项目中，这当然应该是第一步。
&emsp;&emsp;如果配置文件中将`EXTRACT_ALL`选项设置为`NO`，doxygen 将仅为注释了的实体生成文档。有两个基本的选项来注释这些成员、类和命名空间：
1. 在成员、类或命名空间的声明或定义前面放置一个特殊的文档块。对于文件、类和命名空间成员，还允许将文档直接放在成员之后。参见 4.1 节。
2. 将特殊的文档块放在其他位置（另一个文件或另一个位置），并在文档块中放置一个结构命令。结构命令将文档块链接到可以记录的某个实体（例如，成员，类，命名空间或文件）。参见 4.1.1.3 节。
   
## 4 注释源码
本章涵盖两个主题：
1. 如何在代码中放置注释，以便 doxygen 将它们合并到它生成的文档中。这将在下一节中进一步详细介绍。
2. 如何构建注释块的内容，使输出看起来不错，如注释块剖析一节中所述。

### 4.2 特殊注释块
&emsp;&emsp;特殊注释块是带有一些附加标记的 C 或 C++ 样式注释块，因此 doxygen 知道它是一段结构化文本，需要最终出现在生成的文档中。下一节介绍 doxygen 支持的各种样式。
&emsp;&emsp;对于 Python，VHDL 和 Fortran 代码，有不同的注释约定，可以分别在 Python，VHDL 和 Fortran 中的注释块中找到。

#### 4.2.1 类 C 语言注释块
&emsp;&emsp;对于代码中的每个实体，都有两种（或在某些情况下三种）类型的描述，它们共同构成了该实体的注释；简要描述和详细说明，两者都是可选的。对于方法和函数，还有第三种类型的描述，即所谓的 body 描述，它由在方法或函数的主体中找到的所有注释块的组成。
&emsp;&emsp;允许使用多个简短或详细的描述（但不建议这样做，因为未指定描述的显示顺序）。
&emsp;&emsp;顾名思义，简要描述是简短的单行说明，而详细说明则提供更长、更详细的文档。“in body”描述也可以用作详细说明，也可以描述实现细节的集合。对于 HTML 输出，简要说明还用于在引用项的位置提供工具提示。
&emsp;&emsp;有几种方法可以将注释块标记为详细描述：
1. 可以使用 Javadoc 风格，它包括一个 C 风格的注释块，注释块由两个 * 开始，例如：
```c
/**
 * ... text ...
 */
```
2. 也可以使用 Qt 风格并在 C 风格注释块后添加一个 !，例如：
```c
/*!
 * ... text ...
 */
```

&emsp;&emsp;在这两种情况下，中间的 * 都是可选的，因此
```c
/*!
    ... text ...
 */
```
&emsp;&emsp;也是有效的。

3. 第三种方法是使用至少两个C++注释行的块，其中每行都以额外的斜杠或感叹号开头。以下是两种情况的示例：
```c
///
/// ... text ...
///
```
&emsp;&emsp;或
```c
//!
//!... text ...
//!
```
&emsp;&emsp;请注意，在这种情况下，空行将结束文档块。

4. 有些人喜欢让他们的评论块在文档中更加明显。为此，您可以使用以下命令：
```c
/********************************************//**
 *  ... text
 ***********************************************/
```
&emsp;&emsp;注意：2 个斜杠结束正常的注释块并启动一个特殊的注释块。
&emsp;&emsp;注意：使用像 clang 格式这样的重新格式化程序时要小心，因为它可能会将这种类型的注释视为2个单独的注释，并在它们之间引入间距。
&emsp;&emsp;或
```c
/////////////////////////////////////////////////
/// ... text ...
/////////////////////////////////////////////////
```
&emsp;&emsp;或
```c
/************************************************
 *  ... text
 ***********************************************/
```
&emsp;&emsp;只要`JAVADOC_BANNER = YES`
```c
/**
 * JavaDoc 风格（C 风格）注释简史.
 *
 * 这是典型的 JavaDoc 样式的 C 样式注释。它从两个 * 开始.
 *
 * @param theory Even if there is only one possible unified theory. it is just a
 *               set of rules and equations.
 */
void cstyle( int theory );
 
/*******************************************************************************
 * JavaDoc 风格（C 风格）横幅注释简史.
 *
 * 这是典型的 JavaDoc 风格的 C 风格“横幅”注释. 它有一个 / 开始，后面跟 n 个 *，n > 2.
 * 这种方式写的注释对于正在写源码的开发者来说更显眼。
 *
 * 通常，开发者不知道这不是（默认情况下）一个有效的 Doxygen 注释块.
 *
 * 但是，只要在 Doxyfile 中加上 JAVADOC_BLOCK = YES，它就会按预期般工作. 
 *
 * 这种注释风格在clang格式中表现良好。
 *
 * @param theory Even if there is only one possible unified theory. it is just a
 *               set of rules and equations.
 ******************************************************************************/
void javadocBanner( int theory );
 
/***************************************************************************//**
 * Doxygen 风格的横幅注释简史.
 *
 * 这是一个 Doxygen 风格的 C 风格横幅注释. 它由一个正常的注释开始，然后再第一行的结尾
 * 转成特殊的注释块，当开发者在写源码时这种写法更显眼。
 * 这种注释风格在 clang 格式下表现不佳。
 *
 * @param theory Even if there is only one possible unified theory. it is just a
 *               set of rules and equations.
 ******************************************************************************/
void doxygenBanner( int theory );
```
&emsp;&emsp;对于 brief 描述，有几种可供使用的方法：
1. 可以将 \brief 命令与上述注释块之一一起使用。此命令在段落末尾结束，因此详细说明跟在段落末尾的空行之后。
&emsp;&emsp;下面是一个示例：
```c
/*! \brief Brief description.
 *         Brief description continued.
 *
 *  Detailed description starts here.
 */
```
2. 如果在配置文件中设置了 JAVADOC_AUTOBRIEF = YES，则使用 Javadoc 样式的注释块将自动开始一个简短的描述，该描述以第一个句点 . 结尾，后跟空格或新行。下面是一个示例：
```c
/** Brief description which ends at this dot. Details follow
 *  here.
 */
```
&emsp;&emsp;该选项对多行特殊 C++ 注释具有相同的效果：
```c
/// Brief description which ends at this dot. Details follow
/// here.
```
3. 第三种选择是使用特殊的C++样式注释，该注释不会跨越多行。下面是两个示例：
```c
/// Brief description.
/** Detailed description. */
```
&emsp;&emsp;或
```c
//! Brief description.

//! Detailed description
//! starts here.
```
&emsp;&emsp;请注意最后一个示例中的空行，这是将简要说明与包含详细说明的块分开所必需的。对于这种情况，JAVADOC_AUTOBRIEF也应设置为 NO。

&emsp;&emsp;与大多数其他文档系统不同，doxygen 还允许您将成员的文档（包括全局函数）放在定义之前。这样，文档可以放在源文件中，而不是放在头文件中。这使头文件保持紧凑，并允许成员的实现者更直接地访问文档。作为折衷办法，可以在声明之前进行简要说明，在成员定义之前放置详细说明。

##### 4.2.1.1 将注释放在成员之后
&emsp;&emsp;如果要记录文件、结构、联合、类或枚举的成员，有时需要将文档块放在成员之后而不是之前。为此，您必须在注释块中添加一个额外的 < 标记。请注意，这也适用于函数的参数。
&emsp;&emsp;以下是一些示例：
```c
int var; /*!< Detailed description after the member */
```
&emsp;&emsp;这个块可以用来把Qt样式的详细文档块放在一个成员之后。执行相同操作的其他方法包括：
```c
int var; /**< Detailed description after the member */
```
&emsp;&emsp;或
```c
int var; //!< Detailed description after the member
         //!<
```
&emsp;&emsp;或
```c
int var; ///< Detailed description after the member
         ///<
```
&emsp;&emsp;大多数情况下，人们只想在成员之后添加简短的描述。具体操作如下：
```c
int var; //!< Brief description after the member
```
&emsp;&emsp;或
```c
int var; ///< Brief description after the member
```
&emsp;&emsp;对于函数，可以使用 @param 命令来记录参数，然后使用 [in]、[out]、[in,out] 来记录方向。对于内联注释，也可以从 direction 属性开始。
```c
void foo(int v /**< [in] docs for input parameter v. */);
```
&emsp;&emsp;请注意，这些块与上一节中的特殊注释块具有相同的结构和含义，只有 < 指示成员位于块的前面而不是块之后。
&emsp;&emsp;以下是使用这些注释块的示例：
```c
/*! A test class */
 
class Afterdoc_Test
{
    public:
        /** An enum type. 
         *  The documentation block cannot be put after the enum! 
         */
        enum EnumType
        {
          int EVal1,     /**< enum value 1 */
          int EVal2      /**< enum value 2 */
        };
        void member();   //!< a member function.
    
    protected:
        int value;       /*!< an integer value */
};
```
**警告**
&emsp;&emsp;这些块只能用于注释 members 和 parameters。它们不能用于注释 files, classes, unions, structs, groups, namespaces, macros, 和 enums。此外，下一节中提到的结构命令（如 \class）不允许在这些注释块中使用。
&emsp;&emsp;用此构造作为宏定义的一部分时要小心，因为当 MACRO_EXPANSION 设置为 YES 应用宏的位置时，注释也会被替换，然后此注释将用作遇到的最后一项的文档，而不是宏定义本身的文档！

##### 4.2.1.2 示例
&emsp;&emsp;下面是使用 Qt 样式记录的 C++ 代码段的示例：
```c
//!  A test class. 
/*!
  A more elaborate class description.
*/
 
class QTstyle_Test
{
  public:
 
    //! An enum.
    /*! More detailed enum description. */
    enum TEnum { 
                 TVal1, /*!< Enum value TVal1. */  
                 TVal2, /*!< Enum value TVal2. */  
                 TVal3  /*!< Enum value TVal3. */  
               } 
         //! Enum pointer.
         /*! Details. */
         *enumPtr, 
         //! Enum variable.
         /*! Details. */
         enumVar;  
    
    //! A constructor.
    /*!
      A more elaborate description of the constructor.
    */
    QTstyle_Test();
 
    //! A destructor.
    /*!
      A more elaborate description of the destructor.
    */
   ~QTstyle_Test();
    
    //! A normal member taking two arguments and returning an integer value.
    /*!
      \param a an integer argument.
      \param s a constant character pointer.
      \return The test results
      \sa QTstyle_Test(), ~QTstyle_Test(), testMeToo() and publicVar()
    */
    int testMe(int a,const char *s);
       
    //! A pure virtual member.
    /*!
      \sa testMe()
      \param c1 the first argument.
      \param c2 the second argument.
    */
    virtual void testMeToo(char c1,char c2) = 0;
   
    //! A public variable.
    /*!
      Details.
    */
    int publicVar;
       
    //! A function variable.
    /*!
      Details.
    */
    int (*handler)(int a,int b);
};
```

......


&emsp;&emsp;缺省情况下，Javadoc 样式文档块的行为方式与 Qt 样式文档块的行为方式相同。然而，这并不是根据 Javadoc 规范，其中文档块的第一句话自动被视为简要描述。若要启用此行为，应在配置文件中将 JAVADOC_AUTOBRIEF 设置为 YES。如果启用此选项并希望在句子中间放置一个点而不结束它，则应在其后放置反斜杠和空格。下面是一个示例：
```c
/** Brief description (e.g.\ using only a few words). Details follow. */
```
&emsp;&emsp;下面是与上面相同的代码段，这次使用 Javadoc 风格的注释，且将 JAVADOC_AUTOBRIEF 设置为 YES：
```c
/**
 *  A test class. A more elaborate class description.
 */
 
class Javadoc_Test
{
  public:
 
    /** 
     * An enum.
     * More detailed enum description.
     */
 
    enum TEnum { 
          TVal1, /**< enum value TVal1. */  
          TVal2, /**< enum value TVal2. */  
          TVal3  /**< enum value TVal3. */  
         } 
       *enumPtr, /**< enum pointer. Details. */
       enumVar;  /**< enum variable. Details. */
       
      /**
       * A constructor.
       * A more elaborate description of the constructor.
       */
      Javadoc_Test();
 
      /**
       * A destructor.
       * A more elaborate description of the destructor.
       */
     ~Javadoc_Test();
    
      /**
       * a normal member taking two arguments and returning an integer value.
       * @param a an integer argument.
       * @param s a constant character pointer.
       * @see Javadoc_Test()
       * @see ~Javadoc_Test()
       * @see testMeToo()
       * @see publicVar()
       * @return The test results
       */
       int testMe(int a,const char *s);
       
      /**
       * A pure virtual member.
       * @see testMe()
       * @param c1 the first argument.
       * @param c2 the second argument.
       */
       virtual void testMeToo(char c1,char c2) = 0;
   
      /** 
       * a public variable.
       * Details.
       */
       int publicVar;
       
      /**
       * a function variable.
       * Details.
       */
       int (*handler)(int a,int b);
};
```

##### 4.2.1.3 其他地方的文档
&emsp;&emsp;在上一节的示例中，注释块始终位于文件、类或命名空间或定义的前面，或者位于其成员的前面或后面。虽然这通常很舒适，但有时可能有理由将文档放在其他地方。对于记录文件，这甚至是必需的，因为没有“在文件前面”这样的东西。
&emsp;&emsp;Doxygen 允许您将文档块放在几乎任何地方（例外情况是在函数体内部或普通的C样式注释块内）。
&emsp;&emsp;不将文档块直接放在项目之前（或之后）所付出的代价是需要在文档块内放置一个结构命令，这会导致一些重复的信息。因此，在实践中，您应该避免使用结构命令，除非其他要求迫使您这样做。
&emsp;&emsp;结构命令（与所有其他命令一样）以反斜杠（\） 开头，如果您更喜欢 Javadoc 样式，则以 at 符号（@）开头，后跟命令名称和一个或多个参数。例如，如果要在上面的示例中记录类，还可以将以下文档块放在 doxygen 读取的输入中的某个位置：
```c
/*! \class Test
    \brief A test class.

    A more detailed class description.
*/
```
&emsp;&emsp;在这里，特殊的命令 \command 用于指示注释块包含了 Test 类的注释。其他的 structural 命令是：
* \struct to document a C-struct.
* \union to document a union.
* \enum to document an enumeration type.
* \fn to document a function.
* \var to document a variable or typedef or enum value.
* \def to document a #define.
* \typedef to document a type definition.
* \file to document a file.
* \namespace to document a namespace.
* \package to document a Java package.
* \interface to document an IDL interface.

&emsp;&emsp;若要记录 C++ 类的成员，还必须记录该类本身。这同样适用于命名空间。要记录全局 C 函数、typedef、枚举或预处理器定义，必须首先记录包含它的文件（通常这将是一个头文件，因为该文件包含导出到其他源文件的信息）。

**注意**
&emsp;&emsp;让我们重复一遍，因为它经常被忽视：要记录全局对象（函数，typedefs，枚举，宏等），您必须记录定义它们的文件。换句话说，此文件中必须至少有一行：
```c
/*! \file */ 
```
&emsp;&emsp;或
```c
/** @file */ 
```
&emsp;&emsp;下面是一个 structcmd.h 的 C 头文件，该头文件使用结构命令进行记录
```c
/*! \file structcmd.h
    \brief A Documented file.
    
    Details.
*/
 
/*! \def MAX(a,b)
    \brief A macro that returns the maximum of \a a and \a b.
   
    Details.
*/
 
/*! \var typedef unsigned int UINT32
    \brief A type definition for a .
    
    Details.
*/
 
/*! \var int errno
    \brief Contains the last error code.
 
    \warning Not thread safe!
*/
 
/*! \fn int open(const char *pathname,int flags)
    \brief Opens a file descriptor.
 
    \param pathname The name of the descriptor.
    \param flags Opening flags.
*/
 
/*! \fn int close(int fd)
    \brief Closes the file descriptor \a fd.
    \param fd The descriptor to close.
*/
 
/*! \fn size_t write(int fd,const char *buf, size_t count)
    \brief Writes \a count bytes from \a buf to the filedescriptor \a fd.
    \param fd The descriptor to write to.
    \param buf The data buffer to write.
    \param count The number of bytes to write.
*/
 
/*! \fn int read(int fd,char *buf,size_t count)
    \brief Read bytes from a file descriptor.
    \param fd The descriptor to read from.
    \param buf The buffer to read into.
    \param count The number of bytes to read.
*/
 
#define MAX(a,b) (((a)>(b))?(a):(b))
typedef unsigned int UINT32;
int errno;
int open(const char *,int);
int close(int);
size_t write(int,const char *, size_t);
int read(int,char *,size_t);
```
&emsp;&emsp;由于上述示例中的每个注释块都包含一个结构命令，因此所有注释块都可以移动到另一个位置或输入文件（例如源文件），而不会影响生成的文档。这种方法的缺点是原型是重复的，因此所有更改都必须进行两次！因此，您应该首先考虑是否真的需要这样做，并尽可能避免使用结构命令。我经常收到一些示例，这些示例在位于函数前面的注释块中包含 \fn 命令。这显然是 \fn 命令是冗余的，只会导致问题。
&emsp;&emsp;当你在具有以下扩展名之一的文件中放置注释块时，.dox, .txt, .doc, .md 或 .markdown 或者当扩展名通过 EXTENSION_MAPPING 映射时，doxygen 将从文件列表中隐藏此文件。
&emsp;&emsp;如果您有一个 doxygen 无法解析的文件，但仍想记录它，则可以使用 \verbinclude 按原样显示它，例如
```c
/*! \file myscript.sh
 *  Look at this nice script:
 *  \verbinclude myscript.sh
 */
```
&emsp;&emsp;确保该脚本在 INPUT 中显式列出，或者 FILE_PATTERNS 包含 .sh 扩展名，并且可以通过 EXAMPLE_PATH 在设置的路径中找到该脚本。

#### 4.2.2 注释块剖析







































