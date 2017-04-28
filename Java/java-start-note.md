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


## 2017-04-22

- 转义字符  
  每个转义字符由两个字符组成，但是编译器会将其当作一个字符，如"\n"为1个字符。

- System.out.print()与System.out.println()的区别  
  System.out.print()该行输出后不换行，下一个输出接着本行后输出；  
  System.out.println()该行输出后换行。

  ```java
  # System.out.print()
  System.out.print("Hello World!");
  System.out.print("Hello World!");
  // 输出的结果：Hello World!Hello World!

  # System.out.println()
  System.out.println("Hello World!");
  System.out.println("Hello World!");
  /** 
  输出的结果为：
  Hello World!
  Hello World!
  */
  ```


## 2017-04-23

- 字符串处理  
  1. 求子串  
     str.substring(startIndex,endIndex)
  2. 测试字符串是否相等(equals)  
     Object equals(==): 比较内存地址  
     String equals: 比较内容相同即可，不管内存地址  
     String equals为True时，Object equals不一定为True;  
     Object equals为True时，String equals一定为True。
  3. 字符串编辑
- 存储空间  
  寄存器、栈、堆、静态存储区、常量存储区(常量池)、其他存储位置；
  其中，堆用来存储new的对象，常量池用来存储final static/String常量池。


## 2017-04-24

- 基本算术运算符  

  \+    加法运算  
  \-     减法运算  
  \*     乘法运算  
  /      除法运算    

- 模运算符  
  %    取模运算  
  %=   取模赋值  

- 算术赋值运算符

  \+=    加法赋值  
  \-=     减法赋值  
  \*=    乘法赋值  
  /=     除法赋值

- 自增自减运算符

  \++   自加运算  
  \--     自减运算  

## 2014-04-25

- 关系运算符
  返回值是boolean,一般用于判断语句中  
  == 、!=、>、<、>=、<=

- 逻辑运算符
  &：逻辑与  
  |：逻辑或  
  !：逻辑非  
  &&：短路与  
  ||：短路或  

  ```java
  boolean a;
  b = condition1&condition2; //先求得1、2的值，再进行判断
  b = condition1&&condition2;//先对1进行判断，如果1为真，再判断2，如果1为假，则不再对2进行判断
  // 由此可见短路算法效率更高
  // 依次类推 逻辑或 与 短路或

  //实战示例
  package day14;

  public class TestRelation2 {
  	
  	public static boolean returntrue(){
  		System.out.println("Return True");
  		return true;
  	}
  	
  	public static boolean returnfalse(){
  		System.out.println("Return False");
  		return false;
  	}
  	
  	public static void main(String[] args){
  		boolean b1;
  		System.out.println("逻辑与运算returntrue()&returnfalse()");
  		b1 = returntrue()&returnfalse();
  		System.out.println(b1);
  		
  		boolean b2;
  		System.out.println("短路与运算returntrue()&&returnfalse()");
  		b2 = returntrue()&&returnfalse();
  		System.out.println(b2);
  		
  		boolean b3;
  		System.out.println("逻辑与运算returnfalse()&returntrue()");
  		b3 = returnfalse()&returntrue();
  		System.out.println(b3);
  		
  		boolean b4;
  		System.out.println("短路与运算returnfalse()&&returntrue()");
  		b4 = returnfalse()&&returntrue();
  		System.out.println(b4);
  	}
  }

  // 实战结果
  逻辑与运算returntrue()&returnfalse()
  Return True
  Return False
  false
  短路与运算returntrue()&&returnfalse()
  Return True
  Return False
  false
  逻辑与运算returnfalse()&returntrue()
  Return False
  Return True
  false
  短路与运算returnfalse()&&returntrue()
  Return False
  false
  ```

- 三元运算符

  ```java
  public class TestRelation3 {

  	public static void main(String[] args){
  		int i,k;
  		i=-5;
  		k=i>0?i:-i;//i>0输出i,i<=0 输出-i
  		System.out.print(k);
  	}
  }
  //i=-5输出--5，即5
  //i=5输出5，即5
  ```

- 运算符优先级


## 2016-04-26

- 选择结构

  1. if语句  

     ```java
     //非嵌套if语句
     if(条件){
       //语句块1
     }else{
       //语句块2
     }

     //嵌套if语句
     if(条件1){
       //语句块1
       if(条件2){
       	//语句块3
     	}else{
       	//语句块4
     	}
     }else{
       //语句块2
     }

     // 嵌套if语句
     if(条件1){
       //语句块1
     }else if(条件2){
       	//语句块2
     	}else if(条件3){
           //语句块3
     	}else{
           //语句块4 
     	}
     ```

  2. switch语句  

     ```java
     switch(表达式){
       case:value1://程序语句1
         		break;
       case:value2://程序语句2
         		break;
       case:value3://程序语句3
         		break;
       case:value4://程序语句4
         		break;
       default://默认程序语句
     }
     ```

     表达式类型：byte short int char String(java7+)  
     value值会表达式类型一致，且不能出现重复的value值。

- 循环结构  

  1. while循环  

     ```java
     // while循环
     while (条件){
       //循环语句
     }

     //do-while循环
     do{
       //循环语句
     }while(条件)
     // do-while首先执行一次循环体，在去判断while条件确定是否再次执行循环体，所以do-while保证了至少执行一次循环体。
     ```

  2. for循环  

     ```java
     for(初始化;条件;迭代运算){
       //循环语句
     }

     //条件必须为boolean表达式
     ```


## 2017-04-27

- break  
  强制当前循环终止。

- continue  
  停止本次循环，继续执行剩下的循环。

  ```java
  //没有continue代码
  public class TestContinue {

  	public static void main(String[] args){
  		for(int i=1;i<10;i++){
  			System.out.print(i);
  		}
  	}
  }
  // 结果为：123456789

  //有continue代码
  public class TestContinue {

  	public static void main(String[] args){
  		for(int i=1;i<10;i++){
  			if(i==6){
  				continue;
  			}
  			System.out.print(i);
  		}
  	}
  }
  // 结果为：12345789
  // 解析：此处当i=6的时候，跳出本次循环，继续执行剩下的循环，那么6将不被打印出来，从7开始继续执行循环
  ```

- return  
  从当前的方法中退出，即跳出整个方法，该方法内所有的代码都将不会执行。

- 数组  
  一组数的集合，且集合中的数据类型必须相同。

  - 创建数组  
    ArrayType arrayname[]=new ArrayType[length];  
    ArrayType[] arrayname=new ArrayType[length];//常用写法

- 初始化数组  

  > 1. 第一种初始化方式
  >
  >    ```java
  >    double[] array1=new double[5];
  >    array1[0]=0;
  >    array1[1]=1;
  >    array1[2]=3;
  >    array1[3]=-16;
  >    array1[4]=0.918;
  >    ```
  >
  > 2. 第二中初始化方式
  >
  >    ```java
  >    double[] array1={0,1,3,-16,0.918};
  >    ```
  >


## 2017-04-28

- Scanner类  
  交互式输入

  ```java
  // 交互式输入提示
  // 创建一个Scanner类的对象，用它获取用户的输入
  Scanner sc=new Scanner(System.in);
  System.out.println("请输入前10名同学的成绩:");

  // 交互式输入获取
  for(student=0;student<temp.length;student++){
  	//读取用户的输入
  	temp[student]=sc.nextDouble();
  	sum+=temp[student];
  ```

- 获取数组长度  

  ```java
  int[] array1=new int[10];
  System.out.println(array1.length);
  ```

- 数组的复制  
  array1=array2;  
  复制之后，两个引用指向同一个数组，不管是哪个引用修改了数组元素的值，对另一个引用来说，元素的值也是修改过的。  

  ```java
  public class TestCopyArray {

  	public static void main(String[] args){
  		int[] array1={1,2,3};
  		int[] array2={4,5,6};
  		
  		System.out.println("两个数组的初始化值为：");
  		for(int i=0;i<array1.length;i++){
  			System.out.println("array1["+i+"]="+array1[i]);
  		}
  		
  		for(int i=0;i<array2.length;i++){
  			System.out.println("array2["+i+"]="+array2[i]);
  		}
  		
  		//把array2赋值给array1
  		array1=array2;
  		System.out.println("数组赋值之后的值为：");
  		for(int i=0;i<array1.length;i++){
  			System.out.println("array1["+i+"]="+array1[i]);
  		}
  		
  		for(int i=0;i<array2.length;i++){
  			System.out.println("array2["+i+"]="+array2[i]);
  		}
  		
  		array1[0]=100;
  		array2[1]=300;
  		System.out.println("array1重新赋值之后的值为：");
  		for(int i=0;i<array1.length;i++){
  			System.out.println("array1["+i+"]="+array1[i]);
  		}
  		
  		for(int i=0;i<array2.length;i++){
  			System.out.println("array2["+i+"]="+array2[i]);
  		}
  	}
  }

  // 输出结果为：
  两个数组的初始化值为：
  array1[0]=1
  array1[1]=2
  array1[2]=3
  array2[0]=4
  array2[1]=5
  array2[2]=6
  数组赋值之后的值为：
  array1[0]=4
  array1[1]=5
  array1[2]=6
  array2[0]=4
  array2[1]=5
  array2[2]=6
  array1重新赋值之后的值为：
  array1[0]=100
  array1[1]=300
  array1[2]=6
  array2[0]=100
  array2[1]=300
  array2[2]=6
  ```

- 多维数组  
  Java中只存在一维数组，多维数组只是数组中的数组