title: Mysql double数据精度导致的相加结果不准确问题
date: 2015-12-03 14:08
tags:
- mysql
categories: 技术日志
comments: true
---

![](/images/mysql/mysql.jpg)
<!--more-->

*** 一、问题描述及分析***

![图1](/images/mysql/doublejingdu.png)

上图中显示的数据是从一张视图中取出的，相信你一眼就可以看出百分数的那一列是有问题的，下面是生成那列数据的sql：

```sql
...略
       ((`p`.`rate`+ `p`.`extend_rate`) * 100) AS `ratePercent
.....略
  from((((`invest` `i`
```

从上面的sql中可以看出该列数据是由另外两列数据相加得到的，先对这两列数据的类型作个说明，rate的值为6.0,，数据类型为double，extend_rate的值为1.0，类型也是double，如下图

![图2](/images/mysql/shujuleixingerror.png)

这样的两列数据使用sql相加会得类似6.9999999这样的数据,下面是纠正后结果：

![图3](/images/mysql/shujuleixing.png)

比较下图3和图2有什么不同，可以发现由double变成了double(10,5)
这是官方给出的说明：
***
Floating-point numbers sometimes cause confusion because they are approximate and not stored as exact values. A floating-point value as written in an SQL statement may not be the same as the value represented internally. Attempts to treat floating-point values as exact in comparisons may lead to problems. They are also subject to platform or implementation dependencies. The FLOAT and DOUBLE data types are subject to these issues. For DECIMAL columns, MySQL performs operations with a precision of 65 decimal digits, which should solve most common inaccuracy problems.
***

大致意思是这样的：浮点数有时可能会给你造成一些困惑，因为它是不精确的，存储的时候也不会使用精确值。在SQL中写出的浮点数可能和数据库内部表示并不相同。将浮点数当做精确值进行比较可能会导致一些问题。同时它的值也依赖于具体的平台和实现方式。FLOAT 和 DOUBLE 数据类型依赖于这些条件。对于 DECIMAL 列来讲,MySQL 使用了65位的数值精度表示，对于这一精度可以解决大多数的货币问题。详细资料可以看下这篇文章：[Problems with Floating-Point Values](http://dev.mysql.com/doc/refman/5.7/en/problems-with-float.html)

到这里我们明白了出问题的原因，浮点数是一个不精确的值，其在数据库内部的值并不是我们看到的那样，即可能看到的extend_rate的值是1.0，但内部表示使用的却是0.99999....这样的数据，所以相加的结果就可想而知了，但是指定精度后就不一样了，当我们为double指定精度后，如上面的DOUBLE(10,5)的精度为5位小数，因此当我们的小数位数小于5时不会出现误差，而大于5时则会出现四舍五入，导致数据结果不精确。

以下内容摘自博客:[mysql下float类型使用一些误差详解](http://www.jb51.net/article/31723.htm),感谢作者的分享:

浮点数是用来表示实数的一种方法，它用 M(尾数) * B( 基数)的E(指数）次方来表示实数，相对于定点数来说，在长度一定的情况下，具有表示数据范围大的特点。但同时也存在误差问题，这就是著名的浮点数精度问题！浮点数有多种实现方法，计算机中浮点数的实现大都遵从 IEEE754 标准，IEEE754 规定了单精度浮点数和双精度浮点数两种规格，单精度浮点数用4字节（32bit）表示浮点数，格式是：1位符号位，8位表示指数 23位表示尾数，双精度浮点数8字节（64bit）表示实数，格式是：1位符号位 11位表示指数 52位表示尾数，同时，IEEE754标准还对尾数的格式做了规范：d.dddddd...，小数点左面只有1位且不能为零，计算机内部是二进制，因此，尾数小数点左面部分总是1。显然，这个1可以省去，以提高尾数的精度。由上可知，单精度浮点数的尾数是用24bit表示的，双精度浮点数的尾数是用53bit表示的，转换成十进制：

```
2^24 - 1 = 16777215；  2^53 - 1 = 9007199254740991
```

由上可见，IEEE754单精度浮点数的有效数字二进制是24位，按十进制来说，是8位；双精度浮点数的有效数字二进制是53位，按十进制来说，是16位。显然，如果一个实数的有效数字超过8位，用单精度浮点数来表示的话，就会产生误差！同样，如果一个实数的有效数字超过16位，用双精度浮点数来表示，也会产生误差！对于 1310720000000000000000.66 这个数，有效数字是24位，用单精度或双精度浮点数表示都会产生误差，只是程度不同：   
单精度浮点数：1310720040000000000000.00；双精度浮点数： 1310720000000000000000.00 可见，双精度差了 0.66 ，单精度差了近4万亿！

以上说明了因长度限制而造成的误差，但这还不是全部！采用IEEE754标准的计算机浮点数，在内部是用二进制表示的，但在将一个十进制数转换为二进制浮点数时，也会造成误差，原因是不是所有的数都能转换成有限长度的二进制数。对于131072.32 这个数，其有效数字是8位，按理应该能用单精度浮点数准确表示，为什么会出现偏差呢？看一下这个数据二进制尾数就明白了 10000000000000000001010001......     显然，其尾数超过了24bit，根据舍入规则，尾数只取 100000000000000000010100，结果就造成测试中遇到的“奇怪”现象！131072.68 用单精度浮点数表示变成 131072.69 ，原因与此类似。实际上有效数字小于8位的数，浮点数也不一定能精确表示，7.22这个数的尾数就无法用24bit二进制表示，当然在数据库中测试不会有问题（舍入以后还是7.22），但如果参与一些计算，误差积累后，就可能产生较大的偏差

*** 二、总结 **
1、浮点数存在误差问题；
2、对货币等对精度敏感的数据，应该用定点数表示或存储；
3、在某些情况下可以通过指定数据类型的精度来避免出现误差