#### 线程的安全问题

需求：某电影院目前正在上映一部电影，共有100张票，而它有3个窗口卖票，设计一个程序模拟该电影院卖票。

我们将三个窗口视为三个线程。

```java
public class MyThread extends Thread {

    int ticket = 0;//0-99

    @Override
    public void run() {
        while(ticket <= 99) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(getName() + "卖票：" + ticket++);
        }
    }
}
```

```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
        MyThread t3 = new MyThread();
        t1.setName("窗口A");
        t2.setName("窗口B");
        t3.setName("窗口C");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

我们发现，三个窗口的卖票操作似乎是独立的，也就是总共卖了300张票。我们用static关键字修饰ticket：

```java
public class MyThread extends Thread {

    static int ticket = 0;//0-99

    @Override
    public void run() {
        while(ticket <= 99) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println(getName() + "卖票：" + ticket++);
        }
    }
}
```

再次运行，我们发现还是有重复卖票的现象，甚至卖的票还超出了范围。

原因是：线程在执行代码的时候，CPU的执行权随时可能被其它线程抢走。例如当某个线程执行到ticket自增，还没来得及输出，执行权就被其它线程抢走了，那么就会出现重复卖票的现象。超出范围的原因也同理，ticket刚刚自增完，还没走出if语句块，执行权就被其它线程抢走了，其它线程也执行ticket自增，于是导致它们都还没走出if语句块，ticket就已经超出了范围。

解决方法：利用同步代码块，将操作共享数据的代码锁起来。当有线程进入被锁起来的代码块，其它线程就算抢到了CPU的执行权也得在外面等着。（实际上，锁的初步理解在我的简易线程池里已经说过了）。就好比你和你同学都想窜稀，但只有一个坑（共享资源），你抢到了坑就把门锁上，你同学再怎么急也进不去。

格式：

```java
synchronized(锁) {
    操作共享数据的代码
}
```

锁是默认打开的，如果有一个线程进去了，锁自动关闭。锁里面的代码全部执行完毕后，锁自动打开。

```java
   @Override
    public void run() {
        while(ticket <= 99) {
            synchronized (obj) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(getName() + "卖票：" + ticket++);
            }
        }
    }
```

可以看到没有重复卖票了，但还是会出现卖票超过99的情况。这是因为条件判断在锁外，就有可能出现：某个线程释放锁之前，其它线程已经通过了while条件判断的情况。修改如下：

```java
    @Override
    public void run() {
        while(true) {
            synchronized (obj) {
                if(ticket > 99) {
                    break;
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(getName() + "卖票：" + ticket++);
            }
        }
    }
```

但是需要注意，不能把while(true)写在锁里面，不然只有一个窗口卖票。另外，锁对象必须是唯一的。就好比厕所门有两个锁，你和你朋友都有这两把锁的钥匙，那你们都能操作共享数据，就失去了加锁原本的意义了。将obj改成this就可以验证一下（this表示当前线程对象）。

当然我们一般不会额外创建一个obj对象，一般使用的是当前类的字节码文件。

```java
    @Override
    public void run() {
        while(true) {
            synchronized (MyThread.class) {
                if(ticket > 99) {
                    break;
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(getName() + "卖票：" + ticket++);
            }
        }
    }
```

类的字节码文件必然是唯一的，因此可以拿来当作锁对象。



如果我们要将一个方法里所有的代码都锁起来，那么就没必要用同步代码块了，直接使用同步方法：将synchronized关键字加到方法上。具体来说，就是将synchronized写在修饰符的后面，返回值类型的前面。只不过同步方法中的锁对象是不能自己指定的，Java已经指定好了。如果当前方法是非静态的，那么锁对象就是this，即当前方法的调用者。 如果是静态的，锁对象就是当前类的字节码文件对象。

```java
public class MyRunnable implements Runnable{
    int ticket = 0;//注意这里不需要加static
    @Override
    public void run() {
        while(true) {
            if (extracted()) break;
        }
    }

    private synchronized boolean extracted() {
        if(ticket == 100) {
            return true;
        } else {
            System.out.println(Thread.currentThread().getName()+"卖票"+ticket++);
        }
        return false;
    }
}
```

不需要加static关键字的原因是，我们只创建一个`MyRunnable`对象，因为它是作为参数传递的，表示线程要执行的任务。而前面要加static，是因为`MyThread`继承了Thread类，会创建多个`MyThread`对象。

抽取方法：选中代码块，按Ctrl+Alt+M，就可以将这些代码抽取成一个方法

```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyRunnable mr = new MyRunnable();

        Thread t1 = new Thread(mr);
        Thread t2 = new Thread(mr);
        Thread t3 = new Thread(mr);

        t1.setName("窗口A");
        t2.setName("窗口B");
        t3.setName("窗口C");

        t1.start();
        t2.start();
        t3.start();
    }
}
```



我们知道，将`StringBuilder`的实例用于多个线程是不安全的，如果需要这样的同步，应当使用`StringBuffer`，因为`StringBuffer`的成员方法都有synchronized，即所有方法都是同步的。而`StringBuilder`的成员方法没有。



上文说过，进入同步代码块后锁自动关闭，同步代码块中的代码执行完毕后锁自动释放。那么如何手动加锁、解锁呢？为了更清晰地表达如何加锁和释放锁，JDK5以后提供了一个新的锁对象Lock

Lock中提供了获得锁和释放锁的方法：

void lock()：获得锁

void unlock()：释放锁

Lock是接口，不能直接实例化，可以采用它的实现类`ReentrantLock`来实例化。

```java
public class MyThread extends Thread {
    static int ticket = 0;//0-99
    static Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while(true) {
            lock.lock();
                if(ticket > 99) {
                    break;
                }
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(getName() + "卖票：" + ticket++);
            lock.unlock();
        }
    }
}
```

```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
        MyThread t3 = new MyThread();

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
```

运行多次后，发现程序没停。分析一下原因：某个线程执行到了sleep，进入阻塞状态，此时另外某个线程拿到了执行权，但因为此时进入阻塞状态的线程还没释放锁，导致这个新拿到执行权的线程无法获得锁。另外，当ticket为100时，会跳过unlock直接break，它自己都死了还不舍得解开锁，这就导致了其它线程一直停在获取锁的那一行，所以程序不会停止。Ctrl+Alt+T选使用try-catch-finally解决，因为不管怎么样finally中的代码都会被执行。

```java
public class MyThread extends Thread {
    static int ticket = 0;//0-99
    static Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while(true) {
            lock.lock();
            try {
                if(ticket > 99) {
                    break;
                }
                Thread.sleep(10);

                System.out.println(getName() + "卖票：" + ticket++);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }
    }
}
```



#### 死锁

一个简单的比喻：你和别人抢东西，争执不下，你在等着对面松手，对面在等着你松手，就这样一直僵持。需要注意的是，不要把两个锁嵌套起来。

