---
title: Java并发编程实战
date: 2020-11-22 14:57:25
tags: Code
---

# Java并发编程实战

### 对象的共享

###### 可见性

~~~java
public class NoVisibility {
    private static boolean ready;
    private static int number;

    private static class ReaderThread extends Thread{
        public void run(){
            while(!ready)
                Thread.yield();
            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new ReaderThread().start();
        number = 42;
        ready = true;
    }
}
~~~

在上述代码中，Novisibility有可能会一直循环下去，因为读线程有可能会一直看不到ready的值。而另一种现象是，NoVisibility可能会输出0.这就是线程的不可见性。

###### 失效数据

当读线程在查看ready变量时，有可能会获得该变量的一个失效值。

~~~java
public class MutableInteger{
    private int value;
    
    public int get() {return value;}
    public void set(int value) { this.value = value; }
}
~~~

上述代码会出现失效值问题，当某个线程调用了set，那么另一个线程可能看到更新后的value值。，也可能看不到。

得到的失效值至少不是随机的，这种安全性保证被称为最低安全性。

### 解决方法

1. 加锁

内置锁可以保证线程以一种可预测的方式查看一个线程的执行情况。所以，可以进一步理解线程之间的同步，就是为了确保某个线程写入该变量的值对于其他线程是可见的。

2. Volatile变量

Java语言提供了一种稍弱的同步机制，及volatile变量，用来确保变量的更新操作通知到其他线程。当该变量声明为volatile类型后，编译器与运行时会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存一起重排序。volatile变量是一种比sychronized关键字更轻量级的同步机制。

并不建议过度依赖volatile变量提供的可见性。会使锁的代码更脆弱，也更难以理解。

volatile变量通常用做某个操作完成、发生中断或者状态的标志。

### 发布与逸出

发布(Publish)一个对象的意思是指，使对象能够在当前作用域之外的代码中使用。例如，将一个指向该对象的引用保存到其他代码可以访问的地方，或者在某一个非私有方法中返回该引用。当某个不应该被发布的对象被发布时，这种情况被称之为逸出（Escape）。

当发布某个对象时，可能会间接地发布其他对象。如果发布了一个列表或集合一类的引用，同时也会发布其中的对象。同样，如果从非私有方法中返回一个引用，那么同样会发布返回的对象。

~~~java
public class UnsafeStates {
    private String[] states = new String[]{"AK","AL"};

    public String[] getStates() { return states; }

    public static void main(String[] args) {
        UnsafeStates us = new UnsafeStates();
        us.getStates()[1] = "1";
        System.out.println(Arrays.toString(us.getStates()));
    }
}
~~~

输出值是：[AK, 1]

还用一种发布对象或其内部状态的机制就是发布一个内部的类实例。当ThisEscape发布EventListener时，也隐含发布了ThisEscape本身。

~~~java
public{
    public ThisEscape(EventSource source){
        source.registerListener({
            new EventListener(){
                public void onEvent(Event e){
                    doSomething(e);
                    this.getClass();
                }
            }
        });
    }
}
~~~

当内部的EventListener实例发布时，在外部封装的ThisEscape实例也逸出了。当在构造函数显式或隐式创建一个线程时，this可被新线程看到。



### 线程封闭

如果仅在单线程内访问数据，那么这种技术称为线程。在Swing中大量使用了线程封闭技术。JDBC也包含这个技术。

1.  Ad-hoc线程封闭

指的是线程封闭性完全由程序来承担。

2. 栈封闭

3. ThreadLocal类

   这个类使得线程中的某些值与保存值的对象联系起来。

