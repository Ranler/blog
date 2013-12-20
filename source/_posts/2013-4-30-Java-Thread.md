layout: post
title: Java Thread 
date: 2013-4-30
categories: JVM
---

## Wait Notify and Interrupt

Java Thread可以执行在多核（并行）或者时间份片的单核处理器上执行（并发）。
一般接口是：

```java
public interface Runable {
	public abstract void run();
}

public class Thread implements Runable {
	public synchronized void start();
	public void run() {...}
	public void interrupt() {...}
	...
}
```

每个Java对象都关联一个**等待集合(wait set)**，集合中的元素是**线程**。
Java通过以下三个方法对等待集合进行原子操作：

- Object.wait()
- Object.notify()
- Object.notifyAll()

等待集合也受线程打断状态影响:

- Thread.interrupt()


### wait

wait动作在调用以下方法时发生：

```
wait()
wait(long millisecs)
wait(long millisecs, int nanosecs)
```

如果调用成功，wait行为将发生阻塞，直到被唤醒。
如果调用失败，将抛出异常。

例如，线程t调用对象m的wait()，并且t在m上获得锁的个数为n。
接下来可能发生以下几种情况：

- 如果n=0,抛出`IllegalMonitorStateException`。这是因为t尚未获得m的锁。
- 如果wait的参数错误，抛出`IllegalArgumentException`。
- 如果t被打断，抛出`InterruptedException`，t的打断状态设为false。
- 否则，以下流程将顺序执行：
  1. t添加到m的等待集合，并且t在m上执行n次unlock动作。
  t被阻塞，直到遇到以下任意动作才被移出等待集合。
	 - m的notify动作被执行且t被选中移出等待集合
	 - m的notifyAll动作被执行
	 - t的interrupt动作被执行
	 - wait的时间到
  2. t在m上执行n次lock动作。
  3. 如果t因为interrupt动作被移出等待集合，那么t的打断状态被设为false，
  并抛出InterruptedException。


### Notification

唤醒动作在调用以下方法时发生：

```
notify()
notifyAll()
```

当调用对象m的`notify()`时，从m的等待集合中移出一个线程t，并使t在m上执行n次lock动作。
具体的从等待集合中选出那个线程是没有保证的。

当调用对象m的`notify()`时，从m的等待集合中移出所有线程。
但是**只有一个线程**能在m上执行n次lock动作，其余线程仍将阻塞，等待锁的释放。


### Interruptions

打断动作在调用以下方法时发生：

```
Thread.interrupt()
ThreadGroup.interrupt()
```

当线程t的interrupt()被调用时，t的打断状态被设为true。
如果t在某个对象的等待集合中，则移出t，然后t竞争对象的锁，打断状态设为false，
最后抛出`InterruptedException`。

调用`Thread.interrupted()`将获得线程的打断状态，并清除其打断状态为false。
调用`Thread.isInterrupted()`将直接获得线程的打断状态，并不清除。


### 总结

如果线程t在等待时同时收到唤醒动作和打断动作，那么线程t可能：

- 从wait正常返回，但仍有一个未决的interrupt(也就是说`Thread.interrupted()`将返回true)
- 从wait抛出`InterruptedException`

在线程正常返回时，可能打断状态可能未重置。

同样，唤醒动作也不会因Interrupt而丢失。只要调用`notify()`，必有一个线程正常wait返回。


### Reference

- JLS7, Chapter17





