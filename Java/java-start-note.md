# Java入门学习笔记

视频教程:   
[Java轻松入门实用教程【柠檬学院】](https://ke.qq.com/course/155221)

## 2017-04-18

- java的工作方式  
  java源文件(.java文件)  ---->  经过java编译器的编译  ---->  字节码文件(.class文件)  ---->  被类装载器装载到java虚拟机(JVM)  ---->  被JVM解释给操作系统  ---->  操作系统执行

- java虚拟机(JVM)的组成  
  指令集、寄存器、堆栈、垃圾处理器、方法区域  

- 类库  
  标准类库、开发者自己的类库

- 包  
  本质上是文件夹的形式，用于组织项目文件

- JRE  
  Java Runtime Environment(java的运行环境)

- JDK  
  Java Development Kit(Java开发组建)

- 搭建JAVA开发环境(win环境)  

  > 1. 安装jdk，默认配置路径：C:\Program Files\Java\jdk1.7.0_67\
  > 2. 配置jdk环境变量(我的电脑---属性---高级系统设置---环境变量---系统变量)  
  >    JAVA_HOME    C:\Program Files\Java\jdk1.7.0_67\    (需要新建)  
  >    PATH    C:\Program Files\Java\jdk1.7.0_67\bin    (该变量已存在，添加新的路径即可，每个路径以;分割)  
  >    CLASSPATH    .    (.表示当前路径，需要新建)  
  > 3. 验证环境搭建成功  
  >    win+r --- cmd --- 键入java(或者java -version) --- 键入javac




## 2017-04-19

- 详解第一个Java程序(以HelloWorld为例)

  >1. 编写java源程序(Helloworld.java文件)  
  >2. 编译源文件(javac HelloWorld.java 产生HelloWorld.class文件)
  >3. 运行程序(java HelloWorld )

- Java的基本元素

  > 1. 空白分隔符：空格，tab键，换行符
  > 2. 关键字
  > 3. 标识符：类、方法、变量的名字。
  >    1. 注释  
  >       单行注释：//这里的内容是注释  
  >       注释块：/*  这里的内容可以换行，因为这是多行注释 */  
  >       文档注释：/**   这里是文档注释，可以换行，文档注释允许把关于程序的信息嵌入到程序内部，可以使用javadoc工具提取这些信息，形成帮助文档 */
  > 4. 分隔符



## 2017-04-20

- 八大数据类型

  > 1. 整型  
  >    byte    1字节  
  >    short    2字节  
  >    int    4字节  
  >    long  8字节
  > 2. 字符型  
  >    char  2字节
  > 3. 浮点型  
  >    float   4字节    单精度  
  >    double    8字节    双精度
  > 4. 布尔型  
  >    boolean   true(真) flase(假)

- Java的变量和常量  

  1. java的变量  
     变量：值可以变
  2. java的常量  
     常量：值不可以变  
     定义常量使用final关键字

## 2017-04-21

- 数据类型转换  

  1. 自动转换  
     逻辑运算时：  
     两个数中有一个double类型，另一个也会被转换为double类型，结果为double类型;  
     否则，两个数中有一个float类型，另一个也会被转换为float类型，结果为float类型;   
     否则，两个数中有一个long类型，另一个也会被转换为long类型，结果为long类型;  
     否则，两个数都转换为int类型。

  2. 强制类型转换

     ```java
     double adouble=3.55555;
     int aint=(int)adouble;//将double类型转换为int类型
     ```

- 赋值与初始化

  1. 赋值

  2. 初始化  
     局部变量：必须显示的进行初始化;  
     实例变量和类变量：编译器可以自动对它们进行初始化。

     ```java
     //局部变量初始化方式1
     int a=1;

     //局部变量初始化方式2
     int a;
     a=1;
     ```

     ​