## Java中成员变量和局部变量的区别

1. 成员变量定义在类中，整个类都可以访问;  
   局部变量定义在函数、语句、局部代码块中，只在所属的区域有效。

   ```java
   public class Car {

   	int num;//成员变量
   	String color;
   	
   	public void run(){
         	 int numlocal = 5;// 局部变量
   		System.out.println(num+","+color);
   	}	
   }
   ```

2. 成员变量存在与堆内存的对象中;  
   局部变量存在与栈内存的方法中。
   因为成员变量是对象的属性，对象是存储在堆内存中的，而局部变量是存储在栈中的。

3. 成员变量随着对象的创建而存在，随着对象的消失而消失;  
   局部变量随着所属区域的执行而存在，随着所属区域的结束而释放。

   ```java
   public class Car {

   	int num=4;//轮胎数
   	String color;//颜色
   	
   	public void run(){
   		int num=8;
   		System.out.println(num);//此时的num为8
   	}
   	System.out.println(num);// 此时的num为4
   }
   ```

4. 成员变量具有默认初始化值;  
   局部变量没有默认初始化值。