# JMM

## volatile

- 修饰成员变量和静态成员变量
- 可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值
- 线程操作volatile变量都是直接操作主存
- 可以禁止指令重排序
- 保证可见性和有序性，不能保证原子性，适合一个写线程多个读线程的情况

## Synchronized

- 保证原子性、可见性，不能保证有序性
- println方法是由Synchronized锁住的，Synchronized可以防止当前的线程从高速缓冲流中获取值

## 指令重排序

- 双重检查锁的单例模式会因为指令重排序而产生错误：一个线程获取到锁时加载对象，加载对象分四步，先new，再dup，再invokespecial，再putstatic，其中第三步和第四步可能会因为指令重排序而调换顺序，导致返回一个还没完成初始化的对象，这时当另一个线程在锁外面获取对象时，就会获取到这个还没有完成初始化的对象，当它调用这个对象的成员变量时，就会出错。改进方案：在单例对象上加一个volatile。

## happens-before

1. 线程解锁m之前对变量的写，对于接下来对m加锁的其他线程对该变量的读可见。可以理解为，一个线程中如果加了synchronized，则在synchronized代码块中的数据都是可见的，其他线程的synchronized代码块中的变量都是最新更改的变量，前提是锁的是同一个对象。

   ```java
   static int x;
   static Object m = new Object();
   new Thread(()->{ 
       synchronized(m) {
           x = 10; 
       } 
   },"t1").start();
   new Thread(()->{ 
       synchronized(m) {
           System.out.println(x);//x=10
       }
   },"t2").start();
   ```

2. 线程对volatile变量的写，对接下来其他线程对该变量的读可见。可以理解为volatile修饰的变量所有线程都能读到它的最新数据值，每次访问都是从主线程中读取值，不会被JIT优化到高速缓存区中。

   ```java
   volatile static int x;
   new Thread(()->{
       x = 10; 
   },"t1").start(); 
   new Thread(()->{ 
       System.out.println(x); 
   },"t2").start();
   ```

3. 线程start前对变量的写，对该线程开始后对该变量的读可见

   ```java
   static int x; 
   x = 10;
   new Thread(()->{
       System.out.println(x); 
   },"t2").start();
   ```

4. 线程结束前对变量的写，对其他线程得知它结束后的读可见

   ```java
   static int x;
   Thread t1 = new Thread(()->{ 
       x = 10; },"t1");
   t1.start(); 
   t1.join();//此处等待t1线程结束
   System.out.println(x);
   ```

5. 线程t1打断t2（interrupt）前对变量的写，对于其他线程得知t2被打断后对变量的读可见

   ```java
   static int x; 
   public static void main(String[] args) {
       Thread t2 = new Thread(()->{ 
           while(true) { 
               if(Thread.currentThread().isInterrupted()) { 
                   System.out.println(x);
                   break;
               }
           }
       },"t2");
       t2.start();
       new Thread(()->{ 
           try {
               Thread.sleep(1000); 
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           x = 10; 
           t2.interrupt();
       },"t1").start();
       while(!t2.isInterrupted()) {
           Thread.yield();
       }
       System.out.println(x);   
   }
   ```

6. 对成员变量或静态变量默认值（0，false，null）的写，对其他线程对该变量的读可见

7. 具有传递性，如果x hb->y并且y hb->z那么有x hb->z

## CAS与原子类

```java
// 需要不断尝试 
while(true) { 
    int 旧值 = 共享变量 ; // 比如拿到了当前值 0 
    int 结果 = 旧值 + 1; // 在旧值 0 的基础上增加 1 ，正确结果是 1 
    /*这时候如果别的线程把共享变量改成了 5，本线程的正确结果 1 就作废了，这时候 compareAndSwap 返回 false，重新尝试，直到： compareAndSwap 返回 true，表示我本线程做修改的同时，别的线程没有干扰 */
    if( compareAndSwap ( 旧值, 共享变量 )) {
        // 成功，退出循环 
    }
}
```

- 修改值的时候，不停的去查看一开始获得的共享变量的值与此时刻的共享变量的值是否一致，如果不一致的话重新执行自增或自减修改数据，直到一致才返回结果
- 共享变量需要使用volatile修饰
- 适用于竞争不激烈、多核CPU的场景下，因为需要不停的尝试会一直抢占CPU
- 因为没有使用Synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一
- 底层依赖于一个Unsafe类来直接调用操作系统底层的CAS指令

## 轻量级锁

- 如果一个对象虽然有多线程访问，但多线程访问时间是错开的（也就是没有竞争），那么可以使用轻量级锁来优化