## Java之创建和启动线程

参考[腾讯课程](https://ke.qq.com/webcourse/index.html#course_id=148614&term_id=100169021&taid=767703929472134&vid=o1411ppyby1)学习笔记。

##### 一、两种方式  

1. 继承Thread类;  
2. 实现Runnable接口。

##### 二、方式一解析

步骤：  

> - 定义一个类A继承于java.lang.Thread类;
> - 在A类中覆盖Thread类中的run方法；
> - 在run方法中编写需要执行的动作 —> 需要并发执行的代码块
> - 在main方法中，创建线程对象并启动线程  
>   创建线程类：AClass a = new AClass();  
>   调用对象的start方法：a.start();//启动一个线程
>
> **注意**
> 千万不要调用run方法，如果调用run方法就是在调用对象的方法，此时它还是一个单线程方法。

示例：

```java
public class Demo {
    public static void main(String[] args){
        Game g = new Game();
        Music m = new Music();
        g.start(); //启动Game的多线程并发
        m.start(); //启动Music的多线程并发
    }
}

// 继承方式创建多线程Game
class Game extends Thread{
    @Override
    public void run() {
        for(int i=0;i<100;i++){
            System.out.println("Play Games!  "+i);
        }
    }
}

// 继承方式创建多线程Music
class Music extends Thread {
    @Override
    public void run() {
        for(int i=0;i<100;i++){
            System.out.println("Play Music!  "+i);
        }
    }
}
```

##### 三、方式二解析

步骤：  

>- 定义一个B类实现Runnable接口,注意B类不是线程类哦；
>- 在B类中覆盖Runnable接口的run方法；
>- 在run方法中编写需要执行的操作 —> 需要并发执行的代码块；
>- 在main方法中创建线程对象，并启动线程  
>  创建线程类对象：Thread t = new Thread(new B());  
>  调用线程对象的start方法：t.start();

示例：

```java
public class ClassDemo {
    public static void main(String[] args){
        Thread t1 = new Thread(new GameClass());
        Thread t2 = new Thread(new MusicClass());
        t1.start(); //启动Game的多线程并发
        t2.start(); //启动Music的多线程并发
    }
}

//实现方式创建多线程Game
class GameClass implements Runnable{
    @Override
    public void run() {
        for(int i=0;i<100;i++){
            System.out.println("Play Games!  "+i);
        }
    }
}

//实现方式创建多线程Music
class MusicClass implements Runnable{
    @Override
    public void run() {
        for(int i=0;i<100;i++){
            System.out.println("Play Music!  "+i);
        }
    }
}
```

