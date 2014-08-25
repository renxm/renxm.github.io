---
layout:     post
title:      利用Java语言中的定序器对字符串排序
category: java 定序器 collator 国际化 GDK orai18n
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

##GDK
通过上面的程序输出可以看出，Collator对于大小写敏感的排序是非常之奇葩的。本来按照字符集编码顺序和人类习惯，小写字母都应该排在大写字母后面，可是Collator偏偏于此相反。

不过好在大型的开发框架或系统都会提供和国际化相关的支持，比如Oracle的国际化开发工具包（Globalization Development Kit）。这个工具包现在貌似在公网上只能找到10.1版本（[下载地址](http://www.oracle.com/technetwork/topics/gdk-098904.html)），核心是名为orai18n.jar的jar包。

注意，如果在Oracle JDBC Driver页面下载orai18n.jar，得到的是一个不完整的GDK。

GDK使用名为OraCollator的类代替java.text.Collator，其中实现的所有定序规则都是和Oracle数据库一一对应的。它的使用方法和Collator非常类似，可以用一个String类型的参数定序规则名。

下面是一段示例代码：

    import java.text.Collator;
    import oracle.i18n.text.*;
    import java.util.*;

    public class test
    {
      public static void main(String[] args) throws Exception
      {
          Collator javaCollator = Collator.getInstance(Locale.FRENCH);
          javaCollator.setStrength(Collator.TERTIARY);
          if( javaCollator.compare("abc", "ABC") < 0 )
              System.out.println("Using java collator, abc is less than ABC");
          
          OraCollator gdkCollator = OraCollator.getInstance("FRENCH");
          gdkCollator.setStrength(OraCollator.TERTIARY);;
          if( gdkCollator.compare("abc", "ABC") > 0 )
              System.out.println("Using GDK Collator, abc is greater than to ABC");
      }
    }

程序输出：

    Using java collator, abc is less than ABC
    Using GDK Collator, abc is greater than to ABC

