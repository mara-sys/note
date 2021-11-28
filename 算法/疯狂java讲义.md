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
### 3.4 基本数据类型
基本数据类型
1. 整数类型
   1. 1字节：byte
   2. 2字节：short
   3. 4字节：int
   4. 8字节：long
2. 字符类型
   1. 2字节：char
3. 浮点类型
   1. 4字节：float
   2. 8字节：double
4. 布尔类型
   1. boolean
   
注意要点：
1. 整型  
   * 直接给出一个整数值默认是int类型
   * 如果使用一个巨大的整数值，java 不会自动把这个值当成long类型来处理，需要在其后加上`L`，如下所示：
```java
long bigValue = 498908408409384903L;
//下面的代码是错误的，因为系统不会把498908408409384903当成long来处理
long bigValue = 498908408409384903;
```
2. 字符型
   * 字符型值必须使用单引号括起来，java 使用16为unicode字符集作为编码方式。
3. 布尔型
   * boolean数值只能是true或false，不能是0或1。
4. java7 引入了一个新功能：可以在数值中使用下划线来更清楚的分辨位数。

#### 3.4.6 使用var定义变量
&emsp;&emsp;var相当于一个动态类型，使用var定义的局部变量的类型由编译器自动推断——定义变量时分配了什么类型的初始值，那么该变量就是什么类型。**因此，java使用var定义局部变量时，必须在定义局部变量的同时指定初始值，否则编译器无法推断该变量的类型。**
### 3.5 基本类型的类型转换
#### 3.5. 自动类型转换
&emsp;&emsp;当把一个表数范围小的数值或变量直接赋给另一个表数范围大的变量时，系统将可以进行自动类型转换；否则就需要强制转换。例如：
```java
//直接把5.6赋给float类型变量会出错，因为5.6默认是double类型
float a = 5.6;
```
&emsp;&emsp;当把任何基本类型的值和字符串进行连接时，基本类型的值将自动转换为字符串类型（虽然字符串类型不是基本类型，而是**引用类型**）。因此，如果希望把基本类型的值转换为字符串时，可以把基本类型的值和一个空字符串进行连接。
&emsp;&emsp;

## 第五章 面向对象（上）
### 5.1 类和对象
#### 5.1.1 定义类
&emsp;&emsp;java语言里定义类的简单语法如下：
```java
[修饰符] class 类名
{
    零到多个构造器定义
    零到多个成员变量
    零到多个方法
}
```
&emsp;&emsp;类的修饰符可以是`public`、`finale`、`abstract`，或者完全省略。对于一个类定义而言，可以包含三个最常见的成员：构造器、成员变量和方法。  
&emsp;&emsp;**类里的各成员之间可以相互调用，但是static修饰的成员不能访问没有static修饰的成员。**
1. 定义成员变量
```java
[修饰符] 类型 成员变量名 [= 默认值];
```
&emsp;&emsp;修饰符：修饰符可以省略，也可以是`public`、`protected`、`private`、`static`、`final`，前三个只能出现一个，可以和后面两个组合起来修饰成员变量。
2. 定义方法
```java
[修饰符] 方法返回值的类型 方法名(形参列表)
{
    //由零到多个可执行语句组成的方法体
}
```
&emsp;&emsp;修饰符：修饰符可以省略，也可以是`public`、`protected`、`private`、`static`、`final`、`abstract`，其中前三个只能出现一个，后两个只能出现一个，他们可以与static组合起来修饰方法。
3. 定义构造器
```java
[修饰符] 构造器名(形参列表)
{
    //由零到多条可执行语句组成的构造器执行体
}
```
&emsp;&emsp;修饰符：修饰符可以省略，也可以是`public`、`protected`、`private`其中之一。
&emsp;&emsp;构造器名：构造器名必须和类名相同。
#### 5.1.2 对象的产生和使用
```java
Person p;
p = new Person();
//或
var p = new Person();
```
&emsp;&emsp;访问变量或方法：
```java
类.类变量|方法
实例.实例变量|方法
```
&emsp;&emsp;static修饰的方法和成员变量，既可以通过类来调用，也可以通过实例来调用；没有使用static修饰的方法和变量，只能通过实例来调用。
#### 5.1.3 对象的this引用
&emsp;&emsp;关键字总是指向调用该方法的对象，对于static修饰的方法而言，可以使用类来直接调用该方法，如果在static修饰的方法中使用this关键字，则这个关键字就无法指向合适的对象。所以，static修饰的方法中不能使用this引用。


### 5.2 方法详解
#### 5.2.2 方法的参数传递机制
&emsp;&emsp;java里方法的参数传递方式只有一种：值传递。  
&emsp;&emsp;但是，基本类型传入的是值的复制品，引用类型传入的是是一个引用（指针，即地址值）。所以，传入基本类型不会改变原来的值，传入引用类型会改变原来的值（自己的理解，不知道对不对）。
#### 5.2.3 形参个数可变的方法
&emsp;&emsp;看书。
#### 5.2.4 方法重载
&emsp;&emsp;java允许同一个类里定义多个同名的方法，只要形参列表不同就行。如果同一个类里包含了两个或两个以上方法的方法名相同，但形参列表不同，则被称为`方法重载`。
### 5.4 隐藏和封装
#### 5.4.2 使用访问控制符
&emsp;&emsp;看书，没理解透，不乱写。

#### 5.4.3 package, import, import static
&emsp;&emsp;这个看懂了，但是需要更多的实践验证我的想法。
### 5.5 深入构造器
#### 5.5.1 使用构造器执行初始化
&emsp;&emsp;构造器名称必须和类名一样。  
&emsp;&emsp;因为构造器主要用于被其他方法访问，用以返回该类的实例，因而通常把构造器设置成public访问权限，从而允许系统中任何位置的类来创建该类的对象。除非在一些极端的情况下，业务需要限制创建该类的对象，可以把构造器设置成其他访问权限，例如设置为protected，主要用于被器子类调用。
#### 5.5.2 构造器重载
&emsp;&emsp;同一个类里具有多个构造器，多个构造器的形参列表不同，即被称为`构造器重载`。  
&emsp;&emsp;如果系统中包含了多个构造器，其中一个构造器的执行体里完全包含了另一个构造器的执行体，可以使用this关键字来调用相应的构造器。如下所示：
```java
/* 
目录结构
./
    DogTest.java
    ./Liusuntao
        Dog.java
 */
//Dog.java
package Liusuntao;

public class Dog {
    private String name;
    private String color;
    private double weight;
    public Dog(){}
    public Dog(String name, String color)
    {
        this.name = name;
        this.color = color;
    }
    public Dog(String name, String color, double weight)
    {
        this(name, color);
        this.weight = weight;
    }
    public void information()
    {
        System.out.println("name: " + this.name);
        System.out.println("color: " + this.color);
        System.out.println("weight: " + this.weight);
        System.out.println();
    }
}
//DogTest.java
public class DogTest
{
    public static void main(String[] args)
    {
        var dog1 = new Liusuntao.Dog();
        var dog2 = new Liusuntao.Dog("孙悟空", "yellow");
        var dog3 = new Liusuntao.Dog("猪八戒", "black", 300);
        dog1.information();
        dog2.information();
        dog3.information();
    }
}
```
&emsp;&emsp;在这里第三个构造器通过this来调用另一个重载构造器的初始化代码。**使用this调用另一个重载的构造器只能在构造器中使用，而且必须作为构造器执行体的第一条语句**
### 5.6 类的继承
&emsp;&emsp;Java的继承具有单继承的特点，每个子类都只有一个直接父类。
#### 5.6.1 继承的特点
&emsp;&emsp;语法格式如下：
```java
修饰符 class SubClass extends SuperClass
{
    //类定义部分
}
```
&emsp;&emsp;Java的子类不能获得父类的构造器。
#### 5.6.2 重写父类方法
&emsp;&emsp;子类包含与父类同名方法的现象被称为`方法重写`。
&emsp;&emsp;方法重写的原则：
1. 方法名相同，形参列表相同；
2. 子类方法返回值类型应该比父类方法返回值类型更小或相等，子类方法声明抛出的异常类应比父类方法声明抛出的异常类更小或相等；
3. 子类方法的访问权限应比父类方法的访问权限更大或相等。
4. 覆盖方法和被覆盖方法要么都是类方法，要么都是实例方法，不能一个是类方法，一个是实例方法。

&emsp;&emsp;如果父类方法具有private访问权限，则该方法对其子类是隐藏的，因此其子类无妨访问该方法也就无法重写。如果子类定义了一个与父类private方法同名同参的方法，也只是定义了一个新的方法，并不是重写。  
&emsp;&emsp;当子类覆盖了父类的方法以后，子类的对象无法访问父类中被覆盖的方法。但是可以在子类方法中访问父类被覆盖的方法：使用super（被覆盖的是实例方法）或父类名（被覆盖的是类方法）作为调用者来调用父类中被覆盖的方法。
#### 5.6.3 super限定
&emsp;&emsp;super是一个关键字，用于限定该对象调用它从父类继承得到的实例变量或方法，正如this不能出现在static修饰的方法中一样，super也不能出现在static修饰的方法中。static修饰的方法是属于类的，该方法的调用者可能是一个类，而不是对象，因此super的限定就失去了意义。
#### 5.6.4 调用父类构造器
&emsp;&emsp;子类不会获得父类的构造器，但是子类构造器里可以调用父类构造器的初始化代码。  
&emsp;&emsp;在子类构造器中调用父类构造器使用super来完成，super调用父类构造器必须出现在子类构造器执行体的第一行，因此this和super调用不会同时出现。  
## 5.7 多态











