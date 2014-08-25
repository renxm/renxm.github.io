---
layout:     post
title:      数据库中的字符串定序规则
category: database
description: Collation，中文对应词为“定序”或“定序规则”，意指字符（串）比较和排序的一组规则。
keywords: 定序 定序规则 collation oracle database 
---
Collation，中文对应词为“定序”或“定序规则”，意指字符（串）比较和排序的一组规则。

##简介
代码世界里最简单的定序规则就是根据字符编码的值来决定对应字符串的顺序，这种方法简单粗暴，但在大多数语言里这个规则不符合语义习惯。

英语习惯上，字母表顺序要比大小写的优先级高，所以在字典里“aaa”排在“ZZZ”的前面。若按照它们在字符集（ASCII）中的值来排序，结果就是“ZZZ”在“aaa”的前面。

非拉丁字符语言的排序方式就更复杂，比如，汉语就有拼音顺序和笔画顺序。实际的软件系统中用字符集编码来无脑定序显然的不友好的，定序规则就是来解决这个问题的。

##数据库相关
传统的关系型数据库都有对定序规则的支持，以满足国际化的需求。大型数据库由于客户众多，对国际化的支持很完善，相对应地定序规则也很多。
比如，Oracle 12c中的定序规则多达115种。在sqlPlus中可以使用如下语句查询全部的定序规则，其中后缀“_M”表示支持在主语言中夹杂多国语言的情况，如“SPANISH_M”。

    SELECT VALUE FROM V$NLS_VALID_VALUES WHERE PARAMETER='SORT'

定序规则在数据库中具体实现时，通常会把一个字符串按照一定算法映射为一个唯一的数值，这个数值被称为定序键（Collation Key）。对于两个字符串，可以逐字节比较它们定序键的二进制值，结果就是定序键所对应的字符串在特定定序规则中的比较结果。

由于数据库的SQL以及扩展SQL语言有多种操作符以字符类型为操作数，在实现定序规则时还要考虑执行操作时所使用的定序规则，以及操作结果所遵循的定序规则。

一般主流关系型数据库都支持新建DB时、新建表时、执行SQL语句时指定定序规则，比如SQLServer, MySQL。

唯独Oracle并不支持在新建表示指定特定列的定序规则，截止最新的12c版本都是如此。 预计在12c的下一个子版本，有望支持这个功能。

##Oracle
Oracle数据库目前在SQL语句中对定序的支持主要基于一个函数————NLSSORT。

NLSSORT接受两个参数，一个是想要转化的字符变量（char），另一个是代表定序规则的字符串（nlsparam）。这两个参数可以是Oracle字符类型（CHAR/VARCHAR2/NCHAR/NVARCHAR2）中的任意一个。它的返回值是RAW类型，代表第一个参数的定序键。
代表定序规则的参数有如下格式要求，其中COLLATION是某个特定语言定序规则的名字，如ENGLISH。BINARY是字符编码定序规则的名字。如果省略第二个参数，NLSSORT函数就会使用当前SESSION默认的定序规则，对应的SESSION参数是NLS_SORT。

    'NLS_SORT = collation'

在指定定序规则名字时，可以附加两个后缀：“_ai”代表对口音不敏感；“_ci”代表对大小写不敏感。但是，在ORDER BY语句中使用NLSSORT函数时，不建议使用这两个后缀，因为它们会使排序结果不确定。

下面是一个简单的例子，对比了在查询中使用和不使用NLSSORT函数带来的差别：

    CREATE TABLE test (name VARCHAR2(15));
    INSERT INTO test VALUES ('Gaardiner');
    INSERT INTO test VALUES ('Gaberd');
    INSERT INTO test VALUES ('Gaasten');

    SELECT *
      FROM test
      ORDER BY name;

    NAME
    ---------------
    Gaardiner
    Gaasten
    Gaberd

    SELECT *
      FROM test
      ORDER BY NLSSORT(name, 'NLS_SORT = XDanish');

    NAME
    ---------------
    Gaberd
    Gaardiner
    Gaasten

