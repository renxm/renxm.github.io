---
layout:     post
title:      本文总结了Java程序员常犯的10个错误
category: programming
keywords:  java programming 
---
本文总结了Java程序员常犯的10个错误。

#1. 把Array转化成ArrayList

把Array转化成ArrayList，程序员经常用以下方法：
```java
List<String> list = Arrays.asList(arr);
Arrays.asList()
```
实际上返回一个ArrayList，但是这个ArrayList是Arrays的一个内部私有类，而不是java.util.ArrayList类。这个私有类java.util.Arrays.ArrayList有set(), get(), contains()方法，但是不能够添加新的元素。它的大小是固定的。如果你想要一个java.util.ArrayList，正确的方法是:
```java
ArrayList<String> arrayList = new ArrayList<String>(Arrays.asList(arr));
```
java.util.ArrayList的构造函数可以接受一个集合类型。java.util.Arrays.ArrayList也继承了集合类型，所以可以作用参数使用。

#2. 检查数组是否包含一个值

开发人员经常做的是：
```java
Set<String> set = new HashSet<String>(Arrays.asList(arr));
return set.contains(targetValue);
```
这个代码是工作的，但没有没有效率。把列表转换成set没有必要，需要额外的时间。正确的方法是：
```java
Arrays.asList(arr).contains(targetValue);
```
或者，一个简单的loop：
```java
for(String s: arr){
	if(s.equals(targetValue))
		return true;
}
return false;
```
第一种比第二种更具有可读性。

#3. 在循环中删除一个列表元素

考虑下面的代码，迭代过程中删除元素：
```java
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
for (int i = 0; i < list.size(); i++) {
	list.remove(i);
}
System.out.println(list);
```
这段代码的输出是：
>[b, d]

这个方法有一个严重的问题。当元素被移除，该列表的大小缩减，元素索引也随之发生了变化。所以，如果你想通过使用索引来删除一个循环内的多个元素，就会导致错误的结果。

你可能猜到可以使用iterator来删除循环中的元素。在Java中的foreach循环的工作原理就像一个iterator。 但是在这里也会发生错误。请看下面的代码：
```java
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
 
for (String s : list) {
	if (s.equals("a"))
		list.remove(s);
}
```
上面的foreach loop代码会抛出一个异常ConcurrentModificationException. 但是下面这段代码不会。
```java
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
	String s = iter.next();
 
	if (s.equals("a")) {
		iter.remove();
	}
}
```
通过分析ArrayList.iterator()的原代码，我们可以发现next()方法必须要在remove()方法前被调用。在foreach loop中，编译器产生的代码会先调用next()方法，从而产生异常ConcurrentModificationException。请查看ArrayList.iterator()的原代码。

#4. Hashtable 与 HashMap

按照算法惯例，Hashtable是数据结构的名称。但在Java中，数据结构的名称是HashMap。Hashtable是同步的版本。所以很多时候你并不需要Hashtable，而是HashMap。 这两篇文章详细介绍了各种Map的区别和常见的问题： [*HashMap vs. TreeMap vs. Hashtable vs. LinkedHashMap*](http://www.programcreek.com/2013/03/hashmap-vs-treemap-vs-hashtable-vs-linkedhashmap/), [*Map常见10大问题*](http://www.programcreek.com/2013/09/top-9-questions-for-java-map/)。

#5.使用原始类型Collection

在Java中，原始类型和无界通配符类型很容易混在一起。以Set为例，Set是原始类型，而Set<?>是无界通配符类型。

考虑下面的代码，它使用原始类型的List作为参数：
```java
public static void add(List list, Object o){
	list.add(o);
}
public static void main(String[] args){
	List<String> list = new ArrayList<String>();
	add(list, 10);
	String s = list.get(0);
}
```
此代码将抛出一个异常：
>Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
	at ...
	
使用原始类型的Collection是危险的，因为原始类型的Collection跳过类型检查。另外值得一提的是Set, Set< ? >, Set< Object >之间存在着巨大的差异。 了解更多，请查看[*原始类型 vs. 无界通配符类型*](http://www.programcreek.com/2013/12/raw-type-set-vs-unbounded-wildcard-set/) 和 [*类型擦除*](http://www.programcreek.com/2011/12/java-type-erasure-mechanism-example/)。

#6. 访问级别

很多时候，开发者使用public修饰字段。这样做的好处是很容易通过直接引用来获取字段的值，但是这是一个非常糟糕的设计。经验法则是“给成员的访问级别尽可能低”。可以查看[*Java4种不同的访问级别public, default, protected, and private*](http://www.programcreek.com/2011/11/java-access-level-public-protected-private/)。

#7. ArrayList 与 LinkedList

当开发人员不知道ArrayList和LinkedList的区别的时候，他们经常使用的是ArrayList，可能因为它看起来面熟。但是ArrayList和LinkedList之间有巨大的性能差异。 简单来说如果有大量的添加/删除操作，而没有很多随机存取操作，LinkedList的应该是首选。

#8.可变性与不变性

不可变对象有很多优点，如简单性，安全性等。但是它需要为每个不同的值创造一个单独的对象，对象太多可能会导致垃圾回收的成本高。所以可变和不可变之间进行选择时应该有一个平衡。

一般情况下，使用可变对象，以避免产生过多的中间对象。一个经典的例子是串联了大量的字符串。如果使用的是不可变的字符串String，会产生很多可以垃圾回收的对象。这样既浪费时间也浪费CPU的运算能力，使用可变对象是正确的解决方案（如StringBuilder）。
```java
String result="";
for(String s: arr){
	result = result + s;
}
```
另外一些情况，可变对象刚更加合适可取。例如排序(Collections.sort())。如果Collection是不可变的，排序方法每次将会返回一个新的Collection，这样会极其浪费资源。 可以看看[*为什么在Java中String被设计成不可变？*](http://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)

#9. 父类和子类的构造函数
```java
class Super {
    String s;
    public Super(String s) {
        this.s = s;
    }
}
public class Sub extends Super {
    int x = 200;
    public Sub(String s) {
    }
    public Sub(){
        System.out.println("Sub");
    }
    public static void main(String[] args) {
        Sub s = new Sub();
    }
}
```

以上这段代码出现编译错误，因为默认的父类构造函数未定义。在Java中，如果一个类没有定义构造函数，编译器会默认插入一个默认的无参数构造函数。如果程序员定义构造函数，编译器将不插入默认的无参数构造函数。上面的代码由于自定义了有参数的构造函数，编译器不再插入无参数的构造函数。子类的构造函数，无论是有参数或无参数，都将调用父类无参构造函数。当子类需要父类的无参数构造函数的时候，就发生了错误。

解决这个问题，可以1）增加一个父类构造函数
```java
public Super(){
    System.out.println("Super");
}
```
，或2）删除自定义的父类构造函数，或3）添加super(value)到子类构造函数。更多请查看父类和子类的构造函数。

#10. "" 与 Constructor?

字符串可以通过两种方式创建：
```java
//1. use double quotes
String x = "abc";
//2. use constructor
String y = new String("abc");
```
这两者有什么区别呢？ 下面的例子可以提供一个快速的答案：
```java
String a = "abcd";
String b = "abcd";
System.out.println(a == b);  // True
System.out.println(a.equals(b)); // True
 
String c = new String("abcd");
String d = new String("abcd");
System.out.println(c == d);  // False
System.out.println(c.equals(d)); // True
```
关于它们是如何分配内存的更多信息，请查看[*创建Java字符串使用“”或构造函数？*](http://www.programcreek.com/2014/03/create-java-string-by-double-quotes-vs-by-constructor/)

小结

以上是我根据GitHub上的开源项目，Stack Overflow上的问题，和谷歌热门搜索词所做的总结。虽然它们不是准确的top 10，但很常见的。如果你有不同的观点或者指出更常见的错误，请留言。我也会更新这个列表。非常感谢。

[Source Link](http://www.programcreek.com/2014/05/top-10-mistakes-java-developers-make/)