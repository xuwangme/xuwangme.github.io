---
layout:     post
title:      "Java中是如何使用CAS算法的"
subtitle:   " \"Java中使用CAS算法保证操作的原子性\""
date:       2019-5-5 21:54:00
author:     "Xu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Java 并发
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>



## Java中是如何使用CAS算法的
> Auther: xuwang
</br>
> Mail: xuwang.me@gmail.com
</br>
> Created Data: 2019/05/05  &emsp; Modified Data: 2019/05/05
### 

#### 1. 原子性问题

Java多线程中存在线程之间的任务切换，线程的任务切换时间多在时间片结束后。而Java作为高级语言，其一条语句往往需要多条CPU指令完成，例如`count += 1`则需要至少三条CPU指令：

1. 指令 1：首先，需要把变量 count 从内存加载到 CPU的寄存器
2. 指令2：在寄存器中执行+1操作
3. 指令3：将结果写入CPU缓存或者内存

操作系统在执行任务切换时，可以发生在任何一条CPU指令执行完成后。因此可以再指令1、指令2、指令3任何一个CPU指令完成后执行切换。

假设此时有两个线程A和B执行`count += 1`操作，count的初始值为0。如果线程A在执行完指令1之后操作系统执行线程切换到线程B，此时线程B同样执行指令1，那么线程A和线程B从内存中读取的count值都为0。执行流程如下图所示：

<img class="shadow" src="/img/post/Java原子性.jpg" width="260">

那么，当两个线程A、B执行完成后我们得到的结果count值为1，而不是2。

出现这个现象的原因即为没有保证Java语言操作的原子性。

> 原子性的定义：把一个或者多个操作在CPU执行的过程中不被中断的特性称为原子性。

CPU能保证的原子操作是CPU指令级别的，而一条Java语句操作可能由多个CPU指令构成，因此需要解决Java语句操作的原子性问题。



#### 2. Java如何解决原子性问题

解决原子性问题有两种方式：

1. 采用悲观锁：在Java语言中可以使用synchronized关键字或者Lock接口的实现类来保证操作的原子性
2. 采用乐观锁：乐观锁最常见的就是CAS

> 注意：volatile关键字并不能保证原子性，volatile关键字可以解决可见性、有序性的问题。

采用悲观锁的方式会导致并发量降低：

1. 直接使用synchronized关键字会导致同一时间只能有一个线程执行操作。
2. 采用Lock接口的实现类如ReentrantLock并发量会有所提升，ReentrantLock内部采用segement，并发量为segement的个数。ReenterLock内部的AQS也是应用了CAS，1.8进行了更改。TODO

除了采用悲观锁这种独占锁来解决原子性之外，还可以采用乐观锁如CAS来解决。

Concurrent包下的类中，无论是**ReentrantLock内部的AQS，还是各种Atomic开头的原子类**，内部都应用到了`CAS`。

我们以最常见的i++为例，采用悲观锁的方式就是加锁独占处理：



```java
//采用synchronized加锁
public class Test {
    public volatile int i = 0;
    public synchronized void add() {
        i++;
    }
}

//采用ReentrantLock方式加锁

public class Test{
    public volatile int i = 0;
    public ReentrantLock lock = new ReentrantLock();
    public void add(){
        lock.lock();
        i++;
        lock.unlock();
    }
}

```

采用悲观锁的方式，在并发量上会有所降低，因此我们可以采用`CAS`的方式来保证操作的原子性。

JUC包中`Atomic`开头的原子类内部及采用CAS实现，我们可以使用`AtomicInteger`来保证i++操作的原子性。
```java
public class Test {
    public AtomicInteger i = 0;
    public void add() {
        i.getAndIncrement();
    }
}
```



#### 3. Atomic开头的原子类的内部实现

以`AtomicInteger`为例，我们来看`getAndIncrement`的内部：

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

`getAndIncrement`调用了Unsafe类中的`getAndAddInt`方法

再来查看`getAndAddInt`的源代码：

```java
//var1：传入的为this，即为AtomicInteger的实例对象
//var2：为传入的valueOffset`，其实就是记录value的偏移量的
//var4：即为需要增加的数值大小1
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

我们先来看传入的三个参数：`var1`，`var2`，`var4`：

`var1`：为传入的`this`，即为`AtomicInteger`的实例对象。

`var2`：为传入的`valueOffset`，其实就是记录`value`的偏移量的

`valueOffset`为`AtomicInteger`中的成员变量，源代码如下：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;
    
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;
    
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    
    private volatile int value;
    ...

```

在`AtomicInteger`数据定义的部分，我们可以看到，其实实际存储的值是放在`value`中的，除此之外我们还获取了`unsafe`实例，并且定义了`valueOffset`。再看到`static`块，懂类加载过程的都知道，`static`块的加载发生于类加载的时候，是最先初始化的，这时候我们调用`unsafe`的`objectFieldOffset`从`Atomic`类文件中获取`value`的偏移量，那么`valueOffset`其实就是记录`value`的偏移量的。

`var4`：即为需要增加的数值大小`1`。

------

下面我们继续回顾`getAndAddInt`的源码：

```java
//var1：传入的为this，即为AtomicInteger的实例对象
//var2：为传入的valueOffset`，其实就是记录value的偏移量的
//var4：即为需要增加的数值大小1
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

根据传入的`var1`和`var2`调用`this.getIntVolatile`方法获取`var5`的值，此处的`this`我们可以回顾前面在最开始的`getAndIncrement`方法中是通过`unsafe`实例调用`getAndAddInt`方法的，因此`getAndAddInt`是属于`unsafe`实例的，因此此处的`this`代表`unsafe`实例。

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

我们看`var5`获取的是什么，通过调用`unsafe`的`getIntVolatile(var1, var2)`，这是个`native`方法，具体实现到`JDK`源码里去看，其实就是获取`var1`中，`var2`偏移量处的值。`var1`就是`AtomicInteger`的实例对象，`var2`就是我们前面提到的`valueOffset`,这样我们就从内存里获取到现在`valueOffset`处的值了。

最后重点来了，`while`循环中的判断条件`compareAndSwapInt（var1, var2, var5, var5 + var4）`，我们将变量换成实际指代值，即为`compareAndSwapInt（obj, valueOffset, expect, update）`比较清楚，意思就是如果`obj`即`AtomicInteger`的实例对象中的`valueOffset`处的现在的`value`和`expect`（即`var5`，即`valueOffset`处一开始的`value`值）相等，就证明没有其他线程改变过这个变量，那么就更新它为`update`(即`var5+1`)，如果这一步的`CAS`没有成功，那就采用自旋的方式继续进行`CAS`操作，乍一看这也是两个步骤了啊，其实在`JNI`里是借助于一个`CPU`指令完成的。所以还是原子操作。

#### 4. CAS的问题

1. ABA问题

   CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。这就是CAS的ABA问题。 常见的解决思路是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么`A-B-A` 就会变成`1A-2B-3A`。 目前在JDK的atomic包里提供了一个类`AtomicStampedReference`来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

2. 自旋的循环时间问题

   上面我们说过如果CAS不成功，则会原地自旋，如果长时间自旋会给CPU带来非常大的执行开销。

【参考文献】：

1. https://time.geekbang.org/column/article/83682
2. <https://juejin.im/post/5a73cbbff265da4e807783f5>