# Java多线程—Callable、Future和FutureTask

[TOC]

> 前言：如有不正确的地方，还望指正。

# Callable

- Callable接口和Runnable接口的作用类似
- 不同的是Callble接口call方法有返回值，返回的是传进来的V类型，而Runnable接口run方法无返回值

![Java多线程——Callable、Future……](http://p3.pstatp.com/large/322c0000733ed7bce5ab)

# Future

- Futurue是一个接口，Future 表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。计算完成后只能使用 get 方法来获取结果，如有必要，计算完成前可以阻塞此方法。取消则由 cancel 方法来执行。还提供了其他方法，以确定任务是正常完成还是被取消了。一旦计算完成，就不能再取消计算。如果为了可取消性而使用 Future 但又不提供可用的结果，则可以声明 Future<?> 形式类型、并返回 null 作为底层任务的结果。
- Cancel方法：试图取消任务的执行，如果任务已完成或者已取消或者无法取消，则返回false。如果任务尚未启动，执行此方法后，将不会再运行。如果任务已经启动，则可以选择参数确定是否尝试停止任务
- isCancelled方法：如果任务正常完成前将其取消，返回true
- get方法：等待计算完成，然后获取结果
- isDone方法：如果任务完成，返回true

![Java多线程——Callable、Future……](http://p3.pstatp.com/large/321200034aa872954d5b)

# FutureTask

- FutureTask是一个类，实现了RunnableFuture接口，而RunnableFuture接口继承了Runnable和Future接口
- FutureTask的有两个构造方法，参数是Callable和Runnable

![Java多线程——Callable、Future……](http://p1.pstatp.com/large/321a000437dec974e3c9)

# AbstractExecutorService的submit方法

- AbstractExecutorService是一个抽象类，也是ThreadPoolExecutor的父类
- public class ThreadPoolExecutor extends AbstractExecutorService
- public Future<?> submit(Runnable task)
- public <T> Future <T> submit< Runnable task, T result>
- public <T> Future<T> submit < Callable <T> task >
- 对比ThreadPoolThread的execute方法，execute无返回值，而三个submit方法返回的都是Future，可以获取任务执行结果，还可以取消任务

# 使用

- Future用法

![Java多线程——Callable、Future……](http://p3.pstatp.com/large/321200034c06df776ff5)

- FutureTask用法

![Java多线程——Callable、Future……](http://p3.pstatp.com/large/320a00049be1da5c9bee)

- 不同的地方，从代码可以看出第一份代码中future作为submit方法的返回值，而第二份代码，futureTask作为submit方法的参数。

总结

- Callable和Runable都是接口，二者作用类似，不同的是Callable有返回值，而Runnable无返回值
- Callable一般和ExecutorService一起使用，其返回的值是submit方法返回的Future得到的
- Future可以查看任务执行情况，可以取消任务
- 有时候并不需要返回值，但是由于需要可取消任务，所以使用submit方法而不使用execute方法
- FutureTask是一个类，实现了RunnableFuture接口（继承了Runnable和Future接口），FutureTask可以做为submit方法的参数，并通过FutureTask可以查看任务执行状态





资料来源：

http://www.toutiao.com/a6454451457054736910/?tt_from=mobile_qq&app=news_article&iid=12910008925