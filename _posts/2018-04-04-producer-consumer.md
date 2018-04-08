---
layout: post
title: 用Python3解决生产者消费者问题
tags: [生产者消费者, 多线程, Python3, 信号量, 死锁]
excerpt_separator: <!--more-->
---
python中常用协程来编写生产者消费者的问题，利用yield和send方法在生产者消费者之间互相通信，这样写的好处是代码编写简洁方便，但是理解起来比较困难，且只能实现生产者和交替者来回交替的多线程形态，比较单一。

<!--more-->
于是我想尝试使用最常规方式来解决这个问题。计划用人工模拟缓冲区和互斥锁来解决这个问题。我们人工模拟一个容量为5的缓冲区，缓冲区容量和模拟缓冲区的列表都定义为全局变量，这是所有线程有要访问和修改的变量，根据生产者消费者问题的问题描述，来分别写生产者和消费者的问题：当缓冲区不满（volunm < 5）的时候，生产者便往缓冲区里生产填充元素，当缓冲区不空的时候，消费者便从缓冲区里取出元素来消费。
其中有两点需要注意：

* 一是在进行在缓冲区的生产或者消费的操作时，需要加上互斥锁来保证全局变量不会被其他线程改写。
* 二是因为生产行为和消费行为都是不停地在进行的，所以需要使这两个函数无限循环来保证不断进行。

当生产者函数和消费者函数都写好以后，就可以直接在主程序中创建一定数量的生产者和消费者线程并等待他们执行完毕。当然，因为所有的线程都是无限循环的，所以这个程序运行起来后会一直不停地在输出打印结果。
代码如下：

    import threading

    global v, buffer
    v = 5
    buffer = []
    lock = threading.Lock()

    def producer():
        while True:
            lock.acquire()
            if len(buffer) < v:
                buffer.append(1)
                print(threading.current_thread().name)
            lock.release()

    def consumer():
        while True:
            lock.acquire()
            if len(buffer) > 0:
                buffer.pop(0)
                print(threading.current_thread().name)
            lock.release()


    if __name__ == '__main__':
        p = []
        c = []
        for i in range(0, 2):
            pnow = threading.Thread(target=producer, name='producer%s' % i)
            pnow.start()
            p.append(pnow)
            cnow = threading.Thread(target=consumer, name='consumer%s' % i)
            cnow.start()
            c.append(cnow)
        for pnow, cnow in zip(p, c):
            pnow.join()
            cnow.join()


在该版本的生产者消费者问题中，手动去模拟了缓冲区，没有使用信号量来解决这个问题，虽然看上去比较好理解，但是也会产生很多的资源浪费，比方说，明明缓冲区已经满了，还是一直进入producer函数执行到if语句再停下，如果使用信号量来解决这个问题的话，当缓冲区满了或者空了，便会直接阻塞不能执行的那类线程，从而在CPU资源利用上更加合理。
Python中的信号量可以用threading模块中的semaphore，对于semaphore对象：每调用acquire方法一次value属性就-1，调用release方法value属性就+1，当semaphore对象的value属性为0时，acquire的调用被阻塞。利用信号量来描述缓冲区的状况，创建两个信号量，full和empty分别表示满的缓冲区个数和空的缓冲区个数，初始分别为0（一开始时0个满的缓冲区）和缓冲区容量。
生产者和消费者的函数代码如下，其他部分和第一版本相同：

    global semaphore_full, semaphore_empty
    semaphore_empty = threading.Semaphore(5)
    semaphore_full = threading.Semaphore(0)
    lock = threading.Lock()

    def producer():
        while True:
            semaphore_empty.acquire()
            print('producer_empty: ', semaphore_empty._value)
            lock.acquire()
            print(threading.current_thread().name)
            lock.release()
            semaphore_full.release()
            print('producer_full: ', semaphore_full._value)


    def consumer():
        while True:
            semaphore_full.acquire()
            print('consumer_full: ', semaphore_full._value)
            lock.acquire()
            print(threading.current_thread().name)
            lock.release()
            semaphore_empty.release()
            print('consumer_empty: ', semaphore_empty._value)

和第一版本的生产者消费者问题比，除了缓冲区用信号量来描述以外，锁的位置也需要注意，锁需要放在在信号量acquire之后，release之前，就是说，对信号量的操作需要放在lock外面，否则会出现死锁的情况（当对信号量的操作放在锁里面的时候，假设一种情况：一开始不停生产，最终empty的值被减到0之后，在生产者的线程里，empty信号量会阻塞，而因为生产者的线程把信号量上了锁，消费者的线程无法修改信号量，所以整个生产消费环节都无法进行了），有兴趣的朋友可以自己改变锁的位置来体验一把死锁是什么情况。