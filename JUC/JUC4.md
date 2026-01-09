#### 生产者与消费者（等待唤醒机制）

生产者-消费者模式时一个十分经典的多线程协作的模式，它可以打破随机性，让两个线程轮流执行。其中一条线程我们称其为生产者，负责生产数据。另一条称之为消费者，负责消费数据。



#### 理想情况

生产者抢到CPU执行权，饭桌为空，于是生产者做一道菜，消费者吃一道菜，吃完之后饭桌上空了，生产者再做一道菜，消费者再吃一道菜，循环往复。



#### 情况一：消费者等待

一开始，消费者抢到CPU执行权，此时饭桌为空，它只能等待（wait）。一旦wait，CPU执行权就会被生产者抢到，饭桌没菜，生产者做一道菜。此时消费者还在wait，所以生产者要通知消费者开吃了（唤醒，notify），消费者被唤醒之后就开吃。

即，对于消费者来说：

1.判断饭桌上是否有食物

2.如果没有，wait

对于生产者来说：

1.制作食物

2.把食物放在饭桌上

3.唤醒等待的消费者开吃



#### 情况二：生产者等待

一开始，生产者抢到CPU执行权，此时生产者制作食物、把食物放在饭桌上、唤醒。虽然没有人在wait，但是喊一喊也无所谓。下一次，依然是生产者抢到CPU执行权。此时饭桌上已经有食物了，生产者不能再做食物了，所以它只能wait.即生产者的执行流程变化为：

1.判断桌子上是否有食物

2.如果有，wait

3.如果没有，制作食物

4.把食物放在饭桌上

5.唤醒等待的消费者开吃

一旦生产者wait，CPU执行权必然被消费者拿到。由于生产者在wait，消费者的执行流程也要变化：

1.判断桌子上是否有食物

2.如果没有，wait

3.如果有，开吃

4.吃完之后，唤醒生产者继续做

这就是生产者与消费者的完整执行流程。

在这个过程中涉及到三个方法

| 方法名称         | 说明                             |
| ---------------- | -------------------------------- |
| void wait()      | 当前线程等待，直到被其它线程唤醒 |
| void noitfy()    | 随机唤醒单个线程                 |
| void notifyAll() | 唤醒所有线程                     |

这在我之前的简易线程池里有详细的讲解。

 

```java
public class Desk {
    //控制生产者与消费者的执行
    
    //是否有食物 0：没有食物 1：有食物
    //boolean只有两个值，只能控制两条线程，通用性不够
    public static int FoodFlag = 0;
    
    //总个数
    public static int count = 10;
    
    //锁对象
    public static Object Lock = new Object();
    
}
```



```java
public class producer extends Thread {
    @Override
    public void run() {
        while(true) {
            synchronized (Desk.Lock) {
                if(Desk.count == 0) {
                    break;
                } else {
                    //判断饭桌上是否有食物
                    if(Desk.FoodFlag == 1) {
                        //如果有，就等待
                        try {
                            Desk.Lock.wait();
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    } else {
                        //如果没有，就做
                        System.out.println("生产者做菜");

                        //修改桌子上的食物状态
                        Desk.FoodFlag = 1;

                        //唤醒等待的消费者
                        Desk.Lock.notifyAll();
                    }
                }
            }
        }
    }
}
```



```java
public class Consumer extends Thread {
    @Override
    public void run() {
        //1.循环
        //2.同步代码块
        //3.判断共享数据是否到末尾（先写到了末尾）
        //4.判断共享数据是否到末尾（再写没有到末尾，执行核心逻辑）

        while(true) {
            synchronized (Desk.Lock) {
                if(Desk.count == 0) {
                    //没有食物了，结束
                    break;
                } else {
                    //先判断饭桌上是否有食物
                    //如果没有，等待
                    //如果有，吃
                    //吃完之后，唤醒生产者
                    //吃的总数减一
                    //修改饭桌的状态

                    if(Desk.FoodFlag == 0) {
                        //饭桌上没有食物，等待
                        try {
                            Desk.Lock.wait();//让当前线程与锁进行绑定
                        } catch (InterruptedException e) {
                            throw new RuntimeException(e);
                        }
                    } else {
                        //如果有，就开吃。吃的总数先减一
                        System.out.println("消费者在吃，还能再吃"+(--Desk.count)+"碗");
                        //吃完之后，唤醒
                        Desk.Lock.notifyAll();//唤醒绑定在这把锁上的所有线程
                        //修改桌子状态
                        Desk.FoodFlag = 0;
                    }
                }
            }
        }
    }
}
```

不管是什么情况，输出都必然是这样：

生产者做菜
消费者在吃，还能再吃9碗
生产者做菜
消费者在吃，还能再吃8碗
生产者做菜
消费者在吃，还能再吃7碗
生产者做菜
消费者在吃，还能再吃6碗
生产者做菜
消费者在吃，还能再吃5碗
生产者做菜
消费者在吃，还能再吃4碗
生产者做菜
消费者在吃，还能再吃3碗
生产者做菜
消费者在吃，还能再吃2碗
生产者做菜
消费者在吃，还能再吃1碗
生产者做菜
消费者在吃，还能再吃0碗



#### 等待唤醒机制（阻塞队列实现）

还是以做菜为例。厨师做好菜之后，可以把菜都放在传送带（阻塞队列）上，顾客就可以从传送带上自己拿菜吃（类似寿司店那种转盘，转到你面前就可以把菜拿下来）。我们可以规定传送带中最多可以放多少碟菜，如果最多一碟，那么就和上面一样，做一碗吃一碗。

队列，先进先出，不赘述。阻塞，意味着put数据时，如果放不进去，会等待，即阻塞。take数据时，取出第一个数据，如果取不到，也会等待，即阻塞。

##### 阻塞队列的继承结构

阻塞队列实现了四个接口：Iterable -> Collection -> Queue -> BlockingQueue

最顶层的是Iterable接口，意味着阻塞队列可以使用迭代器或者增强for来遍历。阻塞队列还实现了Collection接口，所以阻塞队列实际上是一个单列集合。

由于上面四个都是接口，所以我们不能直接创建它们的对象。我们要创建的是两个实现类的对象：`ArrayBlockingQueue`和`LinkedBlockingQueue`，前者底层是数组，有界，创建时要指定长度。后者底层是链表，无界，创建时不需要指定长度。但不是真正的无界，最大为int的最大值。

```java
public class producer extends Thread {

    ArrayBlockingQueue<String> queue;

    public producer(ArrayBlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while(true) {
            //厨师不断把菜放进阻塞队列中
            try {
                queue.put("菜");
                System.out.println("厨师做了一碟菜");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }

        }
    }
}
```



```java
public class Consumer extends Thread {

    ArrayBlockingQueue<String> queue;

    public Consumer(ArrayBlockingQueue<String> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        while(true) {
            //不断从阻塞队列中获取菜
            try {
                String food = queue.take();
                System.out.println("顾客吃了一碟"+food);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }

    }
}
```



```java
public class ThreadDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //需求：利用阻塞队列完成生产者与消费者（等待唤醒机制）的代码
        //生产者与消费者必须使用同一个阻塞队列

        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(1);

        producer pro = new producer(queue);
        Consumer con = new Consumer(queue);

        pro.start();
        con.start();
    }
}
```



之所以不需要锁，是因为在put和take方法底层已经有锁了。

```java
    public void put(E e) throws InterruptedException {
        Objects.requireNonNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length)
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
```

```java
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
```



运行后发现，有重复的输出。实际上是因为打印语句写在了锁的外面。不过无伤大雅，因为打印语句并没有改变共享数据，只不过在控制台上的输出有点混乱。 



#### 线程的生命周期

虽然在JUC2中说过，但这次进行一些补充：

1.创建线程对象：新建状态

调用start方法后：

2.有执行资格（有抢的资格），没有执行权（还没有抢到），即正在抢但没有抢到：就绪状态

抢到CPU的执行权后：

3.有执行资格，有执行权：运行代码状态。（在这个过程中，其它线程可能会抢走CPU的执行权，被抢走后又回到就绪状态）（实际上没有这个状态，只是为了我们方便理解）

(i) 如果调用sleep方法，就进入计时等待状态（没有执行资格和执行权），直到到时间了才回到就绪状态

(ii) 如果调用wait方法，就进入等待状态（没有执行资格和执行权），直到被notify唤醒了才回到就绪状态

(iii) 如果无法获取锁，就进入阻塞状态（没有执行资格和执行权），直到得到锁了才回到就绪状态

将run方法的代码执行完毕后：

4.线程死亡，变成垃圾：死亡状态



总结：

新建状态（NEW） -> 创建线程对象

就绪状态（RUNNABLE） -> start方法

阻塞状态（BLOCKED） -> 无法获得锁对象

等待状态（WAITING） -> wait方法

计时等待（TIME_WAITING） -> sleep方法

结束状态（TERMINATED） -> 全部代码运行完毕