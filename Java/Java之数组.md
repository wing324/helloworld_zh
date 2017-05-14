## Java之数组

##### 数组的内存空间

> 数组其实是对象的一种，所以它存在堆内存空间中。

##### 数组的三种定义格式

```java
// 数组定义格式1
int[] arr1= new int[3];

//数组定义格式2
int[] arr2= new int[]{9,8,7,6};

//数组定义格式3
int[] arr3={0,1,2,3};
```

##### 数组的操作方法

- 数组遍历

```java
int[] arr3={0,1,2,3};
System.out.println(arr3[2]);
System.out.println(arr3.length);
for(int x=0;x<arr3.length;x++){
	System.out.println(arr3[x]);
}
// 结果
0
1
2
3
```

- 获取最值（最大值、最小值）

```java
// 最大值
public static void main(String[] args){
	int[] arr1={-34,-24,-98,-10,-100,-55};
	int maxvalues = GetMaxArr(arr1);
	System.out.println(maxvalues);
}
public static int GetMaxArr(int[] arr){
	int max=arr[0];
	for(int x=1;x<arr.length;x++){
		if(arr[x]>max){
			max=arr[x];
		}
	}
	return max;
}
// 结果
100
```

- 选择排序

```java
public static void SelectSort(int[] arr){
	for(int x=0;x<arr.length-1;x++){
		for(int y=x+1;y<arr.length;y++){
			if(arr[x]>arr[y]){
				int temp=arr[x];
				arr[x]=arr[y];
				arr[y]=temp;
			}
		}
	}
}
```

- 冒泡排序

```java
public static void BubbleSort(int[] arr){
	for(int x=0;x<arr.length-1;x++){
		for(int y=0;y<arr.length-1-x;y++){
			if(arr[y]>arr[y+1]){
				int temp=arr[y];
				arr[y]=arr[y+1];
				arr[y+1]=temp;
			}
		}
	}
}
```

- Java内置数组排序功能

```java
import java.util.*;
Arrays.sort(arr1)
```

- 折半查找

```java
/*
* 二分查找法（折半查找法）
*/
public static int halfSearch(int[] arr,int value){
	int minindex,midindex,maxindex;
	minindex=0;
	maxindex=arr.length-1;
	midindex=(maxindex+minindex)/2;
	while(arr[midindex]!=value){
		if(arr[midindex]>value){
			maxindex=midindex-1;
		}else{
			minindex=midindex+1;
		}
		if(maxindex<minindex){
			return -1;
		}
		midindex=(maxindex+minindex)/2;
	}
	return midindex;
}
```

