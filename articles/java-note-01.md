## 目录

<!-- toc -->

-----
## JDK,JRE,JVM说明
+ **JVM**
JAVA虚拟机，JAVA的代码执行引擎，执行编译后的.class文件，是一个规范，类似于CLR。
+ **JRE**
JAVA运行环境，面向JAVA程序用户，可以让系统运行JAVA应用程序，类似于.Net Framework。（JRE=JVM+JAVA标准核心类库）
framework 
+ **JDK**
JAVA 开发套件，面向开发者，在JRE和JVM之外还包含了一些工具类库和调试工具，类似于.Net Framework SDK

JDK>JRE>JVM，他们是包含关系,如图：

![jvm](http://7tsy9a.com1.z0.glb.clouddn.com/mewujnjava/01/000.png)

在Windows的JDK安装路径中(C:\Program Files\Java\jdk1.8.0_05)，可以看到JDK中包含了JRE，JRE的bin中有一个JVM.dll 用于加载其他的jar包来共同执行JAVA应用程序。

> 参考：[http://www.cnblogs.com/xiaofeixiang/p/4085159.html](http://www.cnblogs.com/xiaofeixiang/p/4085159.html)

## Java开发环境配置

### 1. 下载JDK并安装

下载链接 [http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html](http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html) 

### 2. 配置系统环境变量

+ 新建系统变量JAVA_HOME,值为C:\Program Files\Java\jdk1.8.0_05
+ 查找系统变量Path,增加值为%JAVA_HOME%\bin;
+ 新建系统变量classpath ,值为.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar; (.;一定不能少，因为它代表当前路径) 
*(JDK5.0之后不需要此步骤)*

### 3. 验证
1.在Windows下 按win+R 出现运行窗口，输入 %JAVA_HOME%,看是否进入 C:\Program Files\Java\jdk1.8.0_05 文件夹

![jdk](http://7tsy9a.com1.z0.glb.clouddn.com/mewujnjava/01/003.png)

2.然后打开控制台（Win+R,输入cmd）输入 java -version 如果会出现如下信息，就证明配置成功

![jdk](http://7tsy9a.com1.z0.glb.clouddn.com/mewujnjava/01/002.png)

## Hello World

新建一个文件夹，取名helloworld(最好不要用中文)
进入文件夹，新建一个文本文件，名字为helloworld.java,扩展名为.java，并输入代码

```java
public class HelloWorld {
	public static void main(String[] args){
	    System.out.println("hello world");
	}
}
```
+ 在控制台中进入到之前建的目录 C:\Users\xxx\Desktop\JavaDev\helloworld 输入 javac helloworld.java 然后再输入 java helloworld 程序会输入 hello world

![jdk](http://7tsy9a.com1.z0.glb.clouddn.com/mewujnjava/01/004.png)

几个注意的问题

 + 执行javac编译的时候会生成一个class文件，每个类对应一个class文件。一个java文件中可以有多个类，但是public类只能有一个，public的类名必须与java文件名称一致，大小写敏感。
 + 因为每个类都会生产一个class文件，所以class文件与java文件可以是多对一的关系。
 + javac和java 等命令 都是一些exe文件存在于JAVA_HOME中，功能非常丰富，具体可以在控制台中输入 javac -help 查看
 + 编译UTF-8格式的java文件时会报gbk的编码错误，需要在javac中加入encoding，命令： javac -encoding UTF-8 HelloWorld.java

## JAVA语言基础

### 1. 字符集
JAVA内置使用的是Unicode字符集（变量名等支持多种语言，推荐用英文)，源代码文件字符集推荐使用UTF-8。
> 参考: [http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html](http://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html)

### 2. 数据类型

**原始数据类型**：数值(byte/short/int/long/float/double) 字符(char) 布尔(boolean)

```java
byte b;//1个字节，8个bit(2的8次方256)，范围-128-127
short s;//2个字节，2的16次方(65536)，范围 -32768-32767
int i; //4个字节 2的32次方(4294967296)，范围-2147483648-2147483647
long l;//8个字节 2的64次方
float f;//4字节单精度 
double d;//8字节双精度
```
> 最好不要使用浮点数比较，会因为精度的问题出错。精度问题参考：[http://zh.wikipedia.org/wiki/IEEE_754](http://zh.wikipedia.org/wiki/IEEE_754)

数值常量默认是int类型int i=10;浮点型默认为double类型，如果要声明其他类型的常量，需要在尾部带字母，如long i=100L中的i为long类型，类似的有

```java
int i=10;
long l=100L;
float f=1.9F;
double d=1.0D;
```
数值8进制和16进制以0(零)开头，8进制中最大为8，于是010就是8，10在8进制中表示为012，16进制带字母，儿进制为0b开头。
```java
int oct=010; //8进制.8
int hex=0xf; //16进制.15
int binary=0b010 //2进制.2
```

**引用数据类型**

### 3. 运算符

特别的几个需要注意：
1. instanceof 是否是某个类的对象判断，只能用于引用类型，不能用于基础类型，isAssignableFrom可以用于所有类型
2. 二进制位运算 (^是异或运算，互斥，相同为0，不同为1)
3. 移位运算<< >> (**<<左移1位是num乘2**，**>>右移1位是num除2**) 
4. (>>>)(按位右移不足补0),表示无符号右移, expression1>>>expression2 运算符把 expression1 的各个位向右移 expression2 指定的位数。右移后左边空出的位用零来填充。移出右边的位被丢弃。


### 4. 命名规范

+ package包的命名（全部小写，由域名定义）如 me.wujn.helloworld
+ 类的命名 （单词首字母大写） 如 HelloWorld
+ 方法的命名 （首字母小写，字母开头大写）如 sayHello()
+ 常量的命名 （全部大写 ，常加下划线） 如 final int MAX_AGE=25;
+ 变量命名 (单个单词全小写，多个单词首字母小写，后面单词开头大写) 
+ 实现接口的类命名 (以Impl结尾) 如 HelloWorldImpl:IHelloWorldService
+ 异常以及测试类命名 (以xxxxException和xxxTest结尾) 如 RuntimeException 和 HelloWorldTest
+ 注释规范以javadoc的规范为参考



