## Java中常用的三个字符串类

[腾讯课堂学习笔记](https://ke.qq.com/webcourse/index.html#course_id=147646&term_id=100167776&taid=747655022133438&vid=m1411x3q827)

##### 一、分类  

- 不可变的字符串  
  代表类：String  
  当对象创建完毕之后，该字符串的内容是不可以改变的，一旦内容改变就是一个新的对象。  
- 可变的字符串  
  代表类：StringBuilder/StringBuffer  
  当对象创建完毕之后，该对象的内容可以发生改变，并且对象保持不变。  

##### 二、字符串的本质

其实就是char[]。  

```java
// 底层是等价的
String str = "ABCDE";//创建一个String对象
char[] cs = new char[]{'A','B','C','D','E'};
//在java.lang.String的源代码里面，存在如下的代码
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```

##### 三、String类

- 不可变的字符串。  

```java
String s1 = "Wing";
System.out.println(s1.hashCode()); // 2696235
s1="Domain";
System.out.println(s1.hashCode()); // 2052636900
// 可见两者的hashcode改变，即String类是不可变的字符串
```

- String对象的空值  

  > - 表示引用为null，没有初始化，没有分配内存空间，`String s = null;`
  > - 内容为空字符串，已经初始化，已经分配内存空间，`String s = "";`

- String对象的创建方式  

  > - 方式一：直接赋值`String s1 = "Wing";`  
  > - 方式二：通过构造器创建`String s2 = new String("Wing");`
  >
  > 两种方式的区别：  
  > 方式一：最多创建一个String对象，最少不创建对象，如果常量池中存在"Wing"，那么s1直接引用，此时不创建对象；否则，先在常量池中创建"Wing"常量，然后s1再引用。  
  > 方式二：最多创建两个String对象，至少创建一个String对象。**new**这个关键字绝对会在堆内存中创建对象(这是比不可少的对象)；如果常量池中存在"Wing"，那么s2直接引用，此时不创建对象；否则，先在常量池中创建"Wing"常量，然后s2再引用。  

##### 四、StringBuilder类

- 可变字符串
- 使用StringBuilder无参数的构造器，在底层创建了一个长度为16的char数组：  
  char[] value = new char[16];
  此时该数组只能存储16个字符，如果超过了，就得自动扩容。  
  自动扩容的步骤为：自动扩容一个更大的数组，再把之前的数组拷贝到新的数组中，此时的性能极低，所以一般情况我们事先知道存储多少字符，就应该构造器中先设置，免得出现自动扩容的步骤。如`StringBuilder s = new StringBuilder(80);`

##### 五、StringBuffer类  

StringBuffer类的功能和方法参考StringBuilder类。

##### 六、三类之间的区别

- String类创建的字符串对象是不可变的；  
  StringBuilder/StringBuffer类创建的字符串对象是可变的。

- 三者性能不一样,StringBuilder/StringBuffer远远大于String  

  ```java
  // 分别使用String/StringBuilder/StringBuffer拼接300000次字符串，对比各自之间的损耗
  public class StringRuntimeDemo {
      public static void main(String[] args){
          Long stringstart = System.currentTimeMillis();
          String s1="";
          for(int i=0;i<300000;i++){
              s1 = s1+"A";
          }
          Long stringstop = System.currentTimeMillis();
          Long stringtime = stringstop - stringstart;
          System.out.println("stringtime: "+stringtime+"ms"); //19603ms

          Long stringbuilderstart =System.currentTimeMillis();
          StringBuilder s2 = new StringBuilder("");
          for(int i=0;i<300000;i++){
              s2.append("A");
          }
          Long stringbuilderstop =System.currentTimeMillis();
          Long stringbuildertime = stringbuilderstop - stringbuilderstart;
          System.out.println("stringbuildertime: "+stringbuildertime+"ms"); // 5ms

          Long stringbufferstart =System.currentTimeMillis();
          StringBuffer s3 = new StringBuffer("");
          for(int i=0;i<300000;i++){
              s3.append("A");
          }
          Long stringbufferstop =System.currentTimeMillis();
          Long stringbuffertime = stringbuilderstop - stringbuilderstart;
          System.out.println("stringbuffertime: "+stringbuffertime+"ms"); //5ms
      }
  }
  ```

- StringBuffer和StringBuilder功能和方法都是相同的，唯一的区别在于
  StringBuffer中的方法都使用了synchronized修饰符，表示同步的，在多线程并发的时候可以保证线程安全，但是保证线程安全的时候性能较低。  
  StringBuilder中的方法都没有使用synchronized修饰符，多线程并发的时候不安全，但是性能较高。  