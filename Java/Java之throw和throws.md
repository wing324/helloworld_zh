## Java之throw/throws+自定义异常类

##### 一、throw和throws区别  

throw:运用于方法内部，用于给调用者返回一个异常对象，和return一样会结束当前方法。  
throws:运用于方法声明之上，用于表示当前方法不处理异常，需要提醒该方法的调用者来处理异常(抛出异常)。  

```java
private static int divide(int num1,int num2) throws Exception{}  
```

##### 二、throw和throws演示  

```java
// throw演示
public class ThrowDemo {

        public static void main(String[] args){
            divide(10,0);
        }
        private static int divide(int num1,int num2){
            System.out.println("begin......");
            if(num2==0){
                throw new RuntimeException("除数不能为0");// 为什么这里不需要throws，因为RuntimeException是可处理可不处理的异常
            }
            System.out.println("----------------------------");
            try {
                int ret = num1/num2;
                System.out.println(ret);
                return ret;
            }catch(ArithmeticException e){
                e.printStackTrace();
            }
            System.out.println("end......");
            return 0;
        }
}
// throw输出
begin......
Exception in thread "main" java.lang.RuntimeException: 除数不能为0
	at com.wing.exceptiondemo.ThrowDemo.divide(ThrowDemo.java:11)
	at com.wing.exceptiondemo.ThrowDemo.main(ThrowDemo.java:6)

// throws演示
public class ThrowsDemo {
    public static void main(String[] args) throws Exception{
        divide(10,0);
    }

    // 在本方法中不处理某种类型的异常，提醒调用者需要来处理该异常
    private static int divide(int num1,int num2) throws Exception{
        System.out.println("begin......");
        if(num2==0){
            throw new Exception("除数不能为0");//为什么这里需要throws，因为Exception是必须处理的异常
        }
        System.out.println("----------------------------");
        try {
            int ret = num1/num2;
            System.out.println(ret);
            return ret;
        }catch(ArithmeticException e){
            e.printStackTrace();
        }
        System.out.println("end......");
        return 0;
    }
}
// throws输出
begin......
Exception in thread "main" java.lang.RuntimeException: 除数不能为0
	at com.wing.exceptiondemo.ThrowDemo.divide(ThrowDemo.java:11)
	at com.wing.exceptiondemo.ThrowDemo.main(ThrowDemo.java:6)
```

##### 三、自定义异常

- 自定义异常类如何定义  

  > - 受检查的异常类: 自定义类并继承于java.lang.Exception  
  > - 运行时期的异常类: 自定义类并继承于java.lang.RuntimeException

- 自定义异常演示  

  ```java
  // 自定义一个RuntimeException异常
  public class SelfExceptionDemo1 extends RuntimeException{
      private static final long serialVersionUID = 1L;

      public SelfExceptionDemo1() {
      }

      public SelfExceptionDemo1(String message) {
          super(message);
      }

      public SelfExceptionDemo1(String message, Throwable cause) {
          super(message, cause);
      }
  }

  // 使用自定义异常
  public class RegisterDemo {
      public static void main(String[] args) {
  //        for(String name : names){
  //            System.out.println(name);
  //        }
          try {
              checkusername("Domain");
          } catch (SelfExceptionDemo1 e){
              System.out.println(e.getMessage());
          }
      }
      private static String[] names={"Wing","Domain","Will"};
      public static boolean checkusername(String username){
          for(String name : names){
              if(name.equals(username)){
                  throw new SelfExceptionDemo1("duplicate username: "+username);
              }
          }
          return true;
      }
  }

  // 返回
  duplicate username: Domain
  ```

  ​