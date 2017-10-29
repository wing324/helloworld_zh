## Java之多态方法中的调用问题

参考：[腾讯课堂](https://ke.qq.com/webcourse/index.html#course_id=147646&term_id=100167776&taid=747414503964862&vid=n1411d4d7v9)

多态方法调用情况存在四类，以doWork()方法为例：

1. doWork()存在子类中，不存在父类中，结果调用子类的doWork()；

   ```java
   package com.wing.multidemo;

   public class MultiDemo2 {
       public static void main(String[] args){
           SuperClass s1 = new SubClass();
           s1.doWork();
       }
   }

   // 父类
   class SuperClass{
   }

   // 子类
   class SubClass extends SuperClass{
       public void doWork(){
           System.out.println("SubClass doWork");
       }
   }

   //此时输出结果
   编译报错。
   因为编译时期需要去编译类型(Superclass)中找是否有doWork()方法，找到则编译通过，找不到则编译失败。
   ```

2. dowork()不存在子类中，存在父类中，结果调用父类的doWork()；

   ```java
   package com.wing.multidemo;

   public class MultiDemo2 {
       public static void main(String[] args){
           SuperClass s1 = new SubClass();
           s1.doWork();
       }
   }

   // 父类
   class SuperClass{
       public void doWork(){
           System.out.println("SuperClass doWork");
       }
   }

   // 子类
   class SubClass extends SuperClass{
   }

   // 此时输出结果
   SuperClass doWork
   ```

3. doWork()存在子类中，存在父类中，结果调用子类的doWork()；[就近原则]

   ```java
   package com.wing.multidemo;

   public class MultiDemo2 {
       public static void main(String[] args){
           SuperClass s1 = new SubClass();
           s1.doWork();
       }
   }

   // 父类
   class SuperClass{
       public void doWork(){
           System.out.println("SuperClass doWork");
       }
   }

   // 子类
   class SubClass extends SuperClass{
       public void doWork(){
           System.out.println("SubClass doWork");
       }
   }

   // 此时输出结果
   SubClass doWork
   此时是先从SubClass中找是否存在doWork()方法，再去SuperClass中找是否存在doWork()方法。
   ```

4. doWork()存在子类中(static方法)，存在父类中(方法)，结果调用子类的doWork()。[就近原则]
   注意，这种方式称之为"隐藏"，而不是"方法覆盖"。

   ```java
   package com.wing.multidemo;

   public class MultiDemo2 {
       public static void main(String[] args){
           SuperClass s1 = new SubClass();
           s1.doWork();
       }
   }

   // 父类
   class SuperClass{
       public static void doWork(){
           System.out.println("SuperClass doWork");
       }
   }

   // 子类
   class SubClass extends SuperClass{
       public static void doWork(){
           System.out.println("SubClass doWork");
       }
   }

   // 此时输出结果
   SuperClass doWork
   解释：静态方法的调用，只需要类即可。如果使用对象来调用方法，其实使用对象的类(该代码中为SuperClass)调用静态方法。
   ```

**总结:**

1. 多态调用方法时，首先方法会去子类中查找方法是否存在，再去父类中查找方法是否存在[方法必须存在父类中，否则多态编译将会失败。]
2. 多态中静态方法的"重写"不叫重写，应该叫"方法隐藏"，为什么呢？因为静态方法的调用，只需要类即可，如果使用对象来调用静态方法，其实使用的是对象的类来调用静态方法的。