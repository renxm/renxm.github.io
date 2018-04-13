---
layout:     post
title:     Javascript 的数据类型
category:  Javascript
description: 这里介绍了 Javascript 的数据类型。
keywords:  Javascript Types
---
最新的 ECMAScript 标准定义了 7 种数据类型:

* 6 种原始类型:
    * Boolean
    * Null
    * Undefined
    * Number
    * String
    * Symbol (ECMAScript 6 新定义)
* 以及 Object
    * Array
    * Boolean
    * Date
    * Function
    * Math
    * Number
    * RegExp
    * String

# 基本包装类型
除了 null 和 undefined, 所有的基本类型值都有一个包含这个基本类型值的等价对象：
* String for the string primitive.
* Number for the number primitive.
* Boolean for the Boolean primitive.
* Symbol for the Symbol primitive.

包装类型对象的 valueOf() 方法可以返回它包含的基本类型值。

## 包装类型的坑
```js
var str = 'string';
str.pro = 'hello';
console.log(str.pro + 'world');
```
上面的代码会输出什么？

要回答这个问题，需要深入理解包装类型的执行过程。

每一次在基本类型变量 str 上执行一个方法的时候，都会有如下过程：
* 创建String类型的一个实例
* 在实例上调用指定的方法
* 销毁这个实例

str.pro在二次调用的时候，其实上一句的包装类型对象已经销毁了，所以，输出的是： undefinedworld。
