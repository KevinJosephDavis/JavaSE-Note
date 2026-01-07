#### 线程与进程

线程是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。而进程是程序的基本执行实体。这个在我之前的简易线程池里面也有提到过。



#### 并发与并行

并发：在同一时刻，有多个指令在单个CPU上**交替**执行

并行：在同一时刻，有多个指令在多个CPU上**同时**执行

并发与并行不冲突，可能是同时进行的。



#### 多线程的实现方式（一）：继承Thread类

1.自定义一个类继承Thread

2.重写run方法

3.创建子类的对象，并启动线程

MyThread.java

```java
package learn.juc;

public class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(getName() + "练剑！");
        }
    }
}
```



ThreadDemo.java

```java
package learn.juc;

public class ThreadDemo {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();

        t1.setName("线程1");
        t2.setName("线程2");

        t1.start();
        t2.start();

    }
}
```

运行后发现报了一个错误：没有为模块`MultiThread`指定JDK（如果是项目全局的JDK配置丢失了，那么会在代码里直接报错）。点击顶部菜单 文件  -> 项目结构（或快捷键Ctrl+Alt+Shift+S） -> 项目 -> 语言级别 -> 选择与你本地JDK版本匹配的项目语言级别。还是在项目结构界面，选择模块，点击`MultiThread` -> 依赖 -> 模块SDK -> 选择与你项目匹配的JDK

再运行，发现左侧弹出一个红框，其内容是”没有为模块`MultiThread`指定输出路径“。没有指定输出路径，那么IDEA就不知道把编译后的class文件放到哪里，所以我们要给模块配置输出目录。还是找到项目结构。

方法一：项目级统一输出路径。在左侧找到项目，在编译器输出那里指定输出目录（在项目文件夹下新建`out\production\MultiThread`作为编译器输出目录）。配置简单，适合简单项目、简单模块。

方法二：模块单独输出路径。在左侧找到模块 -> 右侧切换到路径 -> 点击 使用模块编译输出路径 -> 选择输出目录 -> 测试输出目录可以选择同一个目录。每个模块有独立的编译目录，多模块项目中便于区分不同模块的编译产物，结构更清晰。

再运行，就可以正常看到输出结果了。

方法一的可扩展性较差，不能再继承其它的类。





#### 多线程的实现方式（二）：实现Runnable接口

1.自己定义一个类实现Runnable接口

2.重写里面的run方法

3.创建自己的类的对象

4.创建一个Thread类的对象，并开启线程

MyThread.java

```java
package learn.juc;

public class MyThread implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            //获取到当前线程的对象，因为getName是Thread类的方法
            System.out.println(Thread.currentThread().getName() + "练剑！");
        }
    }
}

```



ThreadDemo.java

```java
package learn.juc;

public class ThreadDemo {
    public static void main(String[] args) {
        //创建MyThread对象，表示多线程要执行的任务
        MyThread mt = new MyThread();

        //创建Thread对象，把MyThread对象作为参数传递
        Thread t1 = new Thread(mt);
        Thread t2 = new Thread(mt);

        //设置线程名称
        t1.setName("线程1");
        t2.setName("线程2");

        //启动线程
        t1.start();
        t2.start();

    }
}
```



#### 多线程的实现方式（三）：利用Callable接口和Future接口方式实现

可以看到前面两个方法中，重写的run方法都没有返回值，拿不到多线程运行的结果。而这个第三种实现方式可以拿到多线程运行的结果。

1.创建一个类`MyCallable`实现`Callable`接口

2.重写call（有返回值，表示多线程运行的结果）

3.创建`MyCallable`对象（表示多线程要执行的任务）

4.创建`FutureTask`对象（管理多线程运行的结果）

5.创建Thread类的对象，并启动（表示线程）

MyCallable.java

```java
package learn.juc;

import java.util.concurrent.Callable;

public class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        int sum = 0;
        for (int i = 0; i < 100; i++) {
            sum += i;
        }
        return sum;
    }
}

```



ThreadDemo.java

```java
package learn.juc;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //1.创建MyCallable对象，表示多线程要执行的任务
        MyCallable mc = new MyCallable();

        //2.创建FutureTask对象，管理多线程运行的结果
        FutureTask<Integer> ft = new FutureTask<>(mc);
        //3.创建Thread对象，把FutureTask对象作为参数传递
        Thread t1 = new Thread(ft);
        //4.启动线程
        t1.start();
        //5.获取多线程运行的结果
        Integer res = ft.get();
        System.out.println(res);
    }
}

```



实际上在我的线程池学习日记里是有提到future这个东西的：

```cpp
// add new work item to the pool
template<class F, class... Args>
auto ThreadPool::enqueue(F&& f, Args&&... args)
    -> std::future<typename std::result_of<F(Args...)>::type>
{
    using return_type = typename std::result_of<F(Args...)>::type;

    auto task = std::make_shared< std::packaged_task<return_type()> >(
        std::bind(std::forward<F>(f), std::forward<Args>(args)...)
    );

    std::future<return_type> res = task->get_future();
    {
        std::unique_lock<std::mutex> lock(queue_mutex);

        // don't allow enqueueing after stopping the pool
        if(stop)
            throw std::runtime_error("enqueue on stopped ThreadPool");

        tasks.emplace([task](){ (*task)(); });
    }
    condition.notify_one();
    return res;
}
```

在这里，future的作用就是异步获取结果，调用者可以通过其get方法拿到结果。这和上面JUC的future类似。