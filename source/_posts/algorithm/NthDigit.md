title: 正整数序列求第n个数字
date: 2016-12-11 11:12:00
tags:
- 算法
categories: 技术日志
comments: true
---
摘自LeetCode，原题链接地址[https://leetcode.com/problems/nth-digit/](https://leetcode.com/problems/nth-digit/)

LeetCode OJ is a platform for preparing technical coding interviews. Pick from an expanding library of more than 190 questions, code and submit your solution to see if you have solved it correctly. It is that easy!
<!--more-->

说到算法，大部分是从个别情况中寻找一般性规律，考验归纳能力，这道题算是比较简单的，我们先来分析一下。

按照上述解题步骤，先来列出一些例子，如下：

n=9（看过原题的同学应该n指的的是什么，在此不再赘述）
结果为 9

n=11时，由于超过9后，数字为两位数，这样第11个数字就是10的第二位数字0。

下面我们来看下有两个数字组成的即两位数有多少个，从10....99,很容易看出是90个，那么三位数呢，即从100到999，可以看出是900个，以此可以类推出四位数有9000个，五位数90000个。

知道这个规律后，我们很容易的就可以计算出n在几位数的区间中，由于每个区间中的数字位数都是一样的，进而就可以很容易计算出是该区间中的第几个数，最后就可以计算出n所指的数字。

代码如下：

```java
public class Solution {
        public int findNthDigit(int n) {
            long length=1,g=9,start=1;
            while(n>length*g){
                n -= length*g;
                length++;
                g *= 10;
                start *= 10;
            }
            Long x = n%length;
            long b = n/length;
            //获取要取得的数字
            String a = String.valueOf(start+b-1+(x==0?0:1));
            if(x==0){
                return Integer.valueOf(a.substring(a.length()-1));
            }
            return Integer.valueOf(a.substring(x.intValue()-1,x.intValue()));
        }
}

```

