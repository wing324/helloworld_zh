## Java之static关键字

##### 一、特点

1. static是一种修饰符，用于修饰成员（成员变量+成员函数）；
2. 数据共享：static修饰后的成员可以被每个对象所共享；
3. static修饰的成员优于对象之前存在，因为static的成员随着类的加载就已经存在了；
4. static修饰后的成员可以被类名直接调用，而不仅仅被对象调用；
5. static修饰的是共享数据，对象中存储的是特有数据;  
6. static修饰的是共享数据，JVM会对其进行单独空间的存储，而不是每个对象存储一份数据，所以static会节省存储空间和内存。


##### 二、成员变量 PK 静态变量

1. 两个变量的生命周期不同。  
   成员变量随着对象的创建而创建，随着对象的回收而释放；  
   静态变量随着类的加载而加载，随着类的消失而消失（类一般在虚拟机结束了，则消失了）。
2. 调用方式不同。  
   成员变量只能被对象调用；  
   静态变量可以被对象调用，也可以被类调用（推荐这种方式使用静态变量）。
3. 别名不同。  
   成员变量称为实例变量或者实例成员(没有被static修饰过的成员);  
   静态变量称为类变量或者类成员(被 static修饰过的成员);  
   **注意**  
   成员变量和静态变量都属于全局变量。
4. 数据存储位置不同。  
   成员变量数据存储在堆内存的对象中，所以也叫对象的特有数据；  
   静态变量数据存储在共享区（方法区）中，所以也叫对象的共享数据。

##### 三、注意事项

1. 静态方法只能访问静态变量。

   ```java
   // 错误代码
   public class staticDemo {

   	public static void main(String[] args){
   		Person.show();
   	}
   }

   class Person{
   	String name;
   	static String country = "CN";
   	public static void show(){
   		System.out.println(country+":"+name); // 此处编译会报错，因为show是静态方法，但是name是非静态变量
   	}
     
   // 正确代码
   public class staticDemo {

   	public static void main(String[] args){
   		Person.show();
   	}
   }

   class Person{
   	String name;
   	static String country = "CN";
   	public static void show(){
   		System.out.println(country);
   	}
   ```

2. 静态方法中不可以使用this/super关键字。（this是因为没有对象==）

3. 主函数是静态的。

   > 主函数特殊之处：
   >
   > - 格式是固定的`public static void main(String[] args){}`
   >   public： 因为权限必须是最大的；
   >   static： 不需要创建对象，直接用主函数所属类名调用即可，如java staticDemo.java时，直接用java staticDemo.main调用方法即可；
   >
   >   void：主函数没有具体的返回值；
   >   main:   函数名，不是关键字，只是一个被jvm识别的固定名字；
   >   String[] args：主函数的参数列表，是一个数组类型的参数。而且元素都是字符串类型。
   >
   > - 被jvm识别和调用

##### 四、静态代码块

```java
// 代码
public class mainTest {

	public static void main(String[] args){
		new Demo().show();
		new Demo().show();
		new Demo().show();
	}
}

class Demo{
	void show(){
		System.out.println("show run");
	}
  
	// 静态代码块
	static{
		System.out.println("hahahhahah");
	}
}

// 结果
hahahhahah
show run
show run
show run
```

特点：  
随着类的加载而执行，并且只执行一次。  

作用：  

用于给类进行初始化。  

##### 五、什么时候使用static进行修饰字段或者方法

如果这个字段/方法属于整个事物(类)，此时需要使用static修饰，被所有对象所共享。    

[在开发中，往往把工具的方法使用static使用，这样方便我们的调用，而不用每一次调用方法时都去new一个对象出来啦。]