title: 改善Java中锁的性能
date: 2015-03-31 16:03
tags:
- 多线程
- java
categories: 技术翻译
comments: true
---
本文由伯乐在线[唐尤华](http://group.jobbole.com/category/feedback/trans-team/) 校稿，翻译自 [ www.javacodegeeks.com](http://www.javacodegeeks.com/2015/01/improving-lock-performance-in-java.html) 

两个月前向Plumbr公司引进线程死锁的检测之后，我们开始收到一些类似于这样的询问：“棒极了！现在我知道造成程序出现性能问题的原因了，但是接下来该怎么做呢？”

我们努力为自己的产品所遇到的问题思考解决办法，但在这篇文章中我将给大家分享几种常用的技术，包括分离锁、并行数据结构、保护数据而非代码、缩小锁的作用范围，这几种技术可以使我们不使用任何工具来检测死锁。
<!--more-->
我们努力为自己的产品所遇到的问题思考解决办法，但在这篇文章中我将给大家分享几种常用的技术，包括分离锁、并行数据结构、保护数据而非代码、缩小锁的作用范围，这几种技术可以使我们不使用任何工具来检测死锁。

锁不是问题的根源，锁之间的竞争才是

通常在多线程的代码中遇到性能方面的问题时，一般都会抱怨是锁的问题。毕竟锁会降低程序的运行速度和其较低的扩展性是众所周知的。因此，如果带着这种“常识”开始优化代码，其结果很有可能是在之后会出现讨人厌的并发问题。

因此，明白竞争锁和非竞争锁的不同是非常重要的。当一个线程试图进入 另一个线程正在执行的同步块或方法时会触发锁竞争。该线程会被强制进入等待状态，直到第一个线程执行完同步块并且已经释放了监视器。当同一时间只有一个线 程尝试执行同步的代码区域时，锁会保持非竞争的状态。

事实上，在非竞争的情况下和大多数的应用中，JVM已经对同步进行了优化。非竞争锁在执行过程中不会带来任何额外的开销。因此，你不应该因为性能问题抱怨锁，应该抱怨的是锁的竞争。当有了这个认识之后，让我们来看下能做些什么，以降低竞争的可能性或减少竞争的持续时间。

### 保护数据而非代码

解决线程安全问题的一个快速的方法就是对整个方法的可访问性加锁。例如下面这个例子，试图通过这种方法来建立一个在线扑克游戏服务器：
```java
class GameServer {
  public Map<<String, List<Player>> tables = new HashMap<String, List<Player>>();
 
  public synchronized void join(Player player, Table table) {
    if (player.getAccountBalance() > table.getLimit()) {
      List<Player> tablePlayers = tables.get(table.getId());
      if (tablePlayers.size() < 9) {
        tablePlayers.add(player);
      }
    }
  }
  public synchronized void leave(Player player, Table table) {/*body skipped for brevity*/}
  public synchronized void createTable() {/*body skipped for brevity*/}
  public synchronized void destroyTable(Table table) {/*body skipped for brevity*/}
}
```

作者的意图是好的——当一个新的玩家加入牌桌 时，必须确保牌桌上的玩家个数不会超过牌桌可以容纳的玩家总个数9。

但是这种解决办法事实上无论何时都要对玩家进入牌桌进行控制——即使是在服务器的访问量较小的时候也是这样，那些等 待锁释放的线程注定会频繁的触发系统的竞争事件。包含对账户余额和牌桌限制检查的锁定块很可能大幅提高调用操作的开销，而这无疑会增加竞争的可能性和持续 时间。

解决的第一步就是确保我们保护的是数据，而不是从方法声明移到方法体中的那段同步声明。对于上面那个简单的例子来说，可能改变不大。但是我们要站在整个游戏服务的接口之上来考虑，而不是单单的一个join()方法。

```java
class GameServer {
  public Map<String, List<Player>> tables = new HashMap<String, List<Player>>();
 
  public void join(Player player, Table table) {
    synchronized (tables) {
      if (player.getAccountBalance() > table.getLimit()) {
        List<Player> tablePlayers = tables.get(table.getId());
        if (tablePlayers.size() < 9) {
          tablePlayers.add(player);
        }
      }
    }
  }
  public void leave(Player player, Table table) {/* body skipped for brevity */}
  public void createTable() {/* body skipped for brevity */}
  public void destroyTable(Table table) {/* body skipped for brevity */}
}
```

原本可能只是一个小小的改变，影响的可是整个类的行为方式。玩家无论何时加入牌桌，先前的同步方法都会对整个GameServer实例加锁，进而会与那些同时试图离开牌桌的玩家产生竞争。将锁从方法声明移到方法体中会延迟锁的加载，进而降低了锁竞争的可能性。

### 缩小锁的作用范围

现在，当确信了需要保护的是数据而非程序后，我们应该确保我们只在必要的地方加锁——例如当上面的代码被重构之后：

```java
public class GameServer {
  public Map<String, List<Player>> tables = new HashMap<String, List<Player>>();
 
  public void join(Player player, Table table) {
    if (player.getAccountBalance() > table.getLimit()) {
      synchronized (tables) {
        List<Player> tablePlayers = tables.get(table.getId());
        if (tablePlayers.size() < 9) {
          tablePlayers.add(player);
        }
      }
    }
  }
  //other methods skipped for brevity
}
```

这样那段包含对玩家账号余额检测（可能引发IO操作）的可能引起费时操作的代码，被移到了锁控制的范围之外。注意，现在锁仅仅被用来防止玩家人数超过桌子可容纳的人数，对账户余额的检查不再是该保护措施的一部分了。

### 分离锁

你可以从上面例子最后一行代码清楚的看到：整个数据结构是由相同的锁保护着。考虑到在这一种数据结构中可能会有数以千计的牌桌，而我们必须保护任何一张牌桌的人数不超过容量，在这样的情况下仍然会有很高的风险出现竞争事件。

关于这个有一个简单的办法，就是对每一张牌桌引入分离锁，如下面这个例子所示：

```java
public class GameServer {
  public Map<String, List<Player>> tables = new HashMap<String, List<Player>>();
 
  public void join(Player player, Table table) {
    if (player.getAccountBalance() > table.getLimit()) {
      List<Player> tablePlayers = tables.get(table.getId());
      synchronized (tablePlayers) {
        if (tablePlayers.size() < 9) {
          tablePlayers.add(player);
        }
      }
    }
  }
  //other methods skipped for brevity
}
```
现在，我们只对单一牌桌的可访问性进行同步而不是所有的牌桌，这样就显著降低了出现锁竞争的可能性。举一个具体的例子，现在在我们的数据结构中有100个牌桌的实例，那么现在发生竞争的可能性就会比之前小100倍。

### 使用线程安全的数据结构

另一个可以改善的地方就是抛弃传统的单线程数据结构，改用被明确设计为线程安全的数据结构。例如，当采用ConcurrentHashMap来储存你的牌桌实例时，代码可能像下面这样：

```java
public class GameServer {
  public Map<String, List<Player>> tables = new ConcurrentHashMap<String, List<Player>>();
 
  public synchronized void join(Player player, Table table) {/*Method body skipped for brevity*/}
  public synchronized void leave(Player player, Table table) {/*Method body skipped for brevity*/}
 
  public synchronized void createTable() {
    Table table = new Table();
    tables.put(table.getId(), table);
  }
 
  public synchronized void destroyTable(Table table) {
    tables.remove(table.getId());
  }
}
```
在join()和leave()方法内部的同步块仍然和先前的例子一样，因为我们要保证单个牌桌数据的完整性。ConcurrentHashMap 在这点上并没有任何帮助。但我们仍然会在increateTable()和destoryTable()方法中使用ConcurrentHashMap创建和销毁新的牌桌，所有这些操作对于ConcurrentHashMap来说是完全同步的，其允许我们以并行的方式添加或减少牌桌的数量。

### 其他一些建议和技巧

* 降低锁的可见度。在上面的例子中，锁被声明为public（对外可见），这可能会使得一些别有用心的人通过在你精心设计的监视器上加锁来破坏你的工作。
* 通过查看java.util.concurrent.locks 的API来看一下 有没有其它已经实现的锁策略，使用其改进上面的解决方案。
* 使用原子操作。在上面正在使用的简单递增计数器实际上并不要求加锁。上面的例子中更适合使用 AtomicInteger代替Integer作为计数器。
* 最后一点，无论你是否正在使用Plumber的自动死锁检测解决方案，还是手动从线程转储获得解决办法的信息，都希望这篇文章可以为你解决锁竞争的问题带来帮助。