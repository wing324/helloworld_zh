## Java中a++和++a的区别

##### 首先、看个代码

```java
package wing;

public class justAtest {

	public static void main(String[] args){
		int a=3,b;
		b = a++;
		int c=3,d;
		d = ++c;
		System.out.println("b="+b+",a="+a);
		System.out.println("c="+c+",d="+d);
	}
}
```

##### 然后、瞅瞅结果

```java
b=3,a=4
c=4,d=4
```

##### 再然后、、、

这个结果咋和我们想象的不一样呢？为什么b和d的结果不一样呢？为什么a和c的结果也不一样呢？为什么为什么为什么呢？？？

##### 最后、给个解释

- 为什么a=4,b=3呢？  
  因为a++的时候，a将自己初始化的值3放入临时内存空间中，然后自身加1，再将临时内存空间内存的值赋值给b，所以此时a=4,b=3
- 为什么c=4,d=4呢？
  因为++c的时候，c会自加1，然后再将加1后的c赋值给d，所以c=4,d=4。

