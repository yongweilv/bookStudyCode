# 4.锁的优化及注意事项
## 4.1 有助于提高锁性能的几点建议
#### <1>减少锁持有的时间
    只在必要的地方进行加锁
例如:<br>
```
public synchronized void syncMethod(){
    othercode1();
    mutexMethod();
    othercode2();
}
```

替换为
```
public void syncMethod2(){
    othercode1();
    synchronized(this){
        mutexMethod();
    }
    othercode2();
}
```
#### <2>减小锁粒度
    典型场景是`ConcurrentHashMap` 对于`HashMap`来说，最重要的两个方法就是`get()`和`set()` 一种最自然的想法就是，
对整个`HashMap`加锁从而得到一个线程安全的对象，但是这样做，加锁粒度太大。对于`ConcurrentHashMap`类，它内部进一步细分了
若干个小的`HashMap`，称之为段（`SEGMENT`）。在默认情况下，一个`ConcurrentHashMap`类可以被细分为16个段.<br>
如果需要在`ConcurrentHashMap`类中增加一个新的表项，并不是将整个`HashMap`加锁，而是首先根据`hascode`得到该表项应该被
存放在哪个段红，然后对该段加锁，并完成`put()`方法.<br>
  但是，减小锁粒度会带来一个新的问题，即当系统需要取得全局锁时，其消耗的资源会比较多.比如执行`size()`方法时，会对所有段进行加锁
然后算size，然后释放锁.<br>
事实上，`ConcurrentHashMap`执行`size()`时，首先会进行无锁尝试获取，如果失败才会进行上诉操作.

#### <3>用读写分离锁来替换独占锁
    使用读写分离锁`ReadWriteLock`
  
#### <4>锁分离
    如果将读写锁的思想进一步延伸，就是锁分离。
  
#### <4>锁粗化
    通常情况下，为了保证多线程间的有效并发，会要求每个线程持有的锁的时间尽量短，即在使用完公共资源后，应该立即释放锁。只有这样，等待在这个
锁上的其他线程才能尽早地获得资源执行任务。但是，凡事都有一个度，如果对同一个锁不停地进行请求、同步和释放，其本身也会消耗系统宝贵的资源，
反而不利于性能的优化。<br>
  为此，虚拟机在遇到一连串连续的对同一个锁不断进行请求和释放的操作时，便会把所有的锁操作整合成对锁的一次请求，从而减少对锁的请求同步次数，
这个操作叫做锁的粗化.
  
## 4.2 JAVA虚拟机对锁优化所做的努力
#### <1>偏向锁
    锁偏向是一种针对加锁操作的优化手段。它的核心思想是:如果一个线程获得了锁，那么锁就进入偏向模式。当这个线程再次请求锁时，无须再做任何同步操作,
这样就节省了大量有关锁申请的操作，从而提高了程序性能.因此，对于几乎没有锁竞争的场合，偏向所有比较好的优化效果,因为连续多次极有可能是同一个线程
请求相同的锁。而对于锁竞争激烈的场合,其效果不佳.<br>
  使用Java虚拟机参数`-XX:+UseBiaseLocking`可以开启偏向锁
#### <2>轻量级锁
    如果偏向锁失败，那么虚拟机并不会立即挂起线程，它还会使用一种称为轻量级锁的优化手段。轻量级锁的操作也很方便，它只是简单地将对象头部作为指针
指向持有锁的线程堆栈内部，来判断一个线程是否持有对象锁。如果线程获得轻量级锁成功，则可以顺利进入临界区。如果轻量级锁加锁失败，则表示其他线程
抢先争夺到了锁，那么当前线程的锁就会膨胀为`重量级锁`
#### <3>自旋锁
    锁膨胀后，为了避免线程真实的在操作系统层面挂起，虚拟机还会做最后的努力---自旋锁。虚拟机会让当前线程做几个空循环，经过若干次循环后，如果可以
得到锁，那么就顺利进入临界区。如果不能获得锁，才会真的姜线程在操作系统层面挂起.
#### <4>锁消除
    Java虚拟机在JIT编译时，会去除不可能存在共享资源竞争的锁。
  
## 4.3 ThreadLocal
    这是一个线程的局部变量。也就是说，只有当前线程可以访问。
  注意：为每一个线程分配不同的对象,需要在应用层面保证.ThreadLocal只起到了简单的容器作用.

## 4.4 无锁
    无锁的策略使用的叫做比较交换（CAS, Compare And Swap)的技术来鉴别线程冲突一但检测到冲突产生，就重试当前操作直到没有冲突为止
  CAS(V,E,N) V代表要更新的变量  E代表该变量现在的预期值 N代表要更新的新值  当且仅当V的值等于E时，才将V值改变为N
  
## 4.5 死锁
    通俗地讲，死锁就是两个或多个线程相互占用对方需要的资源，而都不进行释放,导致彼此之间相互等待对方释放资源，产生了无限制等待的现象。
哲学家进餐问题,产生死锁
```
package chapter4;
public class DeadLock extends Thread {

    protected Object tool;
    static Object fork1 = new Object();
    static Object fork2 = new Object();

    public DeadLock(Object obj){
        this.tool = obj;
        if(tool  == fork1){
            this.setName("哲学家A");
        }
        if(tool == fork2){
            this.setName("哲学家B");
        }
    }

    @Override
    public void run() {
        if(tool==fork1){
            synchronized (fork1){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (fork2){
                    System.out.println("哲学家A开始吃饭了");
                }
            }
        }
        if(tool==fork2){
            synchronized (fork2){
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (fork1){
                    System.out.println("哲学家B开始吃饭了");
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        DeadLock deadlockA = new DeadLock(fork1);
        DeadLock deadlockB = new DeadLock(fork2);
        deadlockA.start();
        deadlockB.start();
        Thread.sleep(1000);
    }
}
```