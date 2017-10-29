## Java中方法覆盖(Override)的原则

参考：[腾讯课堂](https://ke.qq.com/webcourse/index.html#course_id=147646&term_id=100167776&taid=747380144226494&vid=i1411hvv8i8)

原则简称：

**一同两小一大。**

1. 实例的方法签名必须相同。（方法签名=方法名+方法参数列表）

2. 子类方法的返回值类型是和父类方法的返回值类型相同或者是其子类。
   即子类可以返回一个更加具体的类，例如：

   ```java
   //父类
   public class Bird extends Object{
       public Object fly(){
           System.out.println("Flying");
         	return null
       }
   }

   //子类
   public class Penguin extends Bird{
       public Object fly(){
           System.out.println("Flying");
         	return "I am penguin!"
       }
   }
   // 此时子类返回的"I am penguin!"是一个字符串，是Object的一个子类。
   ```

   ​

3. 子类方法声明抛出的异常类型和父类方法声明抛出的异常类型相同或者是其子类。

   - 子类方法声明抛出的异常类型小于或等于方法声明抛出的异常类型；
   - 子类方法可以同时声明多个属于父类方法声明抛出异常类的子类（RunTimeException类型除外）。

4. 子类方法的访问权限比父类方法访问权限更大或者相等(注意private代表私有的，是不能继承的，所以不能被子类所继承，也就不存在Override概念)。例如

   ```java
   //父类
   public class Bird extends Object{
       protected Object fly(){
           System.out.println("Flying");
         	return null
       }
   }

   //子类
   public class Penguin extends Bird{
       public Object fly(){
           System.out.println("Flying");
         	return "I am penguin!"
       }
   }
   // 此时父类fly()方法的权限是protected,子类fly()方法的权限是public,此时的访问权限在override方法中是合理的,因为子类的访问权限比父类的大。
   ```

只有方法具有覆盖的概念，字段是不存在的。那么方法覆盖解决的问题是什么呢？请看下面。

**Override和Overload的区别**

Override=方法覆盖,Overload=方法重载

Overload解决的问题：解决同一个类中，相同功能的方法名不同的问题。

规则为：两同一不同（同类中，方法名不同，方法参数列表不同[参数类型,参数个数,参数顺序不同]）

Override解决的问题：解决当父类中某一行为不符合子类的具体特征的时候，此时子类需要重新定义父类的方法。例如上述示例代码中，Bird都是会飞翔的，但是企鹅作为Bird中的一类，它并不会飞翔(因为它太胖啦=￣ω￣=)，所以此时Bird类中的fly()就需要重新覆盖了。

规则为：一同两小一大。