## Java之集合框架比较

##### 一、List接口

**共同点：**

- 允许元素重复
- 记录元素的先后添加顺序

**不同点：**

> 1. Vector类：
>    - 底层采用数组结构算法，方法都使用了synchronized修饰，线程安全，但是性能相对于ArrayList较低（该类已经被ArrayList取代）
> 2. ArrayList类：
>    - 底层采用数组结构算法，方法都没有使用synchronized修饰，线程不安全，但是性能相对于Vector较低
>      为了保证线程安全，可以使用`List list = Collections.synchronizedList(new ArrayList(...));`
> 3. LinkedList类：
>    - 底层采用双向链表结构算法，方法都没有使用synchronized修饰，线程不安全

**数组结构算法 VS 双向链表结构算法**

> 数组结构算法：插入和删除操作速度低，查询和更新较快
>
> 双向链表结构算法：插入和删除操作速度快，查询和更新较慢

**使用选择：**

Vector类：拒绝使用

ArrayList类：插入和删除频繁的时候使用

LinkedList类：查询和更新频繁的时候使用

##### 二、Set接口

**共同点：**

- 不允许重复数据
- 都不是线程安全的类，即没有使用synchronized同步  
  但是可以通过`Set s = Collections.synchronizedSet(Set对象);`方式解决。

**不同点：**

>1. HashSet类
>   - 不保证元素的先后添加顺序
>   - 底层采用了Hash算法，查询效率极高
>   - 判断两个对象是否相等的规则为：equals为true + hashCode相同
>     所以要求哈希中的对象元素都得覆盖equals和hashCode方法。
>2. LinkedHashSet类
>   - 保证元素的先后添加顺序
>   - HashSet的子类，底层采用了Hash算法，也采用了链表算法来维持元素的先后添加顺序，所以查询效率相对于HashSet较低
>   - 判断两个对象是否相等的规则与HashSet相同
>3. TreeSet类
>   - 不保证元素的先后添加顺序，但会对集合中的元素做自然排序或者定制排序
>   - 底层使用了红黑树算法，范围查询效率极高，如索引。

**使用选择：**

HashSet类：等值查询频繁且不需要保证元素先后添加顺序的时候使用

LinkedHashSet类：等值查询频繁且需要保证元素先后添加顺序的时候使用

TreeSet类：范围查询频繁且不需要保证元素先后添加顺序的时候使用

##### 三、Map接口

**共同点：**

**不同点：**