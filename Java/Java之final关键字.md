## Java之final关键字

参考：[腾讯课堂 ](https://ke.qq.com/webcourse/index.html#course_id=147646&term_id=100167776&taid=747444568735934&vid=r1411l01p38) 

##### 一、为什么使用final关键字  

继承关系中最大的弊端是破坏了封装：子类可以访问父类的实现细节，而且可以通过方法覆盖的形式修改方法的实现细节。那么final就用来让你不可以做任何的更改，只能调用，不允许修改。  

final可以修饰非抽象类/非抽象方法/变量。  

**注意：**构造方法不能使用final修饰，因为构造方法不能被继承，它是最终的一个状态。  

##### 二、关于final

- final修饰的非抽象类
  final修饰的非抽象类是不能被子类继承的。  

  ```java
  package com.wing.finaldemo;

  public class FinalDemo {
      public static void main(String[] args){
      }
  }

  final class Superclass{
  }

  class Subclass extends Superclass{
  }

  //此时会出现如下的报错
  Cannot inherit from final 'com.wing.finaldemo.Superclass'
  即com.wing.finaldemo.Superclass使用了final修饰符，导致子类无法继承。
  ```

  **哪些类需要使用final来修饰呢？**  

  - 该类不是专门为继承而设计的  
  - 处于安全考虑，类的实现细节不许改动，不准修改源代码  
  - 确保该类不会在被拓展。  

- final修饰的非抽象方法  
  final修饰的非抽象方法被称为最终的类，该方法不能被子类修改。

  ```java
  package com.wing.finaldemo;

  public class FinalDemo {
      public static void main(String[] args){

      }
  }

  class Superclass{
      public final void doWork(){
          System.out.println("SuperClass doWork");
      }

  }

  class Subclass extends Superclass{
      public void doWork(){
          System.out.println("SubClass doWork");
      }
  }

  //此时会出现以下错误
  Error:(17, 17) java: com.wing.finaldemo.Subclass cannot override  doWork() in com.wing.finaldemo.Superclass doWork(),overriden method is final.
  ```

  **哪些方法需要使用final修饰呢？**  

  - 在父类中提供的统一的方法不准子类通过Override来修改，只允许子类调用，不允许子类修改。  
  - 在构造器中调用的方法(初始化方法)，初始化方法一般为final修饰。  

- final修饰的变量  
  final修饰的变量被称为常量，该变量只能赋值一次，不能再次被赋值。  
  **final是唯一可以修改局部变量的修饰符。**  
  final修饰基本类型变量：表示该变量的值不能改变，即不能用"="赋值;  
  final修饰引用类型变量：表示该引用变量的引用地址不能改变，而不是引用地址里面的内容不能变。

  ```java
  // 修改final修饰引用类型变量对应的引用地址的内容
  package com.wing.finaldemo;
  public class FinalDemo {
      public static void main(String[] args){
          final Person p1 = new Person();
          System.out.println(p1.info);
          p1.info = "Second Vlaue";
          System.out.println(p1.info);
      }
  }

  class Person{
      public String info="First value";
  }

  //此时输出
  First value
  Second Vlaue
  // 可见：final修饰引用变量时，其引用地址里面的内容可以改变。

  //修改final修饰引用类型变量对应的引用地址改变
  public class FinalDemo {
      public static void main(String[] args){
          final Person p1 = new Person();
          System.out.println(p1.info);
          p1.info = "Second Vlaue";
          System.out.println(p1.info);
          p1 = new Person();
      }
  }

  class Person{
      public String info="First value";
  }

  // 此时编译报错
  Cannot assign a value to final variable 'p1'
  ```

  **哪些变量需要使用final修饰呢？**  
  当在程序中多个地方使用一个不变的变量，就将其定义为常量。