#### 多线程中常用的成员方法

| 方法名称                         | 说明                                     |
| -------------------------------- | ---------------------------------------- |
| String getName()                 | 返回此线程的名称                         |
| void setName(String name)        | 设置线程的名字（构造方法也可以设置名字） |
| static Thread currentThread()    | 获取当前线程的对象                       |
| static void sleep(long time)     | 让线程休眠指定的时间，单位为毫秒         |
| setPriority(int newPriority)     | 设置线程的优先级                         |
| final int getPriority()          | 获取线程的优先级                         |
| final void setDaemon(boolean on) | 设置为守护线程                           |
| public static void yield()       | 出让线程/礼让线程                        |
| public static void join()        | 插入线程/插队线程                        |

在Java中，线程的优先级最小是1，最大是10，默认是中间的5. 线程优先级越大，抢占到CPU的概率越高。



##### `getName`

线程有默认的名字，格式为：Thread-X（X为序号，从0开始）。

我们选中Thread，按下Ctrl+N，搜索Thread，点击”所有非项目条目“，查看`java.lang`包下的Thread.找到它的空参构造：

```java
public Thread() {
        this(null, null, "Thread-" + nextThreadNum(), 0);
    }
```

可以看到，这个格式确实是被写死了的。

```java
    private static int threadInitNumber;
    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }
```



##### `setName`

如果要给线程设置名字，可以用setName方法，也可以使用Thread的构造函数。但是因为MyThread继承了Thread，而子类不继承父类的构造方法，所以我们要自己写一个，用super关键字调用父类的构造。按下Alt+Insert，选择你要继承的构造方法。按住Ctrl不松即可同时选多个。

```java
public class MyThread extends Thread {
    public MyThread() {
    }

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(getName());
        }
    }
}
```



##### `currentThread`

```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        System.out.println(Thread.currentThread().getName());
    }
}
```

可以看到输出的是main.当JVM启动之后，会自动启动多条线程。其中有一条线程就是main线程。其作用就是调用main方法并执行里面的代码。



##### 线程优先级

线程的调度有两种。

1.抢占式调度：多个线程抢夺CPU的执行权，CPU在什么时候执行哪条线程、执行多久都是不确定的

2.非抢占式调度：所有线程轮流执行，执行的时间差不多。

在Java中采取的是抢占式调度的方式，具有很大的随机性。线程的优先级越高，抢到CPU执行权的概率越大。注意，只是概率问题。



##### 守护线程

通俗地说，就是备胎线程。当其它的非守护线程执行完毕后，守护线程会陆续结束。例如非守护线程是打印1-10，守护线程是打印1-100.当非守护线程执行完后（注意，不是等到非守护线程执行完了，守护线程才开始执行），守护线程会被告知可以结束了，只不过在这个被告知的过程中守护线程自己也运行了。所以守护线程可能打印到1-100之间的任意一个数字就结束了。



##### 出让线程/礼让线程

MyThread.java

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(getName() + i);
        }
    }
}
```

ThreadDemo.java

```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.setName("巨人");
        t2.setName("矮人");

        t1.start();
        t2.start();
    }
}
```



现在我们希望输出能够更加均匀一些，使用yield方法

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println(getName() + i);
            Thread.yield();//出让CPU的执行权
        }
    }
}
```

简单地说，当巨人线程打印了之后，由于CPU执行权还在巨人线程，它可能会一下子打印很多个数字，直到CPU执行权被矮人线程抢走。加上yield之后表示，当巨人线程打印完了之后，它会出让CPU的执行权，下一次再运行的时候巨人和矮人会再次抢夺CPU的执行权。但是只是尽可能均匀，不一定真的均匀。



##### 插入线程/插队线程

```java
public class MyThread extends Thread {

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {//100次
            System.out.println(getName() + i);
            Thread.yield();//出让CPU的执行权
        }
    }
}
```



```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread t = new MyThread();
        t.setName("巨人");
        t.start();

        for (int i = 0; i < 10; i++) {
            System.out.println("main" + i);
        }
    }
}
```



现在我想把巨人线程插到main线程之前，等巨人线程执行完了，main线程再执行。可以使用join方法。

```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread t = new MyThread();
        t.setName("巨人");
        t.start();
        t.join();//把t线程插入到当前线程之前

        for (int i = 0; i < 10; i++) {
            System.out.println("main" + i);
        }
    }
}
```



#### 线程的生命周期

1.创建线程对象：新建状态

调用start方法后：

2.有执行资格（有抢的资格），没有执行权（还没有抢到），即正在抢但没有抢到：就绪状态

抢到CPU的执行权后：

3.有执行资格，有执行权：运行代码状态。（在这个过程中，其它线程可能会抢走CPU的执行权，被抢走后又回到就绪状态）

如果此时遇到了sleep方法或其它阻塞式方法，线程没有执行资格与执行权，进入阻塞状态。sleep方法时间到或者其它阻塞方法结束后，又回到就绪状态。

将run方法的代码执行完毕后：

4.线程死亡，变成垃圾：死亡状态

