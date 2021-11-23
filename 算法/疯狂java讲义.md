# 疯狂java讲义
## 第一章 Java语言概述与开发环境
#### 1.4.2 编译java程序
1. 新建文件，例如`Helloworld.java`
2. 运行`javac -d . Helloworld.java`进行编译
3. 会在目录下生成`Helloworld.class`文件
4. 运行`java 类名`，即`java Helloworld`即可

示例代码：
```java
public class Helloworld
{
    public static void main(String[] args)
    {
        System.out.println("Hello World");
        System.out.println("I am lst");
    }
}
```
&emsp;&emsp;注意，类名和文件名要一样。

#### 1.5.2 Java源文件的命名规则
&emsp;&emsp;Java文件的命名必须满足如下规则：  
1. 扩展名必须是`.java`  
2. 文件名通常是任意的，但是如果源码文件中定义了一个public类，那么该源文件的文件名必须与public类名相同。  
   1. 一个java源程序可以包含多个类定义，各个类彼此独立，只是定义在了同一个文件中而已。编译后会生成每个类对应的.class文件。
   2. 推荐做法是，一个java源文件通常只定义一个类；
   3. 源文件的文件名应该与源文件中定义的public类同名（如果定义了的话）。

## 第三章 数据类型和运算符
#### 3.2.2 标识符规则
&emsp;&emsp;标识符必须以字母、下划线、\$开头，其后可以包含任意的字母（此处的字母并不局限于26个英文字母）、数字、下划线和\$。

