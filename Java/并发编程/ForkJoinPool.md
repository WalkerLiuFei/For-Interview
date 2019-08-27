# ForkJoinPool

## 介绍

`ForkJoinPool`  和`ExcutorService` 类似,看源码就知道，它继承了` AbstractExecutorService` 这个类，但是`ForkJoinPool`的优势在于，他能很好的运行那些含有很多子任务的任务，当你把一个Task 交由`ForkJoinPool` 运行时，他可以将Task分解成若干subTask, 这些SubTask也交由`ForkJoinPool`来处理

[参考，工作窃取算法的原理以及引进](https://houbb.github.io/2019/01/18/jcip-39-fork-join)

[参考，fork / join 通俗易通讲解](https://zhuanlan.zhihu.com/p/38204373) 

## Fork and Join Explained 

### Fork 

fork其实就是将Task分解成子任务，每个子任务可以在不同的`CPU`上执行。如果任务被赋予的工作量足够大，那么任务只会将自身分成子任务。 将任务拆分为子任务存在开销，因此对于少量工作，此开销可能大于通过并发执行子任务所实现的加速。所以，必须要要有一个`thredhold` 。

### Join

When a task has split itself up into subtasks, the task waits until the subtasks have finished executing.

Once the subtasks have finished executing, the task may *join* (merge) all the results into one result. This is illustrated in the diagram below:

Of course, not all types of tasks may return a result. If the tasks do not return a result then a task just waits for its subtasks to complete. No result merging takes place then.



## WorkerQueue

`WorkerQueue`内部是一个`ForkJoinTaskd`的双端队列，同时支持`LIFO` 和`FIFO` ，`WorkQueue` 其实是是一个`ForkJoinPool`的一个内部类，在`Pool` 中有多个`WorkerQueue`。其相当于一个Worker。



## ForkJoinTask

是一个可以包装Runable的Task,在`ForkJoinPool` 中它是实际的工作单位。

## ForkJoinWorkerThread

`ForkJoinWorkerThread extends Thread` 只是用来维护 和工作的ForkJoinPool的联系。



## 工作原理图



![img](https://img-blog.csdnimg.cn/20181111222804684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA4NDEyOTY=,size_16,color_FFFFFF,t_70)

## 工作流程图



![img](https://img-blog.csdnimg.cn/20181111222837182.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA4NDEyOTY=,size_16,color_FFFFFF,t_70)