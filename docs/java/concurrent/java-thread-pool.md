## 线程池的创建

生成环境使用`ThreadPoolExecutor`，不要使用JDK预定义的`FixedThreadPool`、`CachedThreadPool`、`SingleThreadPool`。

```java
public ThreadPoolExecutor(int corePoolSize, 
                          int maximumPoolSize, 
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

- `corePoolSize`：核心线程数。当提交一个任务时，线程池会新建一个线程来执行任务，直到当前线程数等于corePoolSize。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。
- `maximumPoolSize`：最大线程数。线程池的阻塞队列满了之后，如果还有任务提交，并且当前的线程数小于maximumPoolSize，则会新建线程来执行任务。注意，如果使用的是无界队列，该参数也就没有什么效果了。
- `keepAliveTime`：存活时间。该参数只有在线程数大于corePoolSize时才会生效。
- `unit`：时间单位
- `workQueue`：任务队列
  - `ArrayBlockingQueue`：基于数组结构的有界阻塞队列，FIFO。
  - `LinkedBlockingQueue`：基于链表结构的有界阻塞队列，FIFO。
  - `SynchronousQueue`：不存储元素的阻塞队列，每个插入操作都必须等待一个移出操作，反之亦然。
  - `PriorityBlockingQueue`：具有优先级别的阻塞队列。
- `threadFactory`：线程工厂
- `handler`：拒绝策略。线程池中的线程已经饱和了，而且阻塞队列也已经满了，则线程池会选择一种拒绝策略来处理该任务。
  - AbortPolicy：直接抛出异常，默认策略。
  - CallerRunsPolicy：用调用者所在的线程来执行任务。
  - DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务。
  - DiscardPolicy：直接丢弃任务。
  - 自定义拒绝策略，实现RejectedExecutionHandler接口。

