## 2022年1月12日

------

- **interrupt()**方法用于请求中断线程
  - 当线程处于运行状态时，调用interrupt()方法后中断标记变为true，可调用**isInterrupted()**方法查看标记状态，如果标记为true就执行退出操作; 
  - 当线程处于阻塞状态时(调用了sleep()、join()、wait()等方法时)，调用interrupt()方法后会抛出**InterruptedException**异常，并清除打断标记，此时标记状态为false，通过捕获该异常来执行退出操作。
- 操作系统层面的线程状态(**5种**)：初始状态 -可运行状态 - 运行状态 - 阻塞状态 - 终止状态
- JAVA API 层面的线程状态(**6种**)：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATIED
  - **BLOCKED：**处于想要获得锁的等待状态
  - **WAITING：**处于没有时间的等待状态，例如调用join()方法
  - **TIMED_WAITING**：处于有时间的等待状态，例如调用Thread.sleep(1000)
- 注意：Java Api 层面的RUNNBALE状态包含了操作系统层面的可运行状态、运行状态、阻塞状态(由于BIO导致的线程阻塞，在Java里无法区分，仍然认为是RUNNABLE状态)

## 2022年1月13日

------

- 笔记地址：https://blog.csdn.net/m0_37989980/article/details/111408759

- **synchronized** 用法：

  - 1.在方法内部：

    - ```java
      Object obj = new Object();
      Thread t1 = new Thread(()->{
          synchronized(obj){
              for (int i = 0; i < 5000; i++) {
                  count++;
              }
          }
      });
      ```

  - 2.在方法上：

    - 修饰**成员方法**

      ```java
      class Test{
      	public synchronized void test() {
          
      	}
      }
      等价于
      class Test{
      	public void test() {
          	synchronized(this) {// 锁住的是调用该方法的实例对象
              
          		}
      	}
      }
      ```

    - 修饰**静态方法**

      ```java
      class Test{
          public synchronized static void test() {
              
          }
      }
      等价于
      class Test{
      	public static void test() {
      		synchronized(Test.class) {// 锁住的是类对象
                  
      		}
      	}
      }
      ```

- 局部变量是线程安全的，但是**局部变量引用的对象不一定安全**：如果该对象没有逃离方法的作用访问，它是线程安全的；如果该对象逃离方法的作用范围，需要考虑线程安全。
  - **子类将局部变量引用暴露给了其他线程：**例如子类中重写了方法，并在方法中创建线程，那么子类方法和父类方法就会访问共享变量，这种情况是不安全的。(可用private或者final修饰方法，防止子类重写)

- **线程安全**是指多个线程访问该类的同一实例的某个方法时是线程安全的，即它们的每个方法是**原子性**的，但是同一类的多个方法的组合不是原子的。

- **不可打断：**对于synchronized来说，如果一个线程在等待锁，那么结果只有两种，要么它获得这把锁继续执行，要么它就保存等待，即使调用中断线程的方法，也不会生效

- **syncronized** 用的是**Monitor锁(也称监视器，管程，是重量级锁)**，Monitor锁由操作系统提供，包含WaitSet, EntryList, Owner(锁拥有者)三部分。
  - 当对象没有与Monitor关联时，对象的Mark Word 保存hashcode、age等信息；当与Monitor关联时，Mark Word指向Monitor对象。
  - 如果线程t1想要访问syncronized锁住的内容，并且与其关联的Monitor的Owner为null，则owner会指向t1；之后如果线程t2,t3也想访问syncronized内部，但是t1没有释放锁，则t2和t3会进入EntryList，处于阻塞状态；当t1释放锁后，会唤醒EntryList中的线程来竞争锁资源。
  - ![image-20220113161540258](C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220113161540258.png)

- **轻量级锁使用场景：**如果一个对象有多个线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以
  使用轻量级锁来优化，语法仍然是 synchronized
  
  - 使用轻量级锁时，不需要申请互斥量，仅仅将Mark Word中的部分字节CAS更新指向线程栈中的Lock Record(锁记录)，如果更新成功，则轻量级锁获取成功，记录锁状态为轻量级锁；否则，说明已经有线程获得了轻量级锁，目前发生了锁竞争（不适合继续使用轻量级锁），接下来膨胀为重量级锁。
  - 轻量锁的使用过程：
    - 创建锁记录对象Lock Record，每个线程的栈帧中都包含一个锁记录的结构，内部可以存储锁定对象的Mark Word；
    - 让锁记录中Object Refference 指向锁对象，并尝试用cas代替锁对象的Mark Word，同时将Mark Word存入Lock Record中；
    - 如果cas替换成功，锁对象的头部会存储**锁记录的地址**和**状态**00，表示由该线程给对象加锁;
    - 如果cas替换失败，有两种可能：
      - 是自己执行了syncronized锁重入，那么再添加一条Lock Record作为重入的计数(这条记录的值为null);
      - 是其他线程已经持有了该Object的轻量级锁，表明有竞争，则会进入锁膨胀过程。
    - 解锁时(退出syncronized代码块)，如果有取值为null的记录，表示有重入，这时重置锁记录，表示重入计数减一。
    - 如果取值不为null，这个时候使用cas将Mark Word值恢复给对象头。
      - 成功，则解锁成功;
      - 失败，说明轻量级锁进行了锁膨胀或者已经升级为重量级锁，进入重量级锁解锁流程。
  
- **锁膨胀：**如果在尝试加轻量级锁的过程中，由于其它线程已经为这个对象加上了轻量级锁，导致cas替换操作无法成功，这时就要进行锁膨胀(有竞争)，将轻量级锁变成重量级锁。
  - 假设t1线程加轻量级锁失败，由于轻量级锁没有阻塞队列，所以需要为对象申请Monitor锁(重量级锁)，让对象指向重量级锁地址10，t1进入EntryList变成BLOCKED状态。
  - 当t0线程退出同步代码块解锁时，使用cas将Mark Word恢复给对象头会失败，这时会进入重量级解锁流程，即按照Monitor地址找到Monitor对象，设置Owner为null，唤醒EntryList中的BLOCKED线程。
  
- **自旋优化：**重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。
  
  - 自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势。
  - 在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。
  - Java 7 之后不能控制是否开启自旋功能。
  
- **偏向锁：**轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将**线程 ID** 设置到对象的 Mark Word 头，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有。
  
  - 偏向锁默认开启，锁对象的Mark Word后三位为101；
  - 偏向锁默认是延迟的，不会在程序启动时立即生效，可通过加VM参数避免延迟；
  - 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的hashcode、age 都为 0，第一次开启偏向锁 hashcode 时才会赋值；
  - 注意：如果处于偏向锁状态，Mark Word只够存54位的线程ID，不能再存下31位的hashCode，因此当一个可偏向的对象调用hashCode()方法后，会被撤销偏向状态，变为正常状态。
    - 轻量级锁会在锁记录中记录 hashCode
    - 重量级锁会在 Monitor 中记录 hashCode
  - **撤销 ：**当有其它线程使用偏向锁对象时，会将偏向锁升级为**轻量级锁**，解锁后对象将撤销偏向状态。
  - **批量重偏向：**如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象的 Thread ID；当撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至加锁线程。
    - 假设新的线程要访问处于偏向的对象(无竞争)，加锁前处于偏向状态101-加锁后变为轻量级锁000-解锁后变为不可偏向状态001，这样的状态改变发生多次达到一定阈值后，就会将锁重新偏向新的线程。
  - **批量撤销：**当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的
  - **wait()、notify()原理：**
    - 当前线程调用wait()方法后，会进入WaitSet ，变为WAITING状态；
    - BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片；
    - BLOCKED 线程会在 Owner 线程释放锁时唤醒；
    - WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入EntryList 重新竞争；

## 2022年1月15日

------

- **Park() & Unpark()**是**LockSupport类**中的方法

- ```java
  // 暂停当前线程
  LockSupport.park();
  // 恢复某个线程的运行
  LockSupport.unpark(暂停线程对象)
  ```

  - 与Object类中的notify()、wait()方法相比：
    - wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必；
    - park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，就不那么【精确】；
    - park & unpark 可以先 unpark，而 wait & notify 不能先 notify；

- **死锁：**一个线程需要获取多把锁，这时就容易发生死锁
  - t1线程获得了A对象的锁，接下来想要获得B对象的锁;
  - t2线程获得了B对象的锁，接下来想要获得A对象的锁；
  - 定位死锁方法：1.使用jconsole工具；2.使用jps定位进程id，再用jstack定位死锁;
- **活锁：**出现在两个线程互相改变对方的结束条件，最后谁也无法结束的场景；

## 2022年1月16日

------

- **ReentranLock**

  - **可重入**(与syncronized相同);

  - **可打断**：调用lockInteruptibly()方法获得锁时，可用interupt()方法来中断；

  - **锁超时**：能够设置超时时间，tryLock()方法获得锁失败后，将立刻结束不会等待，tryLock(time)可设置等待时间；

  - 支持多个**条件变量**(能够创建多个休息室，可单独叫醒不同休息室)：newCondition()方法创建条件变量，await()方法使当前线程进入休息室等待，signal、signalAll方法用于叫醒休息室里的线程;

  - 可设置为**公平锁**；

  - 基本语法：

    ```java
    ReentranLock lock = new ReentranLock();
    lock.lock();// 获取锁	
    try{
    	// 临界区
    }
    finally{
    	lock.unLock();// 释放锁
    }
    ```

    

## 2022年1月18日

------

- **Java 内存模型**(Java Memory Model, JMM)，定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、CPU 指令优化等，体现在以下三个方面：
  - **原子性：**保证指令不受线程上下文切换的影响；
  - **可见性：**保证指令不受CPU缓存的影响；
  - **有序性：**保证指令不受CPU并行优化的影响；

- **volatile：**用来修饰成员变量和静态成员变量，可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存(syncronized也可以解决共享变量的可见性)
  - synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是属于重量级操作，性能相对更低；
  - 底层实现原理是**内存屏障**，Memory Barrier
    - 对 volatile 变量的写指令后会加入**写屏障**;
    - 对 volatile 变量的读指令前会加入**读屏障**;
  - 如何保证**可见性**：写屏障（sfence）保证在该屏障之前的，对共享变量的改动，都同步到主存当中；而读屏障（lfence）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据；
  - 如何保证**有序性：**写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后；读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前
  - Volatile适合一个线程写，其他线程读的场景；

## 2022年1月20日

------

- 笔记地址 https://blog.csdn.net/m0_37989980/article/details/111657782?spm=1001.2014.3001.5502

- **CAS**(CompareAndSet，也称CompareAndSwap)，在单核 CPU 和多核 CPU 下都能够保证【比较-交
  换】的原子性。

- 结合 **CAS** 和 **volatile** 可以实现**无锁并发**，适用于线程数少、多核 CPU 的场景下。

  - CAS 是基于**乐观锁**的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再重试呗。
  - synchronized 是基于**悲观锁**的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会。
  - CAS 体现的是无锁并发、无阻塞并发，请仔细体会这两句话的意思
    - 因为没有使用 synchronized，所以线程不会陷入阻塞，这是效率提升的因素之一
    - 但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响

- 原子整数（内部通过CAS实现）：AtomicBoolean、AtomicInteger、AtomicLong

  - 以AtomicInteger为例

  - ```java
    public static void updateAndGet(AtomicInteger num, IntUnaryOperator operator){
        while(true){
            int cur =  num.get();
            int next = operator.applyAsInt(cur);
            if(num.compareAndSet(cur,next)) break;// 当cur和最新的cur值相等时才执行更新操作
        }
    }
    ```

- 原子引用：**保证引用类型的共享变量是线程安全的(确保这个原子引用没有引用过别人)**
  - **AtomicReference**：引用类型原子类
  - **AtomicStampedReference：**原子更新带有`版本号`的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，**`可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。`**
    - 同时比较引用地址和版本号，二者相同才会执行更新操作
  - **AtomicMarkableReference：**原子更新带有`标记`的引用类型。该类将 boolean 标记与引用关联起来

## 2022年1月21日

------

- **字段更新器：**保证`多线程`访问`同一个对象的成员变量`时, `成员变量的线程安全性`。
- 利用字段更新器，可以针对对象的某个域（Field）进行原子操作，只能**配合 volatile** 修饰的字段使用，否则会出现异常。
  - AtomicReferenceFieldUpdater:引用类型的属性
  - AtomicIntegerFieldUpdater:整形的属性
  - AtomicLongFieldUpdater:长整形的属性
- **原子累加器(LongAdder)**：性能提升的原因很简单，就是在有竞争时，设置多个累加单元，Therad-0 累加 Cell[0]，而 Thread-1 累加Cell[1]... 最后将结果汇总。这样它们在累加时操作的不同的 Cell 变量，因此减少了 CAS 重试失败，从而提高性能。

- **Unsafe:**(原子类底层通过unsafe实现)提供了非常底层的，操作内存、线程的方法，Unsafe 对象不能直接调用，只能通过反射获得
  - 获取成员变量的偏移量：unsafe.objectFieldOffset()
  - 使用cas替换成员变量的值：unsafet.compareAndSwapInt(obj,offset,prev,next)

## 2022年1月22日

------

- 共享模型之**不可变**

- **不可变类：**保护性拷贝机制，例如String调用substring()方法构造新字符串对象时，会生成新的 char[] value，对内容进行复制 。这种通过创建副本对象来避免共享的手段称之为【保护性拷贝（defensive copy）】

- **享元模式：**当需要重用数量有限的同一类对象时，如果要创建的对象已经有了，就不再创建该对象，而是指向已有对象的地址，从而避免大量创建新的对象。

  - 在JDK中 Boolean，Byte，Short，Integer，Long，Character 等包装类提供了 valueOf 方法，例如Long 的valueOf 会缓存 -128~127 之间的 Long 对象，在这个范围之间会重用对象，大于这个范围，才会新建 Long 对象

  - ```java
    public static Long valueOf(long l) {
        final int offset = 128;
        if (l >= -128 && l <= 127) { // will cache
            return LongCache.cache[(int)l + offset];// 如果范围在-128-127之间，则返回缓冲中的对象
        }
        return new Long(l);// 超出范围才会创建新的对象
    }
    ```

- **无状态：**设计 Servlet 时为了保证其线程安全，都会有这样的建议，不要为 Servlet 设置成员变量，这种没有任何成员变量的类是线程安全的。因为成员变量保存的数据也可以称为状态信息，因此没有成员变量就称之为【无状态】
- **上下文切换：**当线程个数大于CPU核心数时，获取不到CPU时间片的线程会进入阻塞，而当前线程时间片用完的时候会把当前线程运行状态保存下来，下次轮到这个线程运行时恢复之前的状态。
- **自定义线程池：**Thread Pool 存放线程连接、BlockingQueue双向队列，用于存放任务
- ![image-20220122120928514](C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220122120928514.png)
  - core线程用take方法，使用后归还到线程池，最大线程使用poll方法，使用后等待一段时间后进行销毁

## 2022年1月23日

------

- 笔记地址 https://blog.csdn.net/m0_37989980/article/details/112126314?spm=1001.2014.3001.5502
- **ThreadPoolExecutor:** 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量, 这些信息存储在一个原子变量 ctl 中，目的是将**线程池状态**与**线程个数**合二为一，这样就可以用一次 cas 原子操作进行赋值。
  - RUNNING：接收新任务，同时处理任务队列中的任务；
  - SHUTDOWN: 不再接收任务，但会处理阻塞队列中的剩余任务;
  - STOP：中断当前执行的任务，并抛弃阻塞队列中的剩余任务；
  - TIDYING: 任务执行完毕，活动线程为0，即将进入终结状态；
  - TERMINATED: 终结状态；

- 核心线程执行完任务后不回收，会调用await进入等待，当有新任务提交时，会唤醒核心线程。

- **线程池工作方式：**
  
  - 刚开始线程池中没有线程，当有新任务提交时，会创建线程来执行该任务；
  - 当线程数达到核心线程数量时，这个时候有新任务提交时，不再创建新线程，任务将加入等待队列中，等到有线程处于空闲状态时会依次从等待队列中取出任务执行；
  - 当线程数达到核心线程数量时，如果采用的是有界队列，那么当有新任务提交时，会创建救急线程来执行任务；
  - 一旦线程数量达到最大线程数量时，仍然有任务提交，那么会执行拒绝策略。
  
- **newFixedThreadPool：** 适用于任务量已知，相对耗时的任务

  - ```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,// 核心线程数等于最大线程数
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>());// 无界队列
    }
    ```

  - 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间

  - 阻塞队列是无界的，可以放任意数量的任务

- **newCachedThreadPool：** 带缓冲功能的线程池, 整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线程。 适合任务数比较密集，但每个任务执行时间较短的情况.

  - ```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,// 核心线程数为0，最大线程数不限
        60L, TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>());
    }
    ```

  - 核心线程数是 0， 最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着

    - 全部都是救急线程（60s 后可以回收）
    - 救急线程可以无限创建

  - 队列采用了 SynchronousQueue 实现特点是，它没有容量，没有线程来取是放不进去的（一手交钱、一手交货）

- **newSingleThreadExecutor:**

  - ```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1, // 核心线程数和最大线程数都为1
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>()));
    }
    ```

  - **使用场景：**希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程也不会被释放。

  - 使用newSingleThreadExecutor和自己创建一个线程去执行这些任务的区别：

    - 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新建一个线程，保证池的正常工作

  - 和Executors.newFixedThreadPool(1) 的区别：

    - Executors.newSingleThreadExecutor() 线程个数始终为1，不能修改
      - FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 ExecutorService 接口，因此不能调用 ThreadPoolExecutor 中特有的方法
    - Executors.newFixedThreadPool(1) 初始时为1，以后还可以修改
      - 对外暴露的是 ThreadPoolExecutor 对象，可以强转后调用 setCorePoolSize 等方法进行修改

- 线程池**提交任务**的几种方式：
  - void **execute**(Runnable command)：无返回值
  - <T> Future<T> **submit**(Callable<T> task)：提交任务 task，用返回值 Future 获得任务执行结果
  - <T> List<Future<T>> **invokeAll**(Collection<? extends Callable<T>> tasks)：提交 tasks 中所有任务
  - <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) ：提交 tasks 中所有任务，带超时时间
  - <T> T **invokeAny**(Collection<? extends Callable<T>> tasks)：提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消
  - <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)：提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消，带超时时间
- 关闭线程池：
  - **shutdown()**: 线程池状态变为 SHUTDOWN
    - 不会接收新任务
    - 但已提交任务会执行完
    - 此方法不会阻塞调用线程的执行
  - **shutdownNow():** 线程池状态变为 STOP
    - 不会接收新任务
    - 会将队列中的任务返回
    - 并用 interrupt 的方式中断正在执行的任务

## 2022年1月24日

------

- **异步模式之工作流程**：让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现就是线程池，也体现了经典设计模式中的享元模式。
- 解决线程池的饥饿问题：1.增加线程池的大小，但不能从根本上解决问题；2.针对不同的任务类型，采用不同的线程池；
- **创建多少线程池合适：**过小会导致程序不能充分地利用系统资源、容易导致饥饿；过大会导致更多的线程上下文切换，占用更多内存。
  - CPU 密集型运算（N+1）：通常采用 cpu 核数 + 1 能够实现最优的 CPU 利用率，+1 是保证当线程由于页缺失故障（操作系统）或其它原因导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费。
  - I/O 密集型运算（2N)：CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。
    - 经验公式：线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间
    - 最佳线程数 = CPU核心数 * (1/CPU利用率) = CPU核心数 * (1 + (I/O耗时/CPU耗时))，一般可设置为2N

- **newScheduledThreadPool 任务调度线程池**：相当于带有延时功能的FixedThreadPool，线程数固定，任务数多于线程数时，会放入无界队列排队。任务执行完毕，这些线程也不会被释放。用来执行延迟或反复执行的任务。

  - 在『任务调度线程池』功能加入之前，可以使用 java.util.Timer 来实现定时功能，**Timer** 的优点在于简单易用，但由于所有任务都是由同一个线程来调度，因此所有任务都是**串行**执行的，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务。

  - ```java
    ScheduledExecutorService pool = Executors.newScheduledThreadPool(3);
    pool.schedule(()->{ //参数1:callable/runnable接口, 参数2:延迟时间, 参数3：时间单位
        System.out.println("线程1执行");
    },1000, TimeUnit.MILLISECONDS); 
    pool.schedule(()->{
        System.out.println("线程2执行");
    },1000, TimeUnit.MILLISECONDS);
    ```

  - **scheduleAtFixedRate **方法用于实现定时周期性执行任务，从任务执行时开始计时 start ->  start

    - ```java
      pool.scheduleAtFixedRate(()->{//参数1:callable/runnable接口, 参数2:延迟时间, 参数3:任务周期，参数4：时间单位
          System.out.println("线程3执行");
      },1000,2000,TimeUnit.MILLISECONDS);//1秒延迟后开始执行任务，以后每隔2秒执行一次任务
      ```

  - **scheduleWithFixedDelay** 方法：以上次任务结束的时间开始计时 end -> start

- 处理任务执行异常的方式
  - try catch主动捕获
  - 使用Future接收任务返回结果，调用Future的get()方法

## 2022年1月25日

------

- JDK提供的4种线程池**拒绝策略**：
- 中止策略：无特殊场景。
- 丢弃策略：无关紧要的任务（博客阅读量）。
- 弃老策略：发布消息。
- 调用者运行策略：不允许失败场景（对性能要求不高、并发量较小）。
- **AbortPolicy 中止策略**：丢弃任务并抛出RejectedExecutionException异常。这是默认策略
  - 这是线程池默认的拒绝策略，在任务不能再提交的时候，抛出异常，及时反馈程序运行状态。如果是比较关键的业务，推荐使用此拒绝策略，这样子在系统不能承载更大的并发量的时候，能够及时的通过异常发现。
  - 功能：当触发拒绝策略时，直接抛出拒绝执行的异常，中止策略的意思也就是打断当前执行流程.
    使用场景：这个就没有特殊的场景了，但是有一点要正确处理抛出的异常。ThreadPoolExecutor中默认的策略就是AbortPolicy，ExecutorService接口的系列ThreadPoolExecutor因为都没有显示的设置拒绝策略，所以默认的都是这个。但是请注意，ExecutorService中的线程池实例队列都是无界的，也就是说把内存撑爆了都不会触发拒绝策略。当自己自定义线程池实例时，使用这个策略一定要处理好触发策略时抛的异常，因为他会打断当前的执行流程。
- **DiscardPolicy 丢弃策略**：丢弃任务，但是不抛出异常。如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。
  - 使用此策略，可能会使我们无法发现系统的异常状态。建议是一些无关紧要的业务采用此策略。例如，本人的博客网站统计阅读量就是采用的这种拒绝策略。
  - 功能：直接静悄悄的丢弃这个任务，不触发任何动作。
    使用场景：如果你提交的任务无关紧要，你就可以使用它 。因为它就是个空实现，会悄无声息的吞噬你的的任务。所以这个策略基本上不用了。
- **DiscardOldestPolicy 弃老策略**：丢弃队列最前面的任务，然后重新提交被拒绝的任务。
  - 此拒绝策略，是一种喜新厌旧的拒绝策略。是否要采用此种拒绝策略，还得根据实际业务是否允许丢弃老任务来认真衡量。
  - 功能：如果线程池未关闭，就弹出队列头部的元素，然后尝试执行
    使用场景：这个策略还是会丢弃任务，丢弃时也是毫无声息，但是特点是丢弃的是老的未执行的任务，而且是待执行优先级较高的任务。基于这个特性，想到的场景就是，发布消息和修改消息，当消息发布出去后，还未执行，此时更新的消息又来了，这个时候未执行的消息的版本比现在提交的消息版本要低就可以被丢弃了。因为队列中还有可能存在消息版本更低的消息会排队执行，所以在真正处理消息的时候一定要做好消息的版本比较。
- **CallerRunsPolicy 调用者运行策略：**由调用线程处理该任务。
  - 功能：当触发拒绝策略时，只要线程池没有关闭，就由提交任务的当前线程处理。
    使用场景：一般在不允许失败的、对性能要求不高、并发量较小的场景下使用，因为线程池一般情况下不会关闭，也就是提交的任务一定会被运行，但是由于是调用者线程自己执行的，当多次提交任务时，就会阻塞后续任务执行，性能和效率自然就慢了
- **Tomcat线程池**：
  - LimitLatch 用来限流，可以控制最大连接个数，类似 J.U.C 中的 Semaphore 后面再讲
  - Acceptor 只负责【接收新的 socket 连接】
  - Poller 只负责监听 socket channel 是否有【可读的 I/O 事件】
  - 一旦可读，封装一个任务对象（socketProcessor），提交给 Executor 线程池处理
  - Executor 线程池中的工作线程最终负责【处理请求】

- Tomcat 线程池扩展了 ThreadPoolExecutor，行为稍有不同
  - 如果总线程数达到 maximumPoolSize：
    - 这时不会立刻抛 RejectedExecutionException 异常
    - 而是再次尝试将任务放入队列，如果还失败，才抛出 RejectedExecutionException 异常

- **Fork/Join**
  - Fork/Join 是 JDK 1.7 加入的新的线程池实现，它体现的是一种分治思想，适用于**能够进行任务拆分的 cpu 密集型运算**
  - 所谓的任务拆分，是将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解。跟递归相关的一些计算，如归并排序、斐波那契数列、都可以用分治思想进行求解
  - Fork/Join 在分治的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升了运算效率
  - Fork/Join 默认会创建与 **cpu 核心数大小相同**的线程池
  - 使用方式：提交给 Fork/Join 线程池的任务需要继承 **RecursiveTask（有返回值）**或 **RecursiveAction（没有返回值）**

- **AQS（重点）**:AbstractQueuedSynchronizer，是阻塞式锁和相关的同步器工具的框架
- 特点：
  - 用 **state 属性**来表示资源的状态（分**独占模式**和**共享模式**），子类需要定义如何维护这个状态，控制如何获取锁和释放锁
    - getState - 获取 state 状态
    - setState - 设置 state 状态
  - compareAndSetState - cas 机制设置 state 状态
  - 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源
  - 提供了基于 FIFO 的等待队列，类似于 Monitor 的 EntryList
  - 条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 WaitSet

## 2022年1月26日

------

- ReentranLock**非公平锁**原理：
  - 加锁原理：
    - 创建一个继承自AQS的同步器，初始状态state为0，当一个线程尝试用cas加锁后，同步器的OwnerThread设置为当前线程，state修改为1;
    - 如果有其他线程来竞争锁，用cas尝试修改state，如果修改失败，同步器的head创建一个哨兵节点和一个双向链表，并将哨兵节点的后继节点设置为该线程，线程的后继节点指向tail；
    - 然后该线程会尝试cas操作，如果依旧失败，那么哨兵节点的状态设置为-1；
    - 之后该线程又会尝试一次cas操作，如果再次失败，那么该线程会调用park()方法进入阻塞；
    - 之后如果有其他线程竞争锁失败，就会被依次插入到双向链表中；
  - 解锁原理：
    - 当持有锁的线程释放锁之后，state修改为0，OwnerThread设置为null，由哨兵节点唤醒其后继节点指向的线程去竞争锁；
    - 如果解锁成功，那么从链表中移除该线程；
    - 如果竞争失败，被队列以外的线程拿到了锁，那么线程将进入acquireQueued流程，获取锁失败，重新进入park阻塞。

## 2022年1月27日

------

- ReentranLock**可打断**原理：
  - **不可打断模式：**在此模式下，即使它被打断，仍会驻留在 AQS 队列中，一直要等到获得锁后方能得知自己被打断了（继续运行，只是打断标记被设置为true)
  - **可打断模式**：在park过程中，如果被interrupt 会抛出异常，就会退出循环，不再等待锁

- ReentranLock**公平锁**原理：与非公平锁主要区别在于 tryAcquire方法的实现：先检查AQS队列中是否有前驱节点，没有才去竞争（队列里有线程，让队列里的线程先执行，即先到先得）

- ReentrantLock**条件变量**实现原理：每个条件变量对应一个等待队列，其实现类是ConditionObject
- **读写锁**:  读-读 并发 ， 而读-写 和 写-写互斥
  - **ReentrantReadWriteLock:** 当读操作远远高于写操作时，这时候使用 读写锁 让 读-读 可以并发，提高性能。
    - 提供一个 数据容器类 内部分别使用读锁保护数据的 read() 方法，写锁保护数据的 write() 方法
    - 读锁不支持条件变量
    - 重入时不支持升级：即持有读锁的情况下去获取写锁，会导致获取写锁永久等待
    - 重入时支持降级：即持有写锁的情况下去获取读锁

- **缓冲更新策略**
  - 先清除缓冲，再更新数据库
  - ![image-20220127160708809](C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220127160708809.png)
  - 先更新数据库，再清除缓冲
  - ![image-20220127160753307](C:\Users\博子\AppData\Roaming\Typora\typora-user-images\image-20220127160753307.png)

- 解决方法：加读写锁实现**一致性缓存**
- 双重检查锁定（Double Check Lock，DCL）

## 2022年1月28日

------

- **StampedLock:** 该类自 JDK 8 加入，是为了进一步**优化读性能**，它的特点是在使用读锁、写锁时都必须**配合【戳】使用**

  - 加解读锁

    ```java
    long stamp = lock.readLock();
    lock.unlockRead(stamp);
    ```

  - 加解写锁

    ```java
    long stamp = lock.writeLock();
    lock.unlockWrite(stamp);
    ```

- **乐观读**：StampedLock 支持 tryOptimisticRead() 方法（乐观读），读取完毕后需要做一次 **戳校验** 如果校验通过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。

- 注意:

  - StampedLock **不支持条件变量**
  - StampedLock **不支持可重入**

- **Semaphore：**信号量，用来**限制能同时访问共享资源的线程上限。**

- 使用方法

  ```java
  public static void main(String[] args) {
      // 1. 创建 semaphore 对象
      Semaphore semaphore = new Semaphore(3);
      // 2. 10个线程同时运行
      for (int i = 0; i < 10; i++) {
          new Thread(() -> {
              // 3. 获取许可
              try {
                  semaphore.acquire();// 如果获取失败，将进入阻塞等待（可打断）
              } catch (InterruptedException e) {
              e.printStackTrace();
              }
              try {
                  log.debug("running...");
                  sleep(1);
                  log.debug("end...");
              } finally {
              // 4. 释放许可
              semaphore.release();
              }
          }).start();
      }
  }
  ```

- **Semaphore应用：**限制对共享资源的使用

  - 使用 Semaphore 限流，在访问高峰期时，让请求线程阻塞，高峰期过去再释放许可，当然它只适合限制单机线程数量，并且仅是限制线程数，而不是限制资源数（例如连接数，请对比 Tomcat LimitLatch 的实现）
  - 用 Semaphore 实现简单连接池，对比『享元模式』下的实现（用wait notify），性能和可读性显然更好，注意下面的实现中线程数和数据库连接数是相等的

- **CountdownLatch(倒计时锁)：**用来进行线程同步协作，等待所有线程完成倒计时。

  - 其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一
  - 例如 CountDownLatch latch = new CountDownLatch(3) 创建latch对象后，线程1调用了await方法，需要其他线程调用countDown方法，并且计数从3减为0后，线程1才会唤醒，继续执行。
  - 相比于join()的优势：相对来说join是更为底层的方法，而countdownlatch属于高级的API，由于在实际开发中会经常使用线程池，而线程池中核心线程一般不会释放，因此join方法等不到运行结束，而countdownlatch则可以适用于大多数场景。

- **CyclicBarrier:** 循环栅栏，用来进行线程协作，等待线程满足某个计数。构造时设置『计数个数』，每个线程执行到某个需要“同步”的时刻调用 await() 方法进行等待，当等待的线程数满足『计数个数』时，继续执行.
  - CyclicBarrier解决了CountdonwLatch不能复用的问题

## 2022年1月29日

------

- **线程安全集合类**：

  - 遗留的安全集合：Hashtable、Vector
  - 修饰的安全集合：SynchronizedMap、SynchronizedList 使用了Collections的方法修饰
  - JUC安全集合：Blocking类、CopyOnWrite类、Concurrent类

- 重点介绍 java.util.concurrent.* 下的线程安全集合类，可以发现它们有规律，里面包含三类关键词：
  Blocking、CopyOnWrite、Concurrent

  - Blocking 大部分实现基于锁，并提供用来阻塞的方法
  - CopyOnWrite 之类容器修改开销相对较重
  - **Concurrent 类型的容器**
    - 内部很多操作使用 cas 优化，一般可以提供较高吞吐量
    - **弱一致性**
      - **遍历时弱一致性**，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍
        历，这时内容是旧的
      - **求大小弱一致性**，size 操作未必是 100% 准确
      - **读取弱一致性**

  - 对于非安全容器来讲, 遍历时如果发生了修改，使用 fail-fast 机制也就是让遍历立刻失败，抛出
    ConcurrentModificationException，不再继续遍历

- **HashMap**:JDK7中相同哈希值的用头插法插入链表，JDK8则采用尾插法

  - 当元素超过容量的3/4时会进行扩容操作 负载因子0.75
  - JDK7在多线程扩容出现**死链问题**：一个线程在扩容时把链表节点反转了，而另一个线程在扩容时正好在前一个节点，形成了环形链表。
  - 因为T1执行完扩容之后B节点的下一个节点是A，而T2线程指向的首节点是A，第二个节点是B，这个顺序刚好和T1扩完容完之后的节点顺序是相反的。**T1执行完之后的顺序是B到A，而T2的顺序是A到B，这样A节点和B节点就形成死循环了**，这就是HashMap死循环导致的原因。
  - HashMap死循环的常用解决方案有以下3个：
    - 使用线程安全容器ConcurrentHashMap替代（推荐使用此方案）。
    - 使用线程安全容器Hashtable替代（性能低，不建议使用）。
    - 使用synchronized或Lock加锁HashMap之后，再进行操作，相当于[多线程](https://www.zhihu.com/search?q=多线程&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2314917909})排队执行（比较麻烦，也不建议使用）。

  - 总结：HashMap死循环发生在JDK1.7版本中，形成死循环的原因是HashMap在JDK1.7使用的是头插法，头插法+链表+多线程并发+HashMap扩容，这几个点加在一起就形成了HashMap的死循环，解决死锁可以采用线程安全容器ConcurrentHashMap替代。

    

## 2022年1月31日

------

- JDK1.7中，ConcurrentHashMap特点：采用分段锁思想，对每一个桶使用Segment加锁，Segment实现了重入锁ReentrantLock，Segment中包含HashEntry(数组+链表)，存放键值对数据，相比于Hashtable用synchronized锁住全局，Segment只锁住了桶，支持多个线程同时访问，能有效减小锁粒度，提高程序的并发度。
- JDK1.8中，ConcurrentHashMap取消了Segment分段锁，而是采用cas+synchronized来锁住Node<K,V>，保证线程安全。
- JDK1.8中，HashMap桶中存放的数据超过了阈值8时，会先判断数组长度是否大于64，如果小于64会先进行一次扩容，当数组长度达到64后，才会将链表结构转换为红黑树；如果进行了删除元素操作导致数组长度小于64后，红黑树将再转换为链表。

## 2022年2月1日

------

- JDK1.8中，ConcurrentHashMap的哈希表Node[] table采用懒惰初始化创建，只有当键值对put进来时才会创建，初始化table过程使用cas，无需synchronized
- 当put操作的桶下标冲突时，需要用synchronized锁住链表头节点
- Java 8 数组（Node） +（ 链表 Node | 红黑树 TreeNode ） 以下数组简称（table），链表简称（bin）
  - 初始化，使用 cas 来保证并发安全，懒惰初始化 table
  - 树化，当 table.length < 64 时，先尝试扩容，超过 64 时，并且 bin.length > 8 时，会将链表树化，树化过程会用 synchronized 锁住链表头
  - put，如果该 bin 尚未创建，只需要使用 cas 创建 bin；如果已经有了，锁住链表头进行后续 put 操作，元素添加至 bin 的尾部
  - get，无锁操作仅需要保证可见性，扩容过程中 get 操作拿到的是 ForwardingNode 它会让 get 操作在新
    table 进行搜索
  - 扩容，扩容时以 bin 为单位进行，需要对 bin 进行 synchronized，但这时妙的是其它竞争线程也不是无事可做，它们会帮助把其它 bin 进行扩容，扩容时平均只有 1/6 的节点会把复制到新 table 中
  - size，元素个数保存在 baseCount 中，并发时的个数变动保存在 CounterCell[] 当中。最后统计数量时累加即可
- JDK1.7 ConcurrentHashMap ：维护了一个 segment 数组，每个 segment 对应一把锁
  - 优点：如果多个线程访问不同的 segment，实际是没有冲突的，这与 jdk8 中是类似的
  - 缺点：Segments 数组默认大小为16，这个容量初始化指定后就不能改变了，并且不是懒惰初始化

- **LinkedBlockingQueue** 
  - **入队：**尾部直接插入
  - **出队：**删除头节点，将第二个节点修改为头节点，返回第二个节点的值，然后将其设置为null，此时的头节点就变成了dummy哨兵节点。
  - **加锁分析：**高明之处在于用了两把锁(ReentrantLock)和 dummy 节点
    - 用一把锁，同一时刻，最多只允许有一个线程（生产者或消费者，二选一）执行
    - 用两把锁，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行
      - 消费者与消费者线程仍然串行
      - 生产者与生产者线程仍然串行
  - LinkedBlockingQueue 与 ArrayBlockingQueue 的**性能比较**
    - Linked 支持有界，Array 强制有界
    - Linked 实现是链表，Array 实现是数组
    - Linked 是懒惰的，而 Array 需要提前初始化 Node 数组
    - Linked 每次入队会生成新 Node，而 Array 的 Node 是提前创建好的
    - Linked 两把锁，Array 一把锁
- **ConcurrentLinkedQueue** 
  - ConcurrentLinkedQueue 的设计与 LinkedBlockingQueue 非常像，也是两把【锁】，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行
  - dummy 节点的引入让两把【锁】将来锁住的是不同对象，避免竞争
  - 只是这【锁】使用了 cas 来实现

- **CopyOnWriteArrayList**
  - CopyOnWriteArraySet 是它的马甲， 底层实现采用了 **写入时拷贝** 的思想，增删改操作会将底层数组拷贝一份，更改操作在新数组上执行，这时	不影响其它线程的**并发读**，**读写分离**
  - 写操作加可重入锁，其它读操作并未加锁
  - 读-读 和 读-写 并发，只有写-写 互斥

