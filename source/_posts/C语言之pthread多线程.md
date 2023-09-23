---
title: C语言之pthread多线程
date: 2023-09-11 15:13:34
categories:
    - 学习笔记
    - C语言
tags:
    - 多线程
    - 进程与线程
abbrlink: 230911151334
---

## 多线程

### 进程与线程

**进程**：是一个具有一定独立功能的程序在一个数据集上的一次动态执行的过程，是操作系统进行资源分配和调度的一个独立单元，是应用程序运行的载体。进程一般由程序、数据集合和进程控制块三部分组成：

-   程序：描述进程的功能，控制进程的指令集（写论文的目的和手段）

-   数据集：程序在执行时所需要的数据和工作区（写论文的材料）

-   进程控制块：跟踪每个进程的状态，操作系统会为进程保留进程列表（写作者）

**线程**：线程是程序执行中的一个单一的**顺序控制流程**，是程序执行流的最小单元，是处理器调度和分派的基本单位。

-   一个进程至少有一个线程，一个进程也可以有多个线程。（一个父亲可以有一个、多个孩子）

-   各个线程之间共享程序的内存空间，即所在进程的内存空间。（多个孩子共享一个家庭空间）

-   一个标准的线程由线程 ID、当前指令指针 PC、寄存器和堆栈组成。（每个孩子有其自身的成长轨迹）

<!-- more -->

**进程与线程的区别**：

-   线程是程序执行的最小单位，而进程是操作系统分配资源的最小单位。

-   一个进程由一个或多个线程组成，线程是一个进程中代码的不同执行路线。

-   进程之间相互独立，但**同一进程下的各个线程之间共享程序的内存空间**（代码段、数据集、堆等）以及一些**进程级的资源**（如打开文件和信号等），某进程内的线程在其他进程中不可见。

-   **线程上下文切换比进程上下文切换要快得多**。

### 上下文切换

#### 时间片

多任务系统往往需要同时执行多道作业。作业数往往大于机器的CPU数，然而一颗CPU同时只能执行一项任务，如何让用户感觉这些任务正在同时进行呢? 操作系统的设计者巧妙地利用了**时间片轮转的方式**。

**时间片是 CPU 分配给各个任务（线程）的时间**。

> 思考：单核CPU为何也支持多线程呢？
> 
> 虽然单核CPU只有一个物理处理单元，但它可以**通过时间分片的方式支持多线程**。在单核CPU中，操作系统通过时间片轮转算法将CPU时间划分为多个时间片段，每个时间片段分配给一个线程执行。当一个线程的时间片用完后，操作系统会暂停该线程的执行，并切换到下一个线程继续执行。这种切换是非常快速的，以至于我们感觉多个线程在同时执行。
> 
> 需要注意的是，在单核CPU上并发执行的多线程是通过时间片轮转调度实现的，每个线程在任意给定的时间点上**只能有一个**线程在执行。而在多核CPU上，可以实现真正的并行执行，每个核心可以同时执行一个线程，从而提高并发性能。

#### 上下文切换

**线程上下文**：是指某一时间点 CPU **寄存器和程序计数器的内容**，CPU 通过时间片分配算法来循环执行任务（线程），因为时间片非常短，所以 CPU 通过不停地切换线程执行。

换言之，单 CPU 这么频繁，多核 CPU 一定程度上可以减少上下文切换。

**上下文切换**：CPU 切换前把当前任务的状态保存下来（以便下次切换回这个任务时可以再次加载这个任务的状态），然后加载下一任务的状态并执行。**任务的状态保存及再加载**，这段过程就叫做上下文切换。

### 多线程编程

-   多进程模式：启动多个进程，每个进程虽然只有一个线程，但是多个进程可以一块执行多个任务。

-   **多线程模式**：启动一个进程，在一个进程内启动多个线程，多个线程一起执行多个任务。

-   多进程 + 多线程模式：启动多个进程，每个进程再启动多个线程，这样同时执行的任务就更多了。

其实创建线程之后，线程并不是始终保持一个状态的，其状态大概如下：

1.  New 创建
2.  Runnable 就绪，等待调度
3.  Running 运行
4.  Blocked 阻塞，阻塞可能在 Wait / Locked / Sleeping 阶段
5.  Dead 消亡

线程有着不同的状态，也有不同的类型。大致可分为：

-   主线程：主线程是程序启动时自动创建的线程，它负责执行程序的主要逻辑。主线程通常负责处理用户交互、调度其他线程的创建和管理等任务。

-   子线程：子线程是由主线程创建的额外线程，用于执行并发任务。子线程可以并行地执行任务，从而提高程序的效率和响应性。

-   守护线程（后台线程）：守护线程是一种特殊类型的线程，它在后台运行，**不会阻止程序的退出**。当所有的非守护线程都退出时，守护线程也会自动结束。守护线程通常用于执行一些后台任务，如日志记录、定时任务等。

-   前台线程：前台线程是与守护线程相对的概念，它是指**会阻止程序退出的线程**。当所有的前台线程都退出时，程序才会结束。

### pthread多线程

POSIX 线程（Pthreads）是一套标准的线程API，用于多线程编程。该库定义了一组C语言函数，允许程序员创建和管理多个线程，并提供同步和互斥机制，以确保线程之间的正确协调。

Pthreads库是POSIX标准的一部分，其全称是“Portable Operating System Interface”，旨在为Unix-like操作系统（如Linux、FreeBSD、Mac OS X等）提供一致的接口。由于该标准的广泛接受和实现，因此Pthreads库现在在许多不同的平台上都可用。

Pthreads库的一个优点是它允许程序员创建轻量级线程（LWP），这些线程比进程更轻量级，因此在创建和销毁它们时所需的开销较小。此外，由于它是标准的POSIX接口，因此Pthreads库可在不同的操作系统上重用，从而提高了代码的可移植性。

pthread库需要头文件：`pthread.h`

gcc 编译链接参数：`lpthread`

```shell
gcc ./demo.c -o demo -lpthread
```

#### 创建线程相关

##### pthread_create

`pthread_create`是一个用于创建线程的函数，它的原型如下：

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);

// pthread_t类型的原型定义
typedef unsigned long int pthread_t;
```

该函数接收四个参数，分别是：

1.  `thread`：一个指向线程标识符的指针，由函数创建并返回。

2.  `attr`：一个指向线程属性的指针，用于设置线程的属性。如果不需要设置属性，传入`NULL`即可。

3.  `start_routine`：线程执行的函数指针，该函数接受一个 `void *` 类型的参数，并返回一个 `void *` 类型的值。

4.  `arg`：传递给线程执行函数的参数，如果有多个参数，可以传递一个指向参数结构体的指针。

当调用`pthread_create`函数时，它会创建一个新的线程，并将其标识符存储在`thread`指针中。新线程的执行将从`start_routine`函数开始，`arg`参数将作为`start_routine`的参数传递给它。

创建线程时，可以选择使用默认线程属性，也可以使用`pthread_attr_t`结构体来设置一些属性，例如线程的调度策略、栈大小、优先级等等。如果不需要设置属性，可以将`attr`参数设置为`NULL`。

`pthread_create`函数成功时返回0，否则返回一个错误码。如果返回非零错误码，可以使用`perror`函数或`strerror`函数打印出错误信息。

##### pthread_self

`pthread_self`函数返回调用它的线程的线程ID：

```c
pthread_t pthread_self(void);
```

##### pthread_equal

`pthread_equal`函数通过线程ID比较线程是否相等，如果两个线程相等，返回非0值，如果不相等，返回0：

```c
int pthread_equal(pthread_t t1, pthread_t t2);
```

##### pthread_detach

`pthread_detach` 函数用于将指定的线程分离出去，所谓分离出去就是指**主线程再不需要**通过 `pthread_join` 等方式，等待该线程的结束并回收其线程控制块（TCB）的资源，**被分离的线程结束后由操作系统负责其资源的回收**。

```c
int pthread_detach(pthread_t thread);
```

其中，`thread` 参数是要分离的线程的标识符，返回值为 0 表示成功，非 0 值表示出错。

需要注意的是，如果一个线程被分离了，就不能再对它调用 `pthread_join` 函数，否则会出错。因此，在调用 `pthread_detach` 函数之前，必须确保不会再调用 `pthread_join` 函数。

一般来说，主线程是要负责创建出来的子线程的资源回收工作的：

-   如果主线程先于子线程退出，并且子线程没有设置为分离状态，那么子线程结束后其资源是无法得到回收的，会造成资源浪费和系统臃肿。

-   如果主线程先于子线程退出，但是子线程是分离状态，那么子线程退出的时候操作系统会自动回收其资源。

分离线程并不是分离了之后，就跟主线程没有一点关系了。主线程退出了，分离线程还是一样退出，只是分离线程的资源是由系统回收的。

#### 终止线程相关

终止线程的三种方式：

1. 线程从启动例程返回，返回值就是线程的退出码；

2. 线程可以被同一进程中的其他线程取消（通过`pthread_cancel()`）；

3. 线程自身调用`pthread_ exit()`函数终止。

##### pthread_cancel

`pthread_cancel` 函数是一个用于取消 POSIX 线程的函数。该函数向目标线程发送一个取消请求，如果该线程允许取消，则会在处理该请求时终止该线程的执行。

> 线程可以设置为允许取消（默认情况下）或者禁止取消。如果线程允许取消，它将在收到取消请求后**尽快取消**，并执行一些清理工作；如果线程禁止取消，它将继续运行，直到完成其任务或者显式地调用 `pthread_exit` 函数。

```c
int pthread_cancel(pthread_t thread);
```

`pthread_cancel` 函数有以下两种用法：

1.  `int pthread_cancel(pthread_t thread);` 此用法向线程 ID 为 `thread` 的线程发送取消请求。如果请求成功发送，则返回 0。如果线程 ID 无效或请求无法发送，则返回一个非零错误码。

2.  `void pthread_testcancel(void);` 此用法可以在线程执行期间调用，用于测试是否有取消请求已经发送给该线程。如果是，则在线程执行期间发生取消动作，该线程的执行将立即停止。

需要注意的是，`pthread_cancel` 函数并不保证能够成功地取消目标线程的执行。当目标线程正在执行某些不可取消的操作（例如某些系统调用）时，取消请求可能会被暂时挂起，直到目标线程离开这些操作为止。另外，使用 `pthread_cancel` 函数需要注意线程同步问题，避免出现死锁等问题。

总的来说，`pthread_cancel` 函数可以用于线程的优雅终止，但是需要谨慎使用，避免出现意外的问题。

```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

void* thread_func(void* arg) {
    while (1) {
        printf("Thread is running...\n");
        sleep(1);
        // 在循环中调用 pthread_testcancel 函数检查是否有取消请求
        pthread_testcancel();
    }
    return NULL;
}

int main() {
    pthread_t thread_id;
    // 创建一个线程并启动它
    pthread_create(&thread_id, NULL, &thread_func, NULL);
    sleep(5);
    // 在主线程中调用 pthread_cancel 函数向子线程发送取消请求
    pthread_cancel(thread_id);
    // 等待子线程结束
    pthread_join(thread_id, NULL);
    printf("Thread has been canceled.\n");
    return 0;
}
```

```text
root@fw:~/w26/ccodes# ./demo
Thread is running...
Thread is running...
Thread is running...
Thread is running...
Thread is running...
Thread has been canceled.
root@fw:~/w26/ccodes#
```

在上面的代码中，首先创建了一个子线程并启动它，在子线程的循环中不断输出信息，同时在循环中调用 `pthread_testcancel` 函数检查是否有取消请求。在主线程中等待 5 秒钟后，调用 `pthread_cancel` 函数向子线程发送取消请求，然后等待子线程结束并输出一条信息表示子线程已经被成功取消。

**调用与不调用`pthread_testcancel()`的区别**：

1.  调用 `pthread_testcancel` 函数可以让线程在循环或其他*可取消的操作中*主动检查是否有取消请求，并在检测到取消请求时*及时终止*线程的执行。这样可以增加线程的可靠性，**确保线程在可以取消的时候及时响应取消请求**。

2.  如果不调用 `pthread_testcancel` 函数，线程可能会在某些不可取消的操作中被阻塞（例如在 `sleep` 等待时），无法及时响应取消请求，导致取消请求被暂时挂起。

3.  因此，为了保证线程能够及时响应取消请求，通常建议在线程的循环或其他可取消的操作中调用 `pthread_testcancel` 函数，以便让线程在合适的时机进行取消。但是需要注意的是，在使用 `pthread_testcancel` 函数时，必须确保线程的同步操作是线程安全的，否则可能会导致程序出现不可预期的错误。

4.  当然，在某些情况下，如果线程不会进入可取消的状态，或者线程在处理临界区时不能被取消，那么调用 `pthread_testcancel` 函数可能会导致线程被错误地取消。在这种情况下，可以通过设置线程的取消状态为 `PTHREAD_CANCEL_DISABLE` 来禁用取消操作，以避免意外的取消。

总之，调用 `pthread_testcancel` 函数可以让线程更加及时地、可靠地响应取消请求，从而增加程序的安全性和稳定性，但需要注意线程同步的问题，以避免出现错误。

##### pthread_exit

`pthread_exit()`是一个线程终止函数，它允许一个线程在它的**任意位置退出**。该函数接受一个参数，表示线程的返回值。

```c
void pthread_exit(void *retval);
```

调用`pthread_exit()`函数会立即终止当前线程的执行，并将传递的参数作为线程的返回值。如果该线程被其他线程等待，那么该返回值可以被其他线程获取。

> 注意：指针`retval`指向的内容不能为函数中局部变量，因为一旦线程函数终止，它们将不再存在。

`pthread_exit()`函数通常在以下情况下使用：

1.  在线程执行完任务后，主动结束自己的执行。

2.  当线程执行出现错误时，使用该函数退出线程。

3.  在主线程中调用`pthread_exit()`函数来结束整个程序的执行。

> 注意：如果在主线程中调用了`pthread_exit(NULL)`，则主线程退出，而不是退出进程，因此如果子线程存在，会继续执行。
> 
> 需要注意的是，当一个线程调用`pthread_exit()`函数后，该线程会**立即终止，不会再执行任何其他操作**。因此，如果线程需要进行一些清理工作，比如释放内存、关闭文件等，就需要在调用`pthread_exit()`函数之前完成这些操作。

#### 等待线程结束

##### pthread_join

`pthread_join`函数用于**等待一个指定线程结束，并回收其占用的资源**。它的原型如下：

```c
int pthread_join(pthread_t thread, void **retval);
```

1.  `thread`：要等待的线程的标识符，即线程创建时返回的`pthread_t`类型的值。该参数指定了需要等待的线程。

2.  `retval`：用于存储线程的返回值的指针。该参数是一个**指向指针的指针，因为线程的返回值的类型可能是不确定的，可能是一个整型、浮点型或者指针等类型**。在`pthread_join`函数返回时，线程的返回值将会被存储在`retval`所指向的内存空间中。

需要注意的是，`retval`参数是可选的，如果不需要获取线程的返回值，可以将其设置为`NULL`。另外，如果线程没有返回值，或者**在线程函数中没有显式地调用`pthread_exit`函数退出线程**，那么`retval`参数将被忽略。

该函数会阻塞当前线程，直到指定的线程`thread`结束执行。具体来说，当我们调用`pthread_join`函数时，如果指定的线程`thread`还在运行中，当前线程就会被阻塞，等待该线程结束；如果线程`thread`已经结束了，那么`pthread_join`函数会立即返回，并将线程的返回值存储在`retval`中。此外，`pthread_join`函数会自动回收线程占用的资源，避免了资源泄露的问题。

以下是一个简单的示例代码，用于演示如何使用`pthread_join`函数等待线程结束并获取其返回值：

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

void *thread_func(void *arg) {
    int *num = (int *)arg;
    printf("Thread is running, parameter is %d\n", *num);
    int *res = malloc(sizeof(int));
    *res = (*num) * (*num);
    pthread_exit((void *)res);
}

int main() {
    pthread_t thread;
    int parameter = 10;
    int *result;
    pthread_create(&thread, NULL, thread_func, &parameter);
    pthread_join(thread, (void **)&result);
    printf("Thread returned: %d\n", *result);
    free(result);
    return 0;
}
```

需要注意的是，如果在线程函数中不调用`pthread_exit`函数退出线程，而是直接返回，那么该线程的返回值将是一个未定义的值，可能会导致程序出现不可预料的错误。因此，**在编写多线程程序时，一定要记得在线程函数中调用`pthread_exit`函数退出线程**。

#### 多线程示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

void *my_thread_func(void *pointer) {
    for (int i = 0; i < 3; ++i)
    {
        // sleep(1);
        int id = *(int *)pointer;
        printf("thread %d running, i=%d\r\n", id, i);
    }
    free(pointer);
}

int main(int argc, char const *argv[])
{
    pthread_t threadId[5];
    
    for (int i = 0; i < 5; ++i)
    {
        int *arg = malloc(sizeof(*arg));
        *arg = i;  // 注意一
        pthread_create(&threadId[i], NULL, my_thread_func, (void *)(arg));
    }

    for (int i = 0; i < 5; ++i)
    {
        pthread_join(threadId[i], (void **)(0));
    }

    printf("main thread end\r\n");

    return 0;
}
```

由于线程之间是异步执行的，因此在输出语句中的 `id` 值可能会和线程实际运行的顺序不一致，**无法保证顺序执行**。如果需要保证顺序执行，可以使用互斥锁来同步线程之间的执行顺序。

```text
root@fw:~/w26/ccodes# ./demo
thread 0 running, k=0
thread 2 running, k=0
thread 2 running, k=1
thread 0 running, k=1
thread 1 running, k=0
thread 1 running, k=1
thread 2 running, k=2
thread 1 running, k=2
thread 3 running, k=0
thread 3 running, k=1
thread 3 running, k=2
thread 4 running, k=0
thread 4 running, k=1
thread 4 running, k=2
thread 0 running, k=2
main thread end
root@fw:~/w26/ccodes#
```

对于注意一的解释：在 `main` 函数中循环创建 5 个线程时，每个线程的 `my_thread_func` 函数都被传递了指向 `i` 的指针，而 `i` 是一个*自动变量*，其生命周期仅在循环内部。**由于线程的创建和调度是异步的**，因此当线程实际运行时，`i` 可能已经被更新成另一个值，这会导致线程使用了错误的数据。

使用 `malloc` 申请临时变量来保存自动变量 `i` 的值，每个线程函数都被传递了一个指向分配的临时变量的指针，该变量保存了正确的 `i` 值。在线程函数中，使用 `*(int *)pointer` 获取 `i` 的值，并在使用完后释放该临时变量的内存空间。

> 何为自动变量？

在C和C++等编程语言中，当在函数或代码块内部声明一个变量时，该变量默认为自动变量。自动变量具有以下特点：

1.  作用域：自动变量的作用域仅限于声明它的代码块内部。这意味着在声明的代码块外部是无法访问到该变量的。

2.  存储方式：自动变量通常存储在栈（stack）上。当进入声明变量的代码块时，该变量会在栈上分配存储空间，当代码块执行完毕时，变量会自动释放所占用的栈空间。

3.  初始化：自动变量在声明时可以选择是否进行初始化。如果未初始化，则其值是不确定的。

自动变量适用于那些在局部范围内使用的临时数据和临时存储需求较小的变量。与全局变量和静态变量相比，自动变量具有更短的生命周期和更小的作用域，能够更有效地管理内存和避免命名冲突。

#### 多线程同步示例

保证线程内顺序执行，可以使用互斥锁来同步线程之间的执行顺序。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void *my_thread_func(void *pointer) {
    pthread_mutex_lock(&mutex);
    for (int k = 0; k < 3; ++k)
    {
        // sleep(1);
        int i = *(int *)(pointer);
        printf("thread %d running, k=%d\r\n", i, k);
    }
    free(pointer);
    pthread_mutex_unlock(&mutex);
}

int main(int argc, char const *argv[])
{
    pthread_t threadId[5];

    pthread_mutex_init(&mutex, NULL);
    
    for (int i = 0; i < 5; ++i)
    {
        int *arg = malloc(sizeof(*arg));
        *arg = i;  // 注意一
        pthread_create(&threadId[i], NULL, my_thread_func, (void *)(arg));
    }

    for (int i = 0; i < 5; ++i)
    {
        pthread_join(threadId[i], (void **)(0));
    }
    
    pthread_mutex_destroy(&mutex);

    printf("main thread end\r\n");

    return 0;
}
```

```text
root@fw:~/w26/ccodes# ./demo
thread 1 running, k=0
thread 1 running, k=1
thread 1 running, k=2
thread 3 running, k=0
thread 3 running, k=1
thread 3 running, k=2
thread 2 running, k=0
thread 2 running, k=1
thread 2 running, k=2
thread 0 running, k=0
thread 0 running, k=1
thread 0 running, k=2
thread 4 running, k=0
thread 4 running, k=1
thread 4 running, k=2
main thread end
root@fw:~/w26/ccodes#
```

在修改后的代码中，我们使用 `pthread_mutex_t` 类型定义了一个互斥锁，并在 `main` 函数中初始化了它。在 `my_thread_func` 函数中，我们在循环前加锁（某一个线程获取了锁），循环结束后解锁（该线程释放了锁，此时其它线程可以获取锁了），以保证线程的顺序执行（先获取到锁的线程，会执行完锁之间的内容，不再会出现上面未加锁的示例中，执行到一半，便去执行其它线程的内容）。最后在 `main` 函数结束前销毁互斥锁。

---
