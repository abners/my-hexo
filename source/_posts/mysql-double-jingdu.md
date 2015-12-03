title: Mysql double数据精度导致的相加结果不准确问题
date: 2015-12-03 14:08
tags:
- mysql
categories: 技术日志
comments: false
---

![图1](/images/mysql/doublejingdu.png)

图片中显示的数据是从一张视图中取出的，相信你一眼就可以看出百分数的那一列是有问题的，下面是生成那列数据的sql：

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

大致意思是这样的：浮点数有时可能会给你造成一些困惑，因为它是不精确的，存储的时候也不会使用精确值。。。。

详细资料可以看下这篇文章：[Problems with Floating-Point Values](http://dev.mysql.com/doc/refman/5.7/en/problems-with-float.html)