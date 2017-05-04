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
## 2017-04-29

- 类  
  一些事物共有的行为和属性。  
  类是由属性和方法构成的。

- 类的一般形式  
  比喻：中国有13亿人，就有13亿对象。

  ```java
  class 类名{

  		类型  变量名;

  		类型  方法名(参数){

  			方法内容

  	}

  }
  ```

- 修饰符  
  可以修饰类、属性、方法  
  private:  只有在本类中可以看见。  
  protected:  在本类或者同一个包里面可见。  
  public:  对于所有的类都可见。  
  默认(无修饰符)：在本类或者同一个包里面可见，相似于protected，但两者还是有区别，区别在于继承方面，后续会讲解。

## 2017-04-30

- 方法  

  ```java
  修饰符 类型 方法名 （参数类型 参数1，参数类型 参数2 ......）{
    方法体
  }
  ```

  如果方法中没有返回值，必须写void(不能省略)  
  如果方法有返回值，则定义的方法的类型必须与方法体内返回值的类型相同。

- 主方法  
  它是程序的入口。  

  ```java
  public static void main(String[] args){

  }
  ```

- 构造方法  
  作用是用于初始化参数。  
  所有的数字变量全部设置为0；  
  所有的boolean类型全部设置为false；  
  所有的对象变量全部设置为NULL。  

  设计自己的构造方法：  

  1. 方法名称必须和类的名称完全相同（首字母必须大写哦）；
  2. 没有返回值。
  3. 如果自己写了有参的构造方法，那么编译器不会在提供默认的构造方法了；如果仍然想使用有参的构造方法，那么需要手动添加参数。

- 方法的重载(构造方法的重载类似)  
  在一个类中可以有多个方法共享同一个名称，只要他们的参数不同就可以。  
  如何调用方法：  

  > 根据参数类型和参数数量来决定调用的是哪个方法。

  重载： 具有相同的方法名称，不同的参数列表。  
  不同的参数列表：参数类型不同，参数数量不同，参数次序不同。

  ```java
  // 对于如下的方法，讲解下方法的重载
  public static int sum(int a,int b){
    
  }
  // 方法是否重载取决于判定的依据是sum(int a,int b)这一块内容，至于public static int这三种类型，方法重载的时候是不会关心的。
  ```


- 对象的创建和使用  
  创建和使用对象之前，需要有个类，因为对象是类的实例。  
  比喻：人类是一个类，每个人是类中的一个实例。
- 对象类型的参数传递  
  Java中只有值传递，基本类型直接传值，而引用类型传递的是引用，而这个引用就是值。  
  基本数据类型作为参数，则在栈内存中直接操作；  
  引用类型作为参数，则操作的是栈内存中的引用指向的是堆内存中存储的对象的值。  
  *此处如果还不能理解，可以看视频教程的[第28节](https://ke.qq.com/webcourse/index.html#course_id=155221&term_id=100177657&taid=778961038761557&vid=c1412r0f4u9)*

## 2017-05-01

- static关键字  

  1. 静态变量  
     静态变量是属于类的，和对象没有关系。    

     非静态变量是属于某个对象的，每个对象都有该数据库的副本，而静态变量只有一个。  

     ```java
     // 测试样例
     public class TestStatic {

     	int commonint=0;//普通变量，非静态的变量
     	static int staticint=0;//静态变量
     	
     	TestStatic(int x){
     		this.commonint=x;
     	}
     	
     	public static void main(String[] args){
     		TestStatic t1=new TestStatic(1);
     		TestStatic t2=new TestStatic(2);
     		
     		System.out.println("t1的非静态变量(普通变量):"+t1.commonint);
     		System.out.println("t2的非静态变量(普通变量):"+t2.commonint);
     		
     		t1.staticint=1;
     		System.out.println("t1为staticint的赋值为1");
     		System.out.println("t1的静态变量"+t1.staticint);
     		System.out.println("t2的静态变量"+t2.staticint);
           
           	t2.staticint=15;
     		System.out.println("t2为staticint的赋值为15");
     		System.out.println("t1的静态变量"+t1.staticint);
     		System.out.println("t2的静态变量"+t2.staticint);
     	}
     }

     // 结果
     t1的非静态变量(普通变量):1
     t2的非静态变量(普通变量):2
     t1为staticint的赋值为1
     t1的静态变量1
     t2的静态变量1
     t2为staticint的赋值为15
     t1的静态变量15
     t2的静态变量15
       
     // 结论
     // 由此可见普通变量和静态变量的区别：
     // 非静态变量是属于某个对象的，每个对象都有该数据库的副本，而静态变量只有一个，更改任意一个对象的static静态常量，其他对象的static静态常量也会随之一起更改。
     ```
     访问静态变量**规范的访问方式**是通过(类名.变量名)进行访问的。

  2. 静态方法

     用static修饰的方法称为静态方法。  
     访问静态方法是通过(类名.方法名)    

     注意：    

     > 1. 静态方法不能访问非静态变量  
     > 2. 非静态方法可以访问静态变量  
     >
     > 静态属性或者方法是在类加载的时候产生的，而非静态的属性或方法是在new(新建对象)的时候产生的。    
     >
     > 由于静态方法存在的时候，可能还没有new对象，此时不存在非静态方法和非静态变量；    
     >
     > 非静态方法存在的时候，说明静态方法早已经存在于内存中。  
     > 所以上述的1、2结论是由方法的生命周期所决定的。    

  3. 静态常量

     ```java
     public static final int x=123;
     ```

- final关键字  
  使用final修饰过的变量，它们的值都是不可改变的，不可被重新赋值。    

  1. final修饰变量  
     恒定不变的属性，一般使用final修饰；  
     变量名建议全部使用大写；  
     final修饰的变量不能改变，如果在程序中重新赋值，则会编译报错。
  2. final修饰方法  
     任何继承类无法覆盖该方法，但是重载不会受到限制。
  3. final修饰类  
     该类不能作为任何类的父类，不能被任何类继承；  
     类中的方法会全部被自动定义为final类型。

## 2017-05-02

- 包的介绍  
  未命名包(杜绝使用) ：默认命名为"default package"  
  命名包

- 封装  

  > 1. 封装概述
  > 2. 实现封装

- 继承  

  > 1. 继承概述  
  >    父类=超类=基类  
  >    子类=派生类  
  >
  >    extends只能继承一个类，Java不支持多重继承。  
  >    子类继承父类之后，子类可以调用父类的属性和方法，子类也可以重写父类的属性和方法（除了final类型的方法），还可以增加自己的属性和方法。
  >
  > 2. 实现继承
  >    super();  
  >    调用子类的构造器时，如果没有显示的写super()，则编译器会默认加上super()无参构造器；  
  >    如果想调用父类的有参构造器，那么就需要显示的调用，编译器不会默认的加上super()。
  >
  > 3. 继承关系  
  >    Java只支持单继承关系。

- 多态  

  > 1. 多态概述  
  >    所谓多态，实际上是一个对象的多种状态。
  > 2. 实现多态


## 2017-05-03

- 抽象类  

  > 1. 定义  
  >    抽象类是为子类提供的一个规范。  
  >    修饰符 abstract 类名{
  >
  >    ​	// 类体
  >    ​	修饰符 abstract 返回值类型 方法名(参数列表);
  >    }  
  >    普通方法是有方法体的 public void test(){};  
  >    抽象方法是一种规范，它是没有方法体的，public abstract void test();  
  >    抽象类里面至少有一个抽象方法，并且也可以在抽象类中定义普通方法。
  >
  > 2. 使用  
  >    @Override  用于检测是否重写成功。  
  >    一个类继承了抽象类，就必须重写这个抽象类中所有的抽象方法；  
  >    如果有一个类没有重写抽象类的抽象方法，那么该类也需要定义为抽象类。

- 接口  

  >1. 概述  
  >   abstract  class抽象类的修饰符  
  >   interface 接口的修饰符  
  >   entends只能是一个；  
  >   implements可以是多个。  
  >   接口只有抽象方法。  
  >   接口是抽象方法和常量的属性集合，有且仅有抽象方法和常量的属性。
  >
  >2. 注意事项  
  >
  >   - 接口的修饰符：默认和public。
  >
  >   - 接口内的变量会被设置成公有的、静态的、常量的字段
  >
  >     ```java
  >     int i=1;
  >     // 接口中会被定义为
  >     public static final int i=1;
  >     ```
  >
  >   - 接口内只有抽象方法和常量的属性集合。
  >
  >3. 实现  
  >
  >   ```java
  >   class 类名 implements 接口1,接口2,接口3{
  >     方法1{
  >       
  >     }
  >     方法2{
  >       
  >     }
  >   }
  >   ```
  >
  >   接口实现的注意事项：  
  >
  >   - 为接口中所有的方法提供具体的实现；
  >
  >   - 必须遵守重写的所有规则；  
  >
  >     重写规则：子类的重写方法不能抛出更大的异常；子类的重写方法不能有更小的访问范围。  
  >     ​		例如：父类：public void test();  
  >       			   子类：public void test();  这是正确的重写规则  
  >     ​				       protect void test();  这是错误的重写规则
  >
  >   - 保持相同的返回类型。
  >
  >4. 接口多继承(即继承多个接口)  
  >
  >   - 这些接口之间需要逗号分隔开；
  >   - 如果多个借口都有相同的方法和相同的变量，那么相同的变量可以通过 接口名.变量名 的形式来访问；
  >   - 相同的方法将被其中的一个接口使用，具体被哪个接口使用不需要care。
  >
  >5. 应用  
  >
  >   - 方法的修饰符  
  >     接口中变量的修饰符：public static final  
  >     方法的修饰符：public abstract
  >   - 接口类型引用变量
  >

- 内部类  
  一个类被嵌套定义在另一个类中，该类被称为内部类。内部类相当于外部类的成员或者变量。  
  构造内部类  

  ```java
  Outer out=new Outer();
  Outer.Inner in=Outer.new Inner();
  ```

  ​

