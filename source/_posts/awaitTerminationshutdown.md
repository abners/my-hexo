title: 多线程awaitTermination和shutdown的使用问题
date: 2014-12-18 16:03
tags:
- 多线程
- java
categories: 技术日志
comments: false
---

最近做一个抓取网页数据的任务，由于需要抓取的数据量比较大，并且抓取的间隔比较短，每次抓取任务启动后会有多次网络请求，为了提高抓取的效率采用了多线程的方式实现
<!--more-->

*** 
```java 
ExecutorService pool = new ThreadPoolExecutor(CORE_POOL_SIZE,MAX_POOL_SIZE, 
	KEEPALIVETIME, TimeUnit.MILLISECONDS,new LinkedBlockingDeque<Runnable>());
for (BaseInfoPO baseInfoPO : baseInfos) {
	UseThread useThread = new UseThread(baseInfoPO );
	pool.execute(useThread);
}
pool.shutdown();
pool.awaitTermination(Long.MAX_VALUE, TimeUnit.DAYS);
```
***
初始时采用了上图的实现方式，其中CORE_POOL_SIZE为10，max_pool_size为16，采用此方法启动后发现程序一直处于运行状态，无法结束，通过jdk自带的工具JVM查看程序的执行状态后发现，此时所有线程都处于await状态

![图1.线程截图](/images/20141218160219843.jpg)

开始时认为既然所有的线程都处于等待状态，一定是资源同步出现了问题，导致所有线程都在等待某个共享资源锁的释放，于是重新检查了一下对临界资源的处理是否存在问题，发现并没有什么问题。
这就怪了，既然并没有任何资源占用而没有释放，为什么程序仍不能正常结束呢？
Google了一下这种情况产生的各种可能原因，仍不能解决遇到的问题，无奈之下看了一眼日志，看完后就更奇怪了，日志中显示每个线程的任务都正确的执行结束了，正常情况下java虚拟机应该停止了，这样的话应该不是子线程中出现了问题，而是主线程出现了问题，JVM中显示的10个线程不是出现了问题，而是线程池中的线程，因为CORE_POOL_SIZE为10所以会有是10个线程处于等待状态，等待分配任务。
接下来开始检查主线程中代码（即上图），其实上面的代码本身并没有任何问题，我们先来看一下java官方文档对pool.awaitTermination方法的注释：

```java 
 /**
   * Blocks until all tasks have completed execution after a shutdown
   * request, or the timeout occurs, or the current thread is
   * interrupted, whichever happens first.
   *
   * @param timeout the maximum time to wait
   * @param unit the time unit of the timeout argument
   * @return <tt>true</tt> if this executor terminated and
   *         <tt>false</tt> if the timeout elapsed before termination
   * @throws InterruptedException if interrupted while waiting
   */ 
```
大概意思是这样的：该方法调用会被阻塞，直到所有任务执行完毕并且shutdown请求被调用，或者参数中定义的timeout时间到达或者当前线程被打断，这几种情况任意一个发生了就会导致该方法的执行。
这样就明白了：
当我们调用pool.awaitTermination时，首先该方法会被阻塞，这时会执行子线程中的任务，子线程执行完毕后该方法仍然会被阻塞，因为shutdown()方法还未被调用，而代码中将shutdown的请求放在了awaitTermination之后，这样就导致了只有awaitTermination方法执行完毕后才会执行shutdown请求，这样就造成了死锁。
解决办法自然就很简单了，调换下位置就可以了。