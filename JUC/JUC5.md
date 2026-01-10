#### 线程池

你要吃饭，于是你买了个碗。吃完饭后，你懒得洗碗，于是就把碗砸了。但是下一次吃饭时，你又没有碗了。于是你发现了问题：1.每次都要买碗，浪费时间；2.每次吃完都要把碗摔了，浪费资源。

解决方案：准备一个碗柜即可。

在没有接触线程池之前，我们写多线程都是用到线程的时候就创建，用完之后线程就消失，这会浪费操作系统的资源。所以，我们也准备一个容器——线程池，用来存放线程。

起初线程池为空，当我们提交一个任务时，线程池会自动创建线程来处理这个任务，处理完了再把线程放到线程池中。这样当我们提交第二个任务时，就不需要再创建新的线程了，用线程池中的线程即可。

特殊情况：如果当我们提交第二个任务时，第一个线程还在执行任务，此时线程池就会创建新的线程去执行这第二个任务。如果我们又提交了很多个任务，线程池也同样会创建线程执行这些任务。不过，这并不意味着线程池创建线程的数量没有上限，而且我们可以自己手动设置线程数量上限。如果我们设置最大线程数量为3，那么倘若提交了5个任务，后面的2个任务只能先排队等着。



#### 线程池代码实现

Executors：线程池的工具类，通过调用方法返回不同类型的线程池对象。

| 方法名称                                                     | 说明                     |
| ------------------------------------------------------------ | ------------------------ |
| public static ExecutorService new CachedThreadPool()         | 创建一个没有上限的线程池 |
| public static ExecutorService new newFixedThreadPool(int nThreads) | 创建有上限的线程池       |

注意：没有上限不是真的可以无穷大，上限为int的最大值



```java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        for (int i = 1; i <= 100; i++) {
            System.out.println(Thread.currentThread().getName()+"---"+i);
        }
    }
}
```



```java
public class MyThreadPoolDemo {
    public static void main(String[] args) {
        //1.获取线程池对象
        ExecutorService pool1 = Executors.newCachedThreadPool();

        //2.提交任务
        pool1.submit(new MyRunnable());
        
        //3.销毁线程池
        pool1.shutdown();
    }
}
```



```java
public class MyThreadPoolDemo {
    public static void main(String[] args) {
        //1.获取线程池对象
        ExecutorService pool1 = Executors.newFixedThreadPool(3);

        //2.提交任务
        pool1.submit(new MyRunnable());
        pool1.submit(new MyRunnable());
        pool1.submit(new MyRunnable());
        pool1.submit(new MyRunnable());

        //3.销毁线程池
        //pool1.shutdown();
    }
}
```

可以通过debug验证确实创建了3个线程处理任务。关注两个参数：`workQueue`和pool（pool里面有pool size\active threads\queued tasks\completed tasks这些参数）



#### 自定义线程池

核心元素：核心线程数量（非负）、线程池中最大线程数量（大于等于核心线程数量）、空闲时间（值）（非负）、空闲时间（单位）（用TimeUnit指定）、阻塞队列（非null）、创建线程的方式（非null）、执行任务过多时的解决方案（非null）。

| 任务拒绝策略                           | 说明                                                   |
| -------------------------------------- | ------------------------------------------------------ |
| ThreadPoolExecutor.AbortPolicy         | 默认策略：丢弃任务并抛出RejectedExecutionException异常 |
| ThreadPoolExecutor.DiscardPolicy       | 丢弃任务，但是不抛出异常，不推荐                       |
| ThreadPoolExecutor.DiscardOldestPolicy | 抛弃队列中等待最久的任务，然后把当前任务加入队列中     |
| ThreadPoolExecutor.CallerRunsPolicy    | 调用任务的run()方法，绕过线程池直接执行                |

```java
public class MyThreadPoolDemo {
    public static void main(String[] args) {
        ThreadPoolExecutor pool = new ThreadPoolExecutor(
                3,//核心线程数量
                6,//最大线程数量
                60,//最大空闲时间的值
                TimeUnit.SECONDS,//最大空闲时间的单位
                new ArrayBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy()
        );
    }
}
```



#### 最大并行数

以4核8线程的CPU为例， 相当于CPU有4个大脑，只不过Intel的超线程技术让它把原来的4个大脑虚拟成8个，所以最大并行数为8.在设备管理器的处理器中就可以看到。或者打开任务管理器的性能，看右下角的参数即可。但是有些操作系统不会把所有的CPU资源都交给一个软件用，所以这个方法不能说是百分百稳妥。也可以在IDEA中使用代码查看：

```java
System.out.println(Runtime.getRuntime().availableProcessors());
```

该方法会返回JVM可用处理器的数目。如果和任务管理器中看到的参数一样，那么说明对于你的操作系统来说，Java可以使用所有的资源。



#### 线程池的合适大小

CPU密集型运算：计算较多，读取本地文件或读取数据库的操作较少。对于这种情况，线程池的合适大小为最大并行数+1，可以实现CPU的最优利用率。

对于I/O密集型运算：读取本地文件或读取数据库的操作较多，对于这种情况，线程池的合适大小为：

`最大并行数*期望CPU利用率*总时间/CPU计算时间`

其中总时间为CPU计算时间+等待时间。

例：从本地文件中读取两个数据，并进行相加。这个过程可以拆分成两步：1.读取两个数据；2.相加

我们假设这两个步骤都用时1秒，步骤一与CPU无关，步骤二与CPU相关，因此总时间为2秒，CPU计算时间为1秒。