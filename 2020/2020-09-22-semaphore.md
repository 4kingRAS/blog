Semaphore 笔记
---

1965年荷兰Dijkstra提出的信号量（Semaphores）是一种卓有成效的进程同步工具，在长期的应用中，得到了很大的发展，从整型信号量经过记录型信号量，进而发展为“信号量集”机制。如果信号量是一个任意的整数，通常被称为计数信号量（Counting semaphore），或一般信号量（general semaphore）；如果信号量只有二进制的0或1，称为二进制信号量（binary semaphore）。在linux系统中，二进制信号量（binary semaphore）又称互斥锁（Mutex）。

* 信号量就是OS提供的管理公有资源的有效手段。
* 信号量代表可用资源实体的数量。

---
## 两个原子操作

P操作 wait: `Decrements the value of semaphore variable by 1. If the new value of the semaphore variable is negative, the process executing wait is blocked (i.e., added to the semaphore's queue). Otherwise, the process continues execution, having used a unit of the resource.`

V操作 signal: `Increments the value of semaphore variable by 1. After the increment, if the pre-increment value was negative (meaning there are processes waiting for a resource), it transfers a blocked process from the semaphore's waiting queue to the ready queue.`

## Java concurrent.Semaphore

Semaphore管理着一组许可（permit）,许可的初始数量可以通过构造函数设定，操作时首先要获取到许可，才能进行操作，操作完成后需要释放许可。如果没有获取许可，则阻塞到有许可被释放。

*如果初始化了一个许可为`1`的Semaphore，那么就相当于一个不可重入的互斥锁（Mutex）。*

```java
    public void run() {
            try {
                semaphore.acquire();
                
                // critical section

                semaphore.release();
            } catch (InterruptedException e) {
            }
    }
```

## 公平机制

`boolean hasQueuedThreads()  // 如果此信号量的公平设置为 true，则返回 true。`

"公平信号量"和"非公平信号量"的释放信号量的机制是一样的！不同的是它们获取信号量的机制：线程在尝试获取信号量许可时，对于公平信号量而言，如果当前线程不在CLH队列的头部，则排队等候；而对于非公平信号量而言，无论当前线程是不是在CLH队列的头部，它都会直接获取信号量。该差异具体的体现在，它们的tryAcquireShared()函数的实现不同。

## 应用

个人理解，对于一组资源的排队使用（不保证使用顺序）很适合Semaphore


## 底层AQS

doug lea 大神的AQS机制，可以写一本书了。
