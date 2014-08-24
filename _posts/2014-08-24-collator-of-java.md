---
layout:     post
title:      利用Java语言中的定序器对字符串排序
category: java
description: 如上一篇文章[数据库中的字符串定序规则](http://renxm.com/database/2014/08/22/data-bound-collation-of-database.html)所提到的，定序规则是应用程序对不同语言的语义习惯定制的字符串比较或者排序规则。 Java作为应用最广泛的开发语言之一，也有对定序规则的实现类。
keywords: collator java sort
---
如上一篇文章[数据库中的字符串定序规则](http://renxm.com/database/2014/08/22/data-bound-collation-of-database.html)所提到的，定序规则是应用程序对不同语言的语义习惯定制的字符串比较或者排序规则。

Java作为应用最广泛的开发语言之一，也有对定序规则的实现类。

##java.text.Collator
Collator，中文可以称作定序器。它是Object的直接子类，实现了Cloneable和Comparator接口。所以，在java程序任何使用Comparator的地方都可以使用Collator。

Collator是一个抽象类，要获取它的对象实例需要调用Collator.getInstance()函数。执行getInstance函数时还可以指定使用的定序规则，比如Locale.ENGLISH。

根据[Unicode Consortium](http://www.unicode.org/)的相关文档，java.text.Collator也实现了四种定序算法的敏感程度：

* PRIMARY: “a” 和 “b” 的区别属于这一级。
* SECONDARY："a" 和 "ä" 的区别属于这一级。
* TERTIARY："a" 和 "A" 的区别属于这一级，即大小写敏感。
* IDENTICAL：在这一级，所有Unicode编码不同的字符都视为不同的字符，顺序取决于编码值。

下面是一段使用Collator的代码示例：

    import java.text.Collator;
    import java.util.*;

    public class test
    {
      public static void main(String[] args) throws Exception
      {
          String[] strs4Test = {
                  "ZOO", "alarm", "zoo", "Alarm"
          };
          TreeSet<String> seqBinary = null;
          TreeSet<String> seqEnglish = null;
          seqBinary = new TreeSet<String>();
          Collator enCollator =Collator.getInstance(Locale.ENGLISH);
          enCollator.setStrength(Collator.IDENTICAL);
          seqEnglish = new TreeSet<String>(enCollator);
          for (String str:strs4Test) {
              seqBinary.add(str);
              seqEnglish.add(str);
          }
          System.out.println("Binary sequence:");
          for(String s:seqBinary)
              System.out.println(s);
          System.out.println("English sequence:");
          for(String s:seqEnglish)
              System.out.println(s);
      }
    }

程序输出：

    Binary sequence:
    Alarm
    ZOO
    alarm
    zoo
    English sequence:
    alarm
    Alarm
    zoo
    ZOO

##java.text.RuleBasedCollator
Collator类支持很有限的几个定序规则，要想指定其他自定义规则，就要用到RuleBasedCollator。

RuleBasedCollator是Collator的子类，支持以字符串的形式传入自定义规则。下面的代码就定义了三种不同的自定义定序规则，具体内容一目了然，不在赘述。

    String rule1 = ("< a < b < c");
    String rule2 = ("< c < b  < a");
    String rule3 = ("< c < a  < b");

    RuleBasedCollator rb1 = new RuleBasedCollator(rule1);
    RuleBasedCollator rb2 = new RuleBasedCollator(rule2);
    RuleBasedCollator rb3 = new RuleBasedCollator(rule3);


##java.text.CollationKey
当字符串的比较次数较多时，直接使用Collator效率比较低。提高效率的方法之一就是使用CollationKey类，这比直接使用Collator.compare()要快上不少。

需要调用Collator.getCollationKey()来获取字符冲对应的CollationKey，而且做比较的CollationKey只能是同一个Collator生成的，否则没有意义。
