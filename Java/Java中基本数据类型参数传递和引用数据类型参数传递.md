## Java中基本数据类型参数传递和引用数据类型参数传递

- #### 基本数据类型参数传递

  ```java
  // 基本数据类型参数传递demo
  public class Demo1 {

  	public static void main(String[] args){
  	
  		int x=3;
  		show(x);
  		System.out.println("x="+x);
  	}
  	
  	public static void show(int x){
  		x=4;
  	}
  	
  }

  // 结果
  x=3
  ```

  ![./img/java基本数据类型参数传递.png)

  > 解析：
  >
  > 1. main函数进入栈内存；
  > 2. x为局部变量，所以在栈内存中main函数初始化x=3;
  > 3. show函数进入栈内存；
  > 4. x为局部变量，所以在栈内存中show函数初始化x=4;
  > 5. show函数执行完毕，show函数将出栈，此时show函数中局部变量x=4也会跟着出栈；
  > 6. 此时栈内存中留下main函数，以及man函数初始化的局部变量x=3，所以此时打印出来的x为3。

- #### 引用数据类型参数传递

  ```java
  // 引用数据类型参数传递demo
  public class Demo2 {
  	
  	int x=3;
  	public static void main(String[] args){
  		Demo2 d = new Demo2();
  		d.x = 9;
  		show(d);
  		System.out.println("x="+d.x);
  	}
  	
  	public static void show(Demo2 d){
  		d.x = 4;
  	}
  }

  // 结果
  x=4
  ```

  ![./img/java引用数据类型参数传递.png)

  > 解析：
  >
  > 1. 成员变量(属性)x=3，进入堆内存；
  > 2. main函数进入栈内存，存在一个局部变量d;
  > 3. d指向堆内存中的Demo2对象;
  > 4. 堆内存中Demo2存在属性d.x=9,此时将堆内存中原有的x成员变量替换为9；
  > 5. show函数进入栈内存；
  > 6. main函数将d局部变量指向show函数的d局部变量；
  > 7. 此时show函数的d局部变量和main函数的d局部变量一样，指向堆内存的Demo2对象；
  > 8. show函数的d.x=4，将堆内存的Demo2属性x=9替换为x=4;
  > 9. 此时show函数执行完毕，show函数以及d局部变量出栈；
  > 10. 此时堆内存中的x=4，所以打印出此时的x值为4。