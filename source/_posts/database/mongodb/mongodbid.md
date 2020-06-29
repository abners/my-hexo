title: mongodb文档id生成策略
date: 2017-04-19 16:22:00
tags:
- 数据库
- mongodb
categories: 技术日志
comments: true
---

MongoDB存储的文档必须有一个的“_id”键，这个键值可以是任意类型，默认的是ObjectId类型对象
<!--more-->
每个文档的_id都是唯一的，可以确保集合中的文档被唯一标识

ObjectId 是_id的默认类型，也可以自己定义id


MongoDB采用ObjectId，而不是其他比较常规的做法（比如自动增加的主键）的主要原因，因为在多个服务器上同步自动增加主键值既费力还费时。（不同机器可以通过机器名区分，同一个机器通过时间戳、随机数、进程id区分）

ObjectId 采用12字节的存储空间，每个字节两位16进制数字，是一个24位的字符串。

12位生成规则：

	[0,1,2,3] [4,5,6] [7,8] [9,10,11]
	
	时间戳 |机器码 |PID |计数器

	前四位是时间戳，可以提供秒级别的唯一性。
	接下来三位是所在主机的唯一标识符，通常是机器主机名的散列值。
	
	接下来两位是产生ObjectId的PID，确保同一台机器上并发产生的ObjectId是唯一的。前九位保证了同一秒钟不同机器的不同进程产生的ObjectId时唯一的。
	
	最后三位是自增计数器，确保相同进程同一秒钟产生的ObjectId是唯一的。

获取文档的时间戳

由于ObjectId的前4位是时间戳，因此不需要为文档特意的保存时间戳，可以用getTimestamp()获取
```
> ObjectId("56c56dd4ca446fab71e4c38a").getTimestamp()
ISODate("2016-02-18T07:08:04Z")

```

ObjectId转化为时间戳

```

> new ObjectId().str
56c68c3b64799370c0ef3581
> ObjectId("56c56dd4ca446fab71e4c38a").str
56c56dd4ca446fab71e4c38a

```