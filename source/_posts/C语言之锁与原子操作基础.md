---
title: C语言之锁与原子操作基础
date: 2023-09-21 09:41:27
categories:
    - 学习笔记
    - C语言
tags:
    - 锁
    - 原子操作
abbrlink: 230921094127
---

## 常见锁介绍

在 C 语言中，常见的用于解决多线程访问数据的锁包括互斥锁、读写锁、条件变量、自旋锁、屏蔽等。

<!--more-->

1. 互斥锁（mutex）：互斥锁是一种**最基本的锁机制**，用于**保护共享资源**，防止多个线程同时访问和修改同一个资源。当一个线程持有了互斥锁后，其他线程需要等待该线程释放锁之后才能访问共享资源。

2. 读写锁（read-write lock）：读写锁是一种**特殊的锁机制**，它允许多个线程同时读取共享资源，但是**只允许一个线程写入共享资源**。这种锁可以提高读取操作的并发度，同时保证写入操作的正确性和一致性。如果一个线程获取了写锁，其他线程就必须等待它释放锁后才能继续访问；如果一个线程获取了读锁，其他线程也可以获取读锁并访问资源。

3. 条件变量（condition variable）：条件变量是一种**用于线程之间通信的同步机制**，它允许线程在某个条件成立时才能继续执行。通常与互斥锁一起使用，当条件变量不满足时，线程释放互斥锁并等待条件变量被激活（通过另一个线程来激活条件变量）；当条件变量满足时，通知线程重新获取互斥锁并继续执行。

4. 自旋锁（spinlock）：自旋锁是一种**忙等待锁机制**，当一个线程尝试获取锁时，如果锁已经被占用，它会一直循环等待直到锁被释放。**自旋锁适用于锁的持有时间很短的情况**，因为长时间占用CPU会影响系统性能。

5. 屏障（barrier）：屏障是一种**用于多线程协同的同步机制**，它允许多个线程在某个点上等待，直到所有线程都到达该点后再继续执行。屏障通常用于**一组线程**需要在某个点进行**同步操作**的情况，例如多线程排序算法。

> 需要根据具体的应用场景选择合适的锁。

## 互斥锁

互斥锁原理：互斥锁属于sleep-waiting类型的锁。例如，在一个双核的机器上有两个线程（线程A和线程B），它们分别运行在Core0和Core1上。假设线程A想要通过 `pthread_mutex_lock` 操作去得到一个临界区的锁，而此时这个锁正被线程B所持有，那么线程A就会被阻塞，Core0会在此时进行上下文切换（Context Switch）将线程A**置于等待队列中**，此时Core0就可以运行其它的任务而不必进行忙等待。

互斥锁的实现：通常使用了操作系统提供的原子操作或者硬件提供的锁机制，保证锁的正确性和高效性。

互斥锁有两种类型：递归锁和非递归锁。递归锁允许同一线程在不释放锁的情况下多次获取锁，而非递归锁不允许这种情况发生。

互斥锁使用场景：因互斥锁会引起线程的切换，效率较低；使用互斥锁会引起线程阻塞等待，不会一直占用着CPU。因此，当锁的内容较多、切换不频繁时，建议使用互斥锁。

互斥锁使用笔记：互斥锁的使用非常简单，主要包括以下几个步骤：
-   定义互斥锁变量，一般使用 `pthread_mutex_t` 类型；
-   在需要保护的代码段前调用 `pthread_mutex_lock` 函数获取锁；
-   在代码段执行完毕后调用 `pthread_mutex_unlock` 函数释放锁；
-   释放锁之后其他线程就可以获取锁并访问共享资源。

```c
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// 初始化互斥锁
pthread_mutex_init(&mutex, NULL);

// 在需要保护的代码块前加锁
pthread_mutex_lock(&mutex);
// 访问共享资源
// ...
// 访问完毕后解锁
pthread_mutex_unlock(&mutex);

// 在使用完互斥锁后销毁
pthread_mutex_destroy(&mutex);
```

## 读写锁

读写锁的实现：通常使用了计数器和互斥锁等机制，通过控制读写访问的次数和顺序来保证数据的正确性和一致性。

读写锁使用笔记：读写锁的使用也非常简单，主要包括以下几个步骤：
-   定义读写锁变量，一般使用 `pthread_rwlock_t` 类型；
-   在需要读取共享资源的代码段前调用 `pthread_rwlock_rdlock` 函数获取读锁；
-   在需要写入共享资源的代码段前调用 `pthread_rwlock_wrlock` 函数获取写锁；
-   在代码段执行完毕后调用 `pthread_rwlock_unlock` 函数释放锁；
-   释放锁之后其他线程就可以获取锁并访问共享资源。

```c
#include <pthread.h>

pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

// 初始化读写锁
pthread_rwlock_init(&rwlock, NULL);

// 在需要读取共享资源的代码块前加读锁
pthread_rwlock_rdlock(&rwlock);
// 读取共享资源
// ...
// 读取完毕后释放读锁
pthread_rwlock_unlock(&rwlock);

// 在需要写入共享资源的代码块前加写锁
pthread_rwlock_wrlock(&rwlock);
// 写入共享资源
// ...
// 写入完毕后释放写锁
pthread_rwlock_unlock(&rwlock);

// 在使用完读写锁后销毁
pthread_rwlock_destroy(&rwlock);
```

## 自旋锁

自旋锁原理：自旋锁属于busy-waiting类型的锁。例如，在一个双核的机器上有两个线程（线程A和线程B），它们分别运行在Core0和Core1上。如果线程A是使用 `pthread_spin_lock` 操作去请求锁，那么线程A就会一直在Core0上进行忙等待并**不停的进行**锁请求，直到得到这个锁为止。自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就**一直循环**在那里看是否该自旋锁的保持者已经释放了锁。

自旋锁使用场景：

- 自旋锁的作用是为了解决某项资源的**互斥使用**。因为自旋锁不会引起调用者睡眠，所以自旋锁的效率远高于互斥锁。因此，如果锁的内容较少、阻塞的时间较短，使用自旋锁比较好。

- 自旋锁在未获得锁的情况下，一直运行（自旋），占用着CPU，如果不能在很短的时间内获得锁，这无疑会使CPU效率降低。因此，要慎重使用自旋锁，**自旋锁只有在内核可抢占式或SMP的情况下才真正需要**。在单CPU且不可抢占式的内核下，自旋锁的操作为空操作。自旋锁适用于锁使用者保持锁时间比较短的情况下。

自旋锁使用笔记：

```c
#include <pthread.h>

// 定义自旋锁
pthread_spinlock_t spinlock = PTHREAD_PROCESS_PRIVATE;

// 初始化自旋锁
pthread_spin_init(&spinlock, PTHREAD_PROCESS_PRIVATE);

// 加锁，如果锁已被其他线程占用，则该函数会一直循环忙等待直到获取到锁
pthread_spin_lock(&spinlock);
// 访问共享资源
// ...
// 解锁，使其他线程可以获取锁并访问共享资源
pthread_spin_unlock(&spinlock);

// 销毁自旋锁
pthread_spin_destroy(&lock);
```

## 条件变量

条件变量（condition variable）：条件变量是一种**用于线程之间通信的同步机制**，它允许线程在某个条件成立时才能继续执行。通常与互斥锁一起使用，当条件变量不满足时，线程释放互斥锁并等待条件变量被激活（通过另一个线程来激活条件变量）；当条件变量满足时，通知线程重新获取互斥锁并继续执行。

条件变量使用笔记：

```c
#include <pthread.h>

// 定义互斥锁和定义条件变量
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// 初始化条件变量和自旋锁
pthread_cond_init(&cond, NULL);
pthread_mutex_init(&mutex, NULL);

// 加锁
pthread_mutex_lock(&mutex);

while (条件不满足预期条件) {
    // 等待条件变量通知
    pthread_cond_wait(&cond, &mutex);
}

// 条件满足，访问共享资源
// ...
// 解锁
pthread_mutex_unlock(&mutex);

// 在使用完条件变量后销毁
pthread_cond_destroy(&cond);
// 在使用完互斥锁后销毁
pthread_mutex_destroy(&mutex);

//----------------------------------------------

// 在其他线程中通知条件变量
// ...
// 加锁
pthread_mutex_lock(&mutex);
// 访问共享资源
// ...
// 解锁后再发送通知
pthread_mutex_unlock(&mutex);

// 条件满足，发送条件变量通知
pthread_cond_signal(&cond);
```

### 生产消费者同步示例代码

一个生产者、消费者同步的多线程示例代码：

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define BUFFER_SIZE (4)

int buffer[BUFFER_SIZE];
int count = 0;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void* producer(void* arg) {
    for (int i = 0; i < 5; i++) {
        pthread_mutex_lock(&mutex);
        // 预期条件是buffer不能满（当buffer满时为条件不满足预期）
        while (count == BUFFER_SIZE) {
            pthread_cond_wait(&cond, &mutex);
        }
        buffer[count++] = i;
        printf("Producer produced: %d\n", i);
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&cond);
    }
    return NULL;
}

void* consumer(void* arg) {
    for (int i = 0; i < 5; i++) {
        pthread_mutex_lock(&mutex);
        // 预期条件是buffer不能空（当buffer空时为条件不满足预期）
        while (count == 0) {
            pthread_cond_wait(&cond, &mutex);
        }
        int value = buffer[--count];
        printf("Consumer consumed: %d\n", value);
        pthread_mutex_unlock(&mutex);
        pthread_cond_signal(&cond);
    }
    return NULL;
}

int main() {
    pthread_t producer_thread, consumer_thread;

    pthread_create(&producer_thread, NULL, producer, NULL);
    pthread_create(&consumer_thread, NULL, consumer, NULL);

    pthread_join(producer_thread, NULL);
    pthread_join(consumer_thread, NULL);

    pthread_cond_destroy(&cond);
    pthread_mutex_destroy(&mutex);

    return 0;
}
```

在这个示例代码中，有一个容量为4的缓冲区，生产者线程负责往缓冲区中添加数据，消费者线程负责从缓冲区中取出数据。使用互斥锁和条件变量来保证生产者和消费者的同步。

生产者线程通过加锁后检查缓冲区是否已满，如果已满则等待条件变量通知，否则将数据添加到缓冲区，并发送条件变量通知消费者线程。消费者线程通过加锁后检查缓冲区是否为空，如果为空则等待条件变量通知，否则从缓冲区中取出数据，并发送条件变量通知生产者线程。

> 注意，生产者和消费者线程之间的同步是通过互斥锁和条件变量来实现的。互斥锁用于保护共享资源，条件变量用于线程间的通信和同步。

一种可能的运行结果：

```
Producer produced (from buffer index: 0): 0
Producer produced (from buffer index: 1): 1
Consumer consumed (from buffer index: 1): 1
Consumer consumed (from buffer index: 0): 0
Producer produced (from buffer index: 0): 2
Producer produced (from buffer index: 1): 3
Producer produced (from buffer index: 2): 4
Consumer consumed (from buffer index: 2): 4
Consumer consumed (from buffer index: 1): 3
Consumer consumed (from buffer index: 0): 2
[root@localhost del]#
[root@localhost del]# ./a.out
Producer produced (from buffer index: 0): 0
Producer produced (from buffer index: 1): 1
Producer produced (from buffer index: 2): 2
Producer produced (from buffer index: 3): 3
Consumer consumed (from buffer index: 3): 3
Consumer consumed (from buffer index: 2): 2
Consumer consumed (from buffer index: 1): 1
Consumer consumed (from buffer index: 0): 0
Producer produced (from buffer index: 0): 4
Consumer consumed (from buffer index: 0): 4
[root@localhost del]#
```

## 屏障

屏障（barrier）：屏障是一种**用于多线程协同的同步机制**，它允许多个线程在某个点上等待，直到所有线程都到达该点后再继续执行。屏障通常用于**一组线程**需要在某个点进行**同步操作**的情况，例如多线程排序算法。

```c
#include <pthread.h>

// 定义屏障
pthread_barrier_t barrier;
int thread_nums = 5;

// 初始化屏障
pthread_barrier_init(&barrier, NULL, thread_nums);

// 在每个线程中执行以下代码
pthread_barrier_wait(&barrier); // 等待屏障

// 所有线程都到达屏障后，继续执行以下代码
// ...

// 在使用完屏障后销毁
pthread_barrier_destroy(&barrier);
```

### 屏蔽实现线程同步示例代码

下面是一个完整的示例代码，演示了如何使用 `pthread` 库中的屏障实现线程同步：

```c
#include <stdio.h>
#include <pthread.h>

// 定义屏障
pthread_barrier_t barrier;
int thread_nums = 5;

// 线程函数
void* thread_func(void* arg) {
    // 线程执行的一些操作...
    printf("Thread %ld operation\n", (long)arg);

    // 等待屏障
    pthread_barrier_wait(&barrier);

    // 所有线程都到达屏障后，继续执行以下代码
    printf("Thread %ld continues after the barrier\n", (long)arg);

    // 线程执行的其他操作...

    return NULL;
}

int main() {
    pthread_t threads[thread_nums];

    // 初始化屏障
    pthread_barrier_init(&barrier, NULL, thread_nums);

    // 创建线程
    for (long i = 0; i < thread_nums; i++) {
        pthread_create(&threads[i], NULL, thread_func, (void*)i);
    }

    // 等待线程结束
    for (int i = 0; i < thread_nums; i++) {
        pthread_join(threads[i], NULL);
    }

    // 销毁屏障
    pthread_barrier_destroy(&barrier);

    return 0;
}
```

在这个示例代码中，我们定义了一个屏障 `pthread_barrier_t` 和一个线程数 `thread_nums`。在主函数中，

-   首先调用 `pthread_barrier_init` 函数初始化屏障。
-   然后，创建指定数量的线程，并通过 `pthread_create` 函数将线程函数 `thread_func` 分配给每个线程。
-   在线程函数中，线程首先执行一些操作，然后调用 `pthread_barrier_wait` 函数等待屏障。**当所有线程都到达屏障后，屏障解除，所有线程继续执行后续的代码**。
-   最后，我们使用 `pthread_join` 等待所有线程结束，并使用 `pthread_barrier_destroy` 销毁屏障。

## 原子操作

所谓原子操作，就是该操作绝不会在执行完毕前被任何其他任务或事件打断，也就说，它是**最小的执行单位**，不可能有比它更小的执行单位。因此这里的原子实际是使用了物理学里的物质微粒的概念。

原子操作需要硬件的支持，因此是架构相关的，其API和原子类型的定义都定义在内核源码树的 `include/asm/atomic.h` 文件中，它们**都使用汇编语言实现，因为C语言并不能实现这样的操作**。

原子操作主要用于实现资源计数，很多引用计数（Reference Count, refcnt）就是通过原子操作实现的。

## 总结分析

互斥锁（Mutex lock），sleep-waiting类型的锁：与自旋锁相比它需要消耗大量的系统资源来建立锁；随后当线程被阻塞等待时，线程的调度状态被修改，并且线程被加入等待线程队列；最后当锁可用时，在获取锁之前，线程会被从等待队列取出并更改其调度状态；但是在线程被阻塞期间，它不消耗CPU资源。

互斥锁适用于那些可能会阻塞很长时间的场景：

- 临界区有IO操作
- 临界区代码复杂或者循环量大
- 临界区竞争非常激烈
- 单核处理器

自旋锁（Spin lock），busy-waiting类型的锁：对于自旋锁来说，它只需要消耗很少的资源来建立锁；随后当线程被阻塞时，它就会一直重复检查看锁是否可用了，也就是说当自旋锁处于等待状态时它会一直消耗CPU时间。  

自旋锁适用于那些仅需要阻塞很短时间的场景。

