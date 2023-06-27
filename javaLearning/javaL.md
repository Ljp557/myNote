# 1. 基础

## 1. 作用域

（1） ![截屏2022-11-23 上午7.15.23](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2022-11-23 上午7.15.23.png)

（2） java,c if和for里面定义的变量都是==局部变量==
		  python，php 是全局变量

## 2. [字符串是引用类型，但是不可变](https://blog.csdn.net/qq_25744257/article/details/81665571)

（1） '+' 虽然可以直接拼接字符串，但是，在循环中，每次循环都会创建新的字符串对象，然后扔掉旧的字符串。这样，绝大部分字符串都是临时对象，不但浪费内存，还会影响GC效率。

为了能高效拼接字符串，Java标准库提供了`StringBuilder`，它是一个可变对象，可以预分配缓冲区，这样，往`StringBuilder`中新增字符时，不会创建新的临时对象：

（2） [string，stringbuffer，stringbuilder的区别以及使用场景](https://blog.csdn.net/weberhuangxingbo/article/details/117550035)

（3）

# 2. 多线程

## 1. [run()和start()的区别](https://blog.csdn.net/w15558056319/article/details/118208957)

## 2. runaable和Thread



## 3. 线程停止

（1）自己定义标志flag，重写stop方法，不要使用jdk不推荐的方法。或者设置标志位`boolean running`是一个线程间共享的变量。线程间共享变量需要使用`volatile`关键字标记，确保每个线程都能读取到更新后的变量值。

```public class Main {
    public static void main(String[] args)  throws InterruptedException {
        HelloThread t = new HelloThread();
        t.start();
        Thread.sleep(1);
        t.running = false; // 标志位置为false
    }
}

class HelloThread extends Thread {
    public volatile boolean running = true;
    public void run() {
        int n = 0;
        while (running) {
            n ++;
            System.out.println(n + " hello!");
        }
        System.out.println("end!");
    }
}
```

（2） java代理模式，静态代理一个代理对象只能代理一个目标对象

## 4. 同步方法和同步块

（1）synchronized放在方法声明，同步方法锁的是this对象，若this对象和实际增删改的对象不一致，还是会出现不安全问题

（2）synchronized放在增删改的对象前面，同步块

（3）wait(),notify()管程法，信号灯法实现生产者消费者

# 3. 网络编程



![截屏2022-12-09 上午7.54.12](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2022-12-09 上午7.54.12.png)

## 1. socket.shutdomnOutput()用来改变IO流的方向

例如：客户端在使用输出流传输数据，服务端收到数据后想要发送接收完毕等信息给客户端，此时客户端需要使用shutdown，否则客户端会一直占用输出流从而导致服务端无法使用输出流传输数据。



## [JVM内存机制](https://blog.csdn.net/D812359/article/details/124392532)

# 面试题目

## 1 JAVA基础

### 1.1 [JVM对象池](https://www.cnblogs.com/DreamSea/archive/2011/11/20/2256396.html)

### 1.2 类加载

static在类链接阶段赋初值（0），在初始化阶段赋值，static+final等常量在链接阶段直接赋值了，调用常量不会引起类的初始化。

![截屏2023-03-06 上午7.15.23](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-06 上午7.15.23.png)

### 1.3 反射的意义

看到这里会觉得反射很麻烦，没有体现出它的价值。接下来看个案例，就会一目了然了。这个案例就是写一个类，在不改变该类任何代码的前提下，可以帮助我们创建任意类的对象，并且执行其中任意方法。

步骤：

```
 1. 将需要创建的对象的全类名和需要执行的方法定义在配置文件
 2. 在程序中加载读取配置文件
 3. 使用反射技术来加载类文件进入内存
 4. 创建对象
 5. 执行方法
12345
```

Student类：

```java
@NoArgsConstructor
public class Student {
    public void sleep(){
        System.out.println("睡觉");
    }
}
123456
```

pro.properties配置文件：

```properties
className=com.lyp.domain.Student
methodName=sleep
12
```

ReflectTest类：

```java
public class ReflectTest {
    public static void main(String[] args) throws Exception {
        // 前提：不能修改该类的代码
//        Student s = new Student();
//        s.sleep();

        // 1.加载配置文件
        // 1.1 创建Properties对象
        Properties pro = new Properties();
        // 1.2 加载配置文件，转化为一个集合
        // 1.2.1 获取class目录下的配置文件
        ClassLoader classLoader = ReflectTest.class.getClassLoader();//通过类加载器将字节码文件加载进内存
        InputStream is = classLoader.getResourceAsStream("pro.properties");
        pro.load(is);

        // 2.获取配置文件中定义的数据
        String className = pro.getProperty("className");
        String methodName = pro.getProperty("methodName");

        // 3.加载该类进内存
        Class cls = Class.forName(className);
        // 4.创建对象
        Object o = cls.newInstance();
        // 5.获取方法对象
        Method method = cls.getMethod(methodName);
        // 6.执行方法
        method.invoke(o);

    }
}
123456789101112131415161718192021222324252627282930
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/5a6a5302a2f040cbb32872dad1c4e890.png)

将需要生成的对象的信息放在配置文件中，这样我们不需要去修改ReflectTest的代码，只需要修改配置文件中className，methodName即可；如果不使用配置文件，就需要在ReflectTest类中写死，Student s = new Student(); s.sleep()；那如果想要生成别的对象每次都需要去找到ReflectTest类，修改Student为别的类。**使用反射后，增加了程序的灵活性，避免将代码写死，降低了耦合性**。在框架的设计思想里用到的就是反射

### 1.4 多态的意义

~~~java
public class PersonTest {
  // 形参只要写Person
    public static void speak(Person p){
        p.speak();
    }
    public static void main(String[] args) {
        //将不同子类对象作为形参
        PersonTest.speak(new Worker());
        PersonTest.speak(new Dancer());
    }
}
~~~

### 1.5 重写equals方法还要重写hashcode

因为类似set，map等在做对象插入的时候会有重复性检验，首先检验的就是hashcode这样可以提高效率，如果重写了equals没有重写hashcode会出现我们认为两个对象是相等的（内容），但是hashcode不同，java直接判定为不同。

### 1.5 跨域问题，可用过滤器或者加入注解@crossorigin

### 1.6 为什么要使用包装类:

- 作为 和基本数据类型对应的类 类型存在，方便涉及到对象的操作。
- 包含每种基本数据类型的相关属性如最大值、最小值等，以及相关的操作方法。

### 1.7乐观锁和悲观锁

悲观锁认为对于同一个数据的并发操作一定是会发生修改的，采取加锁的形式，悲观地认为，不加锁的并发操作一定会出问题。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。Java中Synchronized和ReentrantLock等独占锁就是悲观锁思想实现的。

乐观锁

乐观锁正好和悲观锁相反，它获取数据的时候，并不担心数据被修改，每次获取数据的时候也不会加锁，只是在更新数据的时候，通过判断现有的数据是否和原数据一致来判断数据是否被其他线程操作，如果没被其他线程修改则进行数据更新，如果被其他线程修改则不进行数据更新。Lock 是乐观锁的典型实现案例。

典型案例

(1)悲观锁：synchronized 关键字和 Lock 接口相关类

Java 中悲观锁的实现包括 synchronized 关键字和 Lock 相关类等，我们以 Lock 接口为例，例如 Lock 的实现类 ReentrantLock，类中的 lock() 等方法就是执行加锁，而 unlock() 方法是执行解锁。处理资源之前必须要先加锁并拿到锁，等到处理完了之后再解开锁，这就是非常典型的悲观锁思想。

(2)乐观锁：原子类

乐观锁的典型案例就是原子类，例如 AtomicInteger 在更新数据时，就使用了乐观锁的思想，多个线程可以同时操作同一个原子变量。

(3)大喜大悲：数据库

数据库中同时拥有悲观锁和乐观锁的思想。例如，我们如果在 MySQL 选择 select for update 语句，那就是悲观锁，在提交之前不允许第三方来修改该数据，这当然会造成一定的性能损耗，在高并发的情况下是不可取的。相反，我们可以利用一个版本 version 字段在数据库中实现乐观锁。在获取及修改数据时都不需要加锁，但是我们在获取完数据并计算完毕，准备更新数据时，会检查版本号和获取数据时的版本号是否一致，如果一致就直接更新，如果不一致，说明计算期间已经有其他线程修改过这个数据了，那我就可以选择重新获取数据，重新计算，然后再次尝试更新数据。

CAS介绍

Java中的乐观锁大部分都是通过CAS(CompareAndSwap，比较并交换)操作实现的，CAS是一个多线程同步的原子指令，CAS操作包含三个重要的信息，即内存位置、预期原值和新值。如果内存位置的值和预期的原值相等的话，那么就可以把该位置的值更新为新值，否则不做任何修改。

补充说明: 虽然ReentrantLock也是通过CAS实现的,但是是悲观锁。

缺点:ABA问题.

CAS可能会造成ABA的问题，ABA问题指的是，线程拿到了最初的预期原值A，然而在将要进行CAS的时候，被其他线程抢占了执行权，把此值从A变成了B，然后其他线程又把此值从B变成A，然而此时的 A 值已经并非原来的 A 值了，但最初的线程并不知道这个情况，在它进行 CAS 的时候，就会误认为它从来没有被修改过，只对比了预期原值为 A 就进行了修改，这就造成了 ABA 的问题。

以警匪剧为例，假如某人把装了100W现金的箱子放在了家里，几分钟之后要拿它去赎人，然而在趁他不注意的时候，进来了一个小偷，用空箱子换走了装满钱的箱子，当某人进来之后看到箱子还是一模一样的，他会以为这就是原来的箱子，就拿着它去赎人了，这种情况肯定有问题，因为箱子已经是空的了，这就是 ABA 的问题。

JDK在1.5时提供了AtomicStampedReference类也可以解决ABA的问题，此类维护了一个“版本号”Stamp，每次在比较时不止比较当前值还比较版本号，这样就解决了 ABA 的问题。

悲观锁乐观锁优缺点:

优点:

悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，这样的开销往往无法承受。而乐观锁机制避免了长事务中的数据库加锁开销，大大提升了大并发量下的系统整体性能表现。

缺点：

需要注意的是，乐观锁机制往往基于系统中的数据存储逻辑，因此也具备一定的局限性，如在上例中，由于乐观锁机制是在我们的系统中实现，来自外部系统的用户余额更新操作不受我们系统的控制，因此可能会造成脏数据被更新到数据库中。在系统设计阶段，我们应该充分考虑到这些情况出现的可能性，并进行相应调整(如将乐观锁策略在数据库存储过程中实现，对外只开放基于此存储过程的数据更新途径，而不是将数据库表直接对外公开)。

使用场景

有一种说法认为，悲观锁由于它的操作比较重量级，不能多个线程并行执行，而且还会有上下文切换等动作，所以悲观锁的性能不如乐观锁好，应该尽量避免用悲观锁，这种说法是不正确的。因为虽然悲观锁确实会让得不到锁的线程阻塞，但是这种开销是固定的。悲观锁的原始开销确实要高于乐观锁，但是特点是一劳永逸，就算一直拿不到锁，也不会对开销造成额外的影响。反观乐观锁虽然一开始的开销比悲观锁小，但是如果一直拿不到锁，或者并发量大，竞争激烈，导致不停重试，那么消耗的资源也会越来越多，甚至开销会超过悲观锁。所以，同样是悲观锁，在不同的场景下，效果可能完全不同。

(1) 悲观锁适合用于并发写入多、临界区代码复杂、竞争激烈等场景，这种场景下悲观锁可以避免大量的无用的反复尝试等消耗。

(2) 乐观锁适用于大部分是读取，少部分是修改的场景，也适合虽然读写都很多，但是并发并不激烈的场景。在这些场景下，乐观锁不加锁的特点能让性能大幅提高。

### 1.8 sql语句执行慢的原因：

https://baijiahao.baidu.com/s?id=1716929241430590477&wfr=spider&for=pc

### 1.9 事务和线程同步：

1. 加事务的方式：https://blog.csdn.net/csdnlaiyanqi/article/details/121478081

2. redis事务：operation.multi();

 3. 项目什么地方用到了事务：点赞时要在redis中的被点赞用户set中添加当前用户的id，并且点赞数加一，这两个操作要保持原子性；关注，评论类似。

4. 什么地方用到了redis：存储点赞用户和点赞数（使用set存储点赞用户，value存储点赞数），关注；验证码，登录凭证，缓存用户信息;统计; ps：redis默认开启持久化，对于用作缓存功能需要设定存储时间。

5. 事务和同步的关系：

事务为保证一个操作的原子性而设置的，一个事务必定包含多个操作，多个操作再逻辑上要保证完整一致，如果中间只要有一个操作失败，那么事务必须回滚，必须回到整个操作的初始状态

多线程为了提高应用的执行效率而设置的，多个线程可以做同样的事情或不同的事情，单个线程只能处理1个客户请求，那么多线程就可以同时处理多个请求。每一个线程处理的业务涉及到多个操作，如果有一致性的要求，那么必须介入事务

同步是为了解决多线程使用过程中，使用相同资源导致数据不一致而引入的，使用了同步机制，那么多个线程在访问同一资源时，必须等到另一个线程使用完毕，释放了这个资源，其它的线程才有机会使用。

6. 线程同步的方式：

   - sychronized修饰静态方法，类方法，代码块
   - volatile+CAS
   - ReentrantLock
   - AtomicInteger

7. 为什么选择redis，redis的优势？

8. 为什么用threadlocal存储当前用户，还可以用什么方法？

   ![截屏2023-03-27 上午7.07.51](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-27 上午7.07.51.png)

   访问每个页面都有interceptor，其中根据cookie存储的ticketid，去缓存中找到ticket，查询ticket存在与否和有效期判断是否有效，根据ticket信息通过userservice找到user，交给threadlocal持有。在这里还要进行security注册。

   ***TreadLocal的源码：javaguide***

   **每个`Thread`中都具备一个`ThreadLocalMap`，而`ThreadLocalMap`可以存储以`ThreadLocal`为 key ，Object 对象为 value 的键值对。**

   原因：web访问是多线程访问，需要线程隔离

   还可以用session存储，通过cookie存储的id取到当前用户，但是seesion是存储在服务器上的，在分布式的情况下会有很多个服务器在取值时会有麻烦

9. ***项目中用到线程同步的地方***

   Redis stringRedisTemplate.watch(key)的源码，看源码的原因：operations.opsForValue().decrement(userLikeKey)操作会不会线程不安全？

   stringRedisTemplate.watch()实现了乐观锁，监控key过程中如果有别的线程对key做出修改，Object rs = operations.exec();则rs不为空;参考秒杀系统

   ```java
   ```

   

### 2.0 SpringMvc中的Interceptor的实现原理:

(拦截器原理，含源码)[https://blog.csdn.net/zhangyong01245/article/details/99986214)]

### 2.1 Spring IOC

1. 作用：
   - 将控制权交给ioc容器，可以专注业务的开发不用管理对象的创建。
   - 降低代码耦合度，原来客户调用service层，service层new一个dao层的对象，如果需要使用另个dao接口那么需要对service层的new代码进行更改，耦合度较高，如果嵌套的new很多那么每次改变一个功能很麻烦。使用ioc之后，直接依赖注入，不需要更改service层的代码。用户直接修改xml配置文件就可以实现功能的变化。
   - @autowired注入一个对象, 可以重复使用. 打个比方, 如果业务层有两个类都用到了用一个类, 此时使用new的话得创建两个对象实例, 但是使用@autowired注入的对象可以在不同的类之间使用, 这就避免创建重复的对象, 也就减轻了垃圾回收(GC)的压力

### 2.2 sychronized详细过程

![截屏2023-04-12 上午7.51.06](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-04-12 上午7.51.06.png)

锁升级作用和过程：当不存在线程竞争时直接比较对象头中的线程ID即可获得偏向锁（即多次锁定该对象的是同一个锁），如果线程ID不同，那么判断原持有线程是否还需要该对象，如果不需要则修改线程ID为当前线程，如果还需要则升级为轻量级锁（说明存在线程竞争了）；当前锁进行锁自旋等待轻量级锁的释放，如果达到一定次数（说明线程竞争激烈）那么就膨胀成重量级锁（重量级锁会引入monitor指向一个阻塞队列，阻塞和唤醒线程都是需要高昂的开销的），如果锁自旋成功那么就可以避免重量级锁的大量开销了。

偏向锁-》轻量级锁：出现线程竞争，线程2 CAS必定替换不成功，此时检查线程1状态，如果线程1不需要这个对象那么会设置对象mark word线程号为0表示无锁可偏，此时线程2会CAS成功获得偏向锁，否则锁升级。

ps ：从偏向锁的获取过程可以看到，等到竞争出现的时候才会释放。如果没有出现竞争，它不会 去改变Mark Word的相关字段。就算是线程已经执行完同步代码块，不需要加锁了，也不会 去修改对象头，那个锁依旧存在，依旧保持偏向。只是在其他线程需要偏向，出现了竞争的时候会进行判断，如果以前偏向的线程不需要了， 那么对象首先会被设置为匿名偏向，然后CAS替换尝试加锁。如果以前偏向的线程还需要加 锁，升级为轻量级锁。

这里说一下轻量级锁释放失败是就证明锁升级的原理，因为之前 mark word 指向的是本线程的指针，这个是 cas 期望的值，但是被其他线程更改为了指向互斥量的对象了，cas 就失败，就证明升级为了重量级锁

https://cloud.tencent.com/developer/article/2074879

锁自旋https://blog.csdn.net/vincent_wen0766/article/details/108558656

### 2.3 Object类的方法

https://blog.csdn.net/qq_45259214/article/details/119087298

### 2.4多线程和多进程的区别，使用场景

https://blog.csdn.net/abcdefg12345666/article/details/111739157

### 2.5 保证多线程线程安全的方法

1. 加锁：对于共享资源，使用锁机制保证在同一时刻只有一个线程可以访问。Java提供了多种锁机制，如synchronized、ReentrantLock等。
2. 原子操作：原子操作是指不可被中断的操作，它能够保证在同一时刻只有一个线程可以访问。Java提供了一些原子类，如AtomicInteger、AtomicLong、AtomicReference等。
3. 线程本地存储：使用ThreadLocal机制将共享资源保存在每个线程的本地存储中，避免了多个线程之间的竞争。
4. 避免共享资源：尽量避免多个线程同时访问同一个共享资源，可以通过将共享资源分解为多个独立的部分，或者使用消息传递等方式。
5. 使用并发容器：Java提供了一些线程安全的容器，如ConcurrentHashMap、CopyOnWriteArrayList等，可以避免多线程访问时的竞争问题。

## 3 数据库

### 3.1 索引

- MySQL Innodb的主键索引是[聚簇索引](https://so.csdn.net/so/search?q=聚簇索引&spm=1001.2101.3001.7020)，也就是索引的叶子节点存的是整个单条记录的所有字段值，不是主键索引的就是非聚簇索引，非聚簇索引的叶子节点存的是主键字段的值。回表：假设表tbl有a,b,c三个字段，其中a是主键，b上建了索引，然后编写sql语句SELECT * FROM tbl WHERE a=1这样不会产生回表，因为所有的数据在a的索引树中均能找到。
  SELECT * FROM tbl WHERE b=1这样就会产生回表，因为where条件是b字段，那么会去b的索引树里查找数据，但b的索引里面只有a,b两个字段的值，没有c，那么这个查询为了取到c字段，就要取出主键a的值，然后去a的索引树去找c字段的数据。查了两个索引树，这就叫回表。

  [索引覆盖](https://so.csdn.net/so/search?q=索引覆盖&spm=1001.2101.3001.7020)就是查这个索引能查到你所需要的所有数据，不需要去另外的数据结构去查。其实就是不用回表。就是不是必须的字段就不要出现在SELECT里面。或者b,c建联合索引。但具体情况要具体分析，索引字段多了，存储和插入数据时的消耗会更大。这是个平衡问题

- 最左前缀，索引下推；

  索引下推：如果需要查询的字段在索引字段中，那么直接在存储引擎中筛选符合条件的记录，不需要根据找到的主键记录回标查询匹配的记录，减少回表的次数。

# 腾讯

1. 函数执行的过程

   第一步：函数调用
   1、将函数调用语句下一条语句的地址保存到在栈中，以便函数调用完成后返回。（将函数放到栈空间中称为压栈）。

   2、对实参表从后向前，一次计算出实参的值，并且将值压栈。

   3、跳转到函数体处。

   第二步：函数体执行

   4、如果函数体中定义了变量，将变量压栈

   5、将每一个形参以栈中对应的实参值取代，执行函数体的功能体。

   6、将函数体中的变量、保存到栈中的实参值，依次从栈中取出，释放栈空间（出栈）。

   第三步：返回

   7、返回过程执行的是函数体中的return语句。其过程是从栈中取出刚开始调用函数时压入的地址，跳转到函数的下一条语句。当return语句不带有表达式时，按照保存的地址返回，当return语句带有表达式时，将计算出的return表达式的值保存起来，然后再返回。

1. 三次四次握手，timewait过多是什么原因怎么解决？https://www.cnblogs.com/gaoyanbing/p/16406873.html

   https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247520638&idx=5&sn=aaf6970805bb6ad231f5969524c668d4&chksm=fbb11880ccc691969585ac8bdaeabe9b2966ecc9019e2247008a7f613e3a513c313842a71c53&scene=27

1. IO多路复用，epoll

   https://www.cnblogs.com/12345huangchun/p/10066840.html

   epoll：1. 监听多个socket 2. 不同于轮询，为每个监听设置一个回调函数效率更高。

   1，epoll默认是水平触发

   2，水平触发通俗来讲：只要有数据，epoll_wait函数就一直返回；边沿触发通俗来讲：只有[socket](https://so.csdn.net/so/search?q=socket&spm=1001.2101.3001.7020)状态发生变化，epoll_wait函数才会返回。

   3，水平触发优、缺点及应用场景：

   优点：当进行socket通信的时候，保证了数据的完整输出，进行IO操作的时候，如果还有数据，就会一直的通知你。

   缺点：由于只要还有数据，内核就会不停的从内核空间转到用户空间，所有占用了大量内核资源，试想一下当有大量数据到来的时候，每次读取一个字节，这样就会不停的进行切换。内核资源的浪费严重。效率来讲也是很低的。

   应用场景：

   4，边沿触发优、缺点及应用场景：

   优点：每次内核只会通知一次，大大减少了内核资源的浪费，提高效率。

   缺点：不能保证数据的完整。不能及时的取出所有的数据。

   应用场景：处理大数据。使用non-block模式的socket。


   # 美团

   1. 排序算法的时间复杂度（最好，最坏，平均）

      **稳定性**：稳定排序算法会让原本有相等键值的纪录维持相对次序。也就是如果一个排序算法是稳定的，当有两个相等键值的纪录R和S，且在原本的列表中R出现在S之前，在排序过的列表中R也将会是在S之前。

      ![截屏2023-05-23 上午7.36.07](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-05-23 上午7.36.07.png)

   2. MD5加密原理？加盐？密码是以什么形式存储到数据库的？

      - md5是一种信息摘要算法，一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值，用来确保信息传输完整一致性。

        [MD5加密算法](https://so.csdn.net/so/search?q=MD5加密算法&spm=1001.2101.3001.7020)是不可逆的。

        [MD5算法](https://so.csdn.net/so/search?q=MD5算法&spm=1001.2101.3001.7020)是单向散列算法的一种。单向散列算法也称为HASH算法，是一种将任意长度的信息压缩至某一固定长度（称之为消息摘要）的函数(**该压缩过程不可逆**)。在MD5算法中，这个摘要是指将任意数据映射成一个128位长的摘要信息。并且其是不可逆的，即从摘要信息无法反向推演中原文。MD5算法最终生成的是一个128位长的数据，从原理上说，有2^128种可能，这是一个非常巨大的数据，约等于3.4乘10的38次方，虽然这个是个天文数字，但是世界上可以进行加密的数据原则上说是无限的，因此是可能存在不同的内容经过MD5加密后得到同样的摘要信息，但这个碰中的概率非常小。

        注意：md5值不是唯一的，也就是一个原始数据，只对应一个md5值，但是一个md5值，可能对应多个原始数据。且md5值是可以被破解的，例撞库破解。

        **撞库****破解**

        关于撞库，这是概率比较低的解密方法，原理是：通过建立大型的数据库，把日常的各种句子通过md5加密成为密文，不断积累更新大量句子，放在庞大的数据库里；然后，有人拿了别人的密文，想查询真实的密码，就需要把密文拿到这个数据库的网站（免费MD5加密解密：[md5在线加密解密](https://md5.cn/)）去查询。

      - 就是一个随机生成的字符串。我们将盐与原始密码连接/拼接（concat）在一起（放在前面或后面都可以），然后将concat后的字符串加密。Salt这个值是由系统随机生成的，并且只有系统知道。即便两个用户使用了同一个密码，由于系统为它们生成的salt值不同，散列值也是不同的。

        我们知道，如果直接对密码进行散列，那么黑客可以对通过获得这个密码散列值，然后通过查散列值字典（例如MD5密码破解网站），得到某用户的密码。

        加Salt可以一定程度上解决这一问题。所谓加Salt方法，就是加点“佐料”（Salt这个单词就是盐的意思）。其基本想法是这样的：当用户首次提供密码时（通常是注册时），由系统自动往这个密码里撒一些“佐料”，然后再散列。而当用户登录时，系统为用户提供的代码撒上同样的“佐料”，然后散列，再比较散列值，已确定密码是否正确。

        这样也就变成了将密码+自定义的盐值来取MD5。但是如果黑客拿到了你的固定的盐值，那这样也不安全了。所以比较好的做法是用随机盐值。用户登陆时再根据用户名取到这个随机的盐值来计算MD5。

   3. hashmap的结构是什么？读，插入时间复杂度是多少？红黑树与二叉平衡树各自的优势？

      - hashmap根据key查询时间复杂度，要看 key是否发生index冲突，如果我们可以存放在hashMap没有发生index冲突的情况下，那它的时间复杂度就是O1.也就是我只需要查询一次就能找到.如果我们key存放在hashmap中有index冲突，采用链表进行存放。也就是jdk1.7的hashmap,采用链表集合index冲突的key那它的时间复杂度就是O(n)，也就是从头部查询到尾部. 如果key发生了index冲突采用 红黑树来存放，那它的时间复杂度就是O(logn)
      - 

      

   4. redis数据强一致性达不到；缓存双删有什么用？PV，UV是什么？单独存储访问量的话bitmap存储和int存储有什么区别？缓存双删有必要吗？

   5. redis热点你了解吗？redis集群，假如某一个key的访问量非常大，那么应该怎么安排？

      https://zhuanlan.zhihu.com/p/624724087?utm_id=0

   6. mysql联合索引ABC，A=1,B>3,C=1这样能用到索引吗？

      出现<>右侧索引不生效，即c不生效，ACB也是生效的，即存在就行顺序无所谓。

      https://blog.csdn.net/weixin_46301906/article/details/125052747

   7. redis统计UV（独立访问ip）， DAU（单日活跃用户）

      key: 单日 UV：date  多个日期： UV：start：enddate

      - UV：获取当天的redisKey，将IP存入Hyperloglog，统计指定日期范围UV时遍历时间范围内的日期并获得相应的key，将key存入list中，然后设置区间日期key，使用union合并多个日期并存入该key，调用size统计个数。
      - DAU：获取当天的redisKey，使用userid作为索引存入bool值（例如将101位设置为1），统计日期范围内的活跃用户，同样整理key，进行or运算

      

   8. 事务隔离性指的是什么？幻读是什么？怎么避免？

   9. 消息队列的用处有哪些？MQ和RBC的应用场景？

   10. equals为什么要重写hashcode？==是怎么判断的？equals是怎判断相等的？

       如果补充些hashcode，存储hashmap时你希望两个相等的对象（p1,p2存入p1,取p2会是null，与希望的结果不同）

       https://blog.csdn.net/JokerLJG/article/details/119236911

       

# 小米

1. redis强一致性和弱一致性是什么？应用场景

   Redis是一个基于内存的高性能键值存储系统，它提供了多种数据一致性级别，包括强一致性和弱一致性。下面是对这两种一致性级别的介绍，以及它们各自的优劣势和适用场景：

   1. 强一致性：
      - 定义：强一致性要求数据在任何时间点都必须保持一致。即当数据被更新后，所有后续的读操作都能立即看到最新的结果，且所有的副本都是同步的。
      - 优势：强一致性保证了数据的完全一致性，对于要求数据强一致性的业务场景非常重要。它提供了最简单的数据模型，对开发人员来说更加直观和易于理解。
      - 劣势：强一致性通常需要更多的时间和资源来实现，可能会引入一定的延迟。在分布式系统中，为了保证强一致性，可能需要增加同步和通信的开销。

   2. 弱一致性：
      - 定义：弱一致性允许数据在某个时间点上存在不一致的情况，即在更新操作完成后，并不要求所有的读操作都能立即看到最新结果。数据的一致性是在一段时间内达成的，可能存在短暂的不一致状态。
      - 优势：弱一致性可以提供更高的性能和可用性，因为它允许系统在一定程度上牺牲一致性以换取性能上的提升。在分布式系统中，弱一致性更容易实现，并且可以更好地处理网络延迟和节点故障等问题。
      - 劣势：弱一致性可能会导致数据的临时不一致，需要业务逻辑能够容忍和处理这种情况。对于一些要求强一致性的业务场景，弱一致性的模型可能无法满足需求。

   应用场景：
   - 强一致性：强一致性适用于对数据一致性要求非常高的场景，如金融交易系统、订单处理系统等。在这些系统中，任何时间点上的数据不一致都可能引发严重的问题。
   - 数据库和缓存数据强一致场景：更新 db 的时候同样更新 cache，不过我们需要加一个锁/分布式锁来保证更新 cache 的时候不存在线程安全问题。
   - 可以短暂地允许数据库和缓存数据不一致的场景：更新 db 的时候同样更新 cache，但是给缓存加一个比较短的过期时间，这样的话就可以保证即使数据不一致的话影响也比较小。

   - 弱一致性：弱一致性适用于对数据一致性要求相对较低，但对性能和可用性要求较高的场景。例如，社交媒体应用中的点赞、评论等操作，即使存在短暂的不一致，也不会对用户体验产生

2. 如何保证redis和数据库之间的强一致性

   保证Redis和数据库的强一致性是一个复杂的问题，因为Redis是一个内存数据库，而数据库通常是磁盘上的持久化存储。在默认情况下，Redis的持久化机制可能无法提供强一致性，因为它依赖于异步的数据复制。

3. tcp，udp区别，应用场景

4. 简历所有问题

5. 本地缓存的作用，为什么有了redis还要用本地缓存，调用IO会有资源的消耗和时延

   使用本地缓存的主要原因是加速数据访问和减轻后端负载。虽然Redis是一个高性能的内存数据库，但在某些情况下，使用本地缓存可以提供更低的延迟和更高的吞吐量，具体原因如下：

   1. 降低网络延迟：本地缓存通常位于应用程序的进程内或近距离的内存中，与应用程序直接交互，因此可以大大减少网络通信的延迟。相比于通过网络访问Redis，本地缓存能够更快地响应数据请求。

   2. 减轻后端负载：使用本地缓存可以减少对后端数据存储（如Redis）的访问频率。当数据被频繁请求时，本地缓存可以将数据保存在更快速的存储介质中，避免了每次请求都需要查询Redis的开销，减轻了后端负载，提高了整体系统的吞吐量。

   3. 提高并发性能：本地缓存能够有效地缓解高并发场景下的访问压力。通过将热门或频繁访问的数据存储在本地缓存中，可以避免并发请求直接竞争后端存储的情况，从而提高了并发性能和响应速度。

   4. 数据局部性：在某些应用场景中，数据具有较强的局部性，即一些数据的访问频率高于其他数据。通过使用本地缓存，可以将这些热门数据保留在近距离的本地存储中，提高了对这些热门数据的访问速度。

   需要注意的是，使用本地缓存也存在一些潜在的问题和挑战，例如缓存一致性、缓存过期策略、内存管理等。在设计和实现本地缓存时，需要综合考虑系统的特点、访问模式和数据一致性的需求，合理选择合适的缓存方案。

   redis快的原因https://blog.csdn.net/u011663149/article/details/85307615（基于内存，单线程，IO多路复用，数据结构改进）

6. Mybatis 二级缓存：

   - 一级缓存：默认开启，每一个seesion（一次浏览器访问）时缓存数据，会话关闭缓存就消失，不同的session不同的缓存
   - 二级缓存：手动开启，同一个mapper下都有用（不同seesion也有用）。所有数据都先放在一级缓存中，只有当会话提交或关闭时数据才会转移到二级缓存中。

   用户查询流程：

   ![截屏2023-06-02 上午7.10.37](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-06-02 上午7.10.37.png)

# 顺丰

### 1. redis分布式锁怎么实现？（三个几点让其中一台工作）

SETNX命令

### 2. redis什么时候会用到多线程，redis基本数据结构

刚刚说了一堆使用单线程的好处，现在话锋一转，又要说为什么要引入多线程，别不适应。引入多线程说明Redis在有些方面，单线程已经不具有优势了。

因为读写网络的read/write系统调用在Redis执行期间占用了大部分CPU时间，如果把网络读写做成多线程的方式对性能会有很大提升。

Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程。之所以这么设计是不想 Redis 因为多线程而变得复杂，需要去控制 key、lua、事务，LPUSH/LPOP 等等的并发问题。

Redis 在最新的几个版本中加入了一些可以被其他线程[异步处理](https://www.zhihu.com/search?q=异步处理&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A3055585918})的删除操作，也就是我们在上面提到的 UNLINK、FLUSHALL ASYNC 和 FLUSHDB ASYNC，我们为什么会需要这些删除操作，而它们为什么需要通过多线程的方式异步处理？

我们知道Redis可以使用[del命令](https://www.zhihu.com/search?q=del命令&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A3055585918})删除一个元素，如果这个元素非常大，可能占据了几十兆或者是几百兆，那么在短时间内是不能完成的，这样一来就需要多线程的异步支持。

![img](https://pica.zhimg.com/80/v2-635e43abc8c9d51364b3032586d59573_720w.webp?source=1940ef5c)

现在删除工作可以在后台进行。

总结：之前用单线程是因为基于内存速度快，而且多路复用有多路复用的作用，也就是足够了，现在引入是因为在某些操作要优化，比如删除操作，因此引入了多线程。

### 3. redis持久化，linux父子进程写时复制

- 创建子线程写RDB，如果主线程也一直在写怎么办？怎么保持一致

  当 Redis 在执行持久化过程中，即子进程正在写入 RDB 文件时，如果主线程又写入了新的数据，Redis 会按照以下方式处理：

  1. Redis 使用了写时复制（copy-on-write）机制，所以主线程的写入操作不会直接影响子进程正在写入的数据。

  2. 当子进程完成 RDB 的写入后，Redis 会对主线程在写入过程中的新数据进行追加。

  这意味着主线程的写入操作会在持久化过程结束后才会被写入到磁盘上的 RDB 文件中，以确保持久化的一致性。这种方式避免了数据在持久化过程中的损坏或不一致性。

- 父进程创建子进程，两者共享内存空间，父进程对内存做了改变，子进程怎么保证自己的内存空间不受影响？

  fork（）会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，linux中引入了“写时复制“技术，也就是只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。于是起初我就感到奇怪，子进程的物理空间没有代码，怎么去取指令执行exec系统调用呢？！原来在fork之后exec之前两个进程用的是相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间，也就是说，两者的虚拟空间不同，但其对应的物理空间是同一个。当父子进程中有更改相应段的行为发生时，再为子进程相应的段分配物理空间，如果不是因为exec，内核会给子进程的数据段、堆栈段分配相应的物理空间（至此两者有各自的进程空间，互不影响），而代码段继续共享父进程的物理空间（两者的代码完全相同）。而如果是因为exec，由于两者执行的代码不同，子进程的代码段也会分配单独的物理空间。
  在网上看到还有个细节问题就是，fork之后内核会通过将子进程放在队列的前面，以让子进程先执行，以免父进程执行导致写时复制，而后子进程执行exec系统调用，因无意义的复制而造成效率的下降。

### 4. hashmap扩容原理，redis中hash扩容（需要全部复制一遍吗）

- hashmap扩容

1、先生成新数组
2、遍历老数组中的每个位置上的链表或者红黑树
3、如果是链表，则直接将链表中的每个元素重新计算下标，并添加到新数组中去
4、如果是红黑树，则先遍历红黑树，先计算出红黑树中每个元素对应在新数组中的下标问题
（1）统计每个下标位置的元素个数
（2）如果该位置下的元素个数超过了8，则生成一个新的红黑树，并将根节点添加到新数组的对应位置
（3）如果该位置下的元素个数没有超过8，那么则生成一个链表，并将链表的头节点条件到新数组的对应位置
5、所有元素转移完了之后，将新数组赋值给hashMap对象的table属性

- redis中hash扩容

  **在扩容和收缩的时候，如果哈希字典中有很多元素，一次性将这些键全部rehash到`ht[1]`的话，可能会导致服务器在一段时间内停止服务。所以，采用渐进式rehash的方式，详细步骤如下：**

  1. **为`ht[1]`分配空间，让字典同时持有`ht[0]`和`ht[1]`两个哈希表**
  2. **将`rehashindex`的值设置为`0`，表示rehash工作正式开始**
  3. **在rehash期间，每次对字典执行增删改查操作是，程序除了执行指定的操作以外，还会顺带将`ht[0]`哈希表在`rehashindex`索引上的所有键值对rehash到`ht[1]`，当rehash工作完成以后，`rehashindex`的值`+1`**
  4. **随着字典操作的不断执行，最终会在某一时间段上`ht[0]`的所有键值对都会被rehash到`ht[1]`，这时将`rehashindex`的值设置为`-1`，表示rehash操作结束**

  **渐进式rehash采用的是一种分而治之的方式，将rehash的操作分摊在每一个的访问中，避免集中式rehash而带来的庞大计算量。**

  **需要注意的是在渐进式rehash的过程，如果有增删改查操作时，如果`index`大于`rehashindex`，访问`ht[0]`，否则访问`ht[1]。`**

  

### 5. spring

https://www.zhihu.com/question/38597960

- 为什么会有spring
- bean的生存周期
- bean是怎么创建的

### 6. mysql什么样的字段适合创建索引

性别字段：

大家都知道索引分[聚集索引](https://so.csdn.net/so/search?q=聚集索引&spm=1001.2101.3001.7020)和非聚集索引，性别字段因为可重复肯定只能建立非聚集索引，然而因为非聚集索引叶子节点存储的是索引值和聚集索引值，需要回表。所以在性别这种辨别度较低的字段上建立索引，索引树可能只有两个节点，跟线性查找没有太大区别，并且因为回表的存在导致在聚集索引树和非聚集索引树来回切换反而导致查询时间更慢。并且维护该索引还要一定的开销。另外，数据库优化器最终很大概率也不会选择走这个索引。综上，在辨别度较低的字段上建立索引得不偿失。

但是，并不是通用规则！

若这些可选择性非常低的字段，在其中的一种分布非常少，而且查询非常频繁的话，可以对该字段进行索引！

比如有一个枚举字段[1,2,3], 在上百万行数据中，1占1%， 2占%2， 3占%97，然后业务经常需要查询1和2的数据，那么就可以在该字段进行索引。

学号：设置主键索引

### 7. 延时双删问题， 删除缓存失败的解决策略

https://blog.csdn.net/prefect_start/article/details/124113776







# 项目

权限模块主要实现了注册登录和权限管理等功能。
帖子模块主要实现了发帖，评论和私信等功能。
性能模块使用Redis实现点赞，关注和网站数据统计等功能。
通知模块使用Kafka实现系统通知功能。
搜索模块使用Elasticsearch实现帖子搜索功能

### 热帖排行: 

在进行点赞，评论，加精等影响帖子热度的操作时，将该帖子id存储在redis中，qurts定时每五分钟从redis中得到需要改变的帖子分数并重新计算帖子得分。

后续优化：使用caffine做热门帖子的缓存，设置缓存三分钟自动清理（帖子会有三分钟延迟，但是不影响使用），存入热门帖子，设置热门帖子总长度为100，不用把所有数据都存入caffine。当用户需要帖子按热度排列的时候就从caffine中调取数据。

### springSecurity：

Interceptor拦截每次访问，查询当前是否有用户登录，如果有则给当前用户授权并存储Securitycontext中。在配置中配置每个需要用户权限访问的路径。

### 缓存双删:

user信息缓存在redis中，在对用户信息例如头像名称等做更改时使用缓存双删，避免出现数据不一致。

一、什么是 Redis [延时双删](https://so.csdn.net/so/search?q=延时双删&spm=1001.2101.3001.7020)？

1、[延迟双删](https://so.csdn.net/so/search?q=延迟双删&spm=1001.2101.3001.7020)策略是分布式系统中数据库存储和缓存数据保持一致性的常用策略，但它不是强一致。不管哪种方案，都无法绝对避免Redis存在脏数据的问题，只能减轻这个问题

2、因为双删策略执行的结果是把redis中保存的那条数据删除了，以后的查询就都会去查询数据库。经常修改的数据表不适合使用redis缓存

3、Redis适用的是读频率远远大于改频率的数据表，不适合改频率大于读频率的数据

二、为什么要使用延时双删？

先看下面两种脏数据情况：

1、情况一：先删除缓存数据，再update更新数据表

当请求1执行删除缓存数据后，还未来得及更新数据表或更新动作还未完成，此时请求2查询到数据表中仍然是更新之前的数据、并把脏数据写入了Redis缓存

2、情况二：先update更新数据表，再删除缓存数据

当请求1执行update更新数据表后，还未来得及删除缓存数据或删除缓存动作还未完成，此时请求2查询到Redis缓存中仍然是旧数据、并返回给前端

为了避免上述的两种情况数据不一致问题，就需要用到我们介绍的延时双闪策略：先删除缓存数据 》再执行update更新数据表 》最后（延时N秒）执行删除缓存

三、实现延时双删，示例：

1、有实现注释，代码如下

更新操作代码：

```plaintext
@Service



public class UserInfoServiceImpl extends ServiceImpl<UserInfoMapper, UserInfo> implements UserInfoService {



 



    @Resource



    private RedisUtil redisUtil;



    @Resource



    private ScheduledExecutorService scheduledExecutorService;



 



 



    /**



     * <p>保存用户信息</p>



     *



     * @author hkl



     * @date 2022/11/22



     */



    @Override



    @Transactional(rollbackFor = Exception.class)



    public void saveUserInfo(UserInfo userInfo) {



        if(ObjectUtil.isNull(userInfo)){



            return;



        }



 



        if(ObjectUtil.isNull(userInfo.getId())){



            this.save(userInfo);



            //Hash方式 start



            redisUtil.putHash(RedisConstants.USER_INFO_HASH, String.valueOf(userInfo.getId()), userInfo);



            //Hash方式 end



        } else {



            //测试Redis延时双删，Hash方式 start



            //1、删除缓存数据



            UserInfo getUserInfoResFirst = this.getById(userInfo.getId());



            Optional.ofNullable(getUserInfoResFirst).ifPresent(obj -> {



                redisUtil.deleteHash(RedisConstants.USER_INFO_HASH, String.valueOf(obj.getId()));



            });



            



            //2、更新数据表



            this.updateById(userInfo);



            



            //3、延时2秒后，再删除缓存数据



            scheduledExecutorService.schedule(() -> {



                Optional.ofNullable(getUserInfoResFirst).ifPresent(obj -> {



                    redisUtil.deleteHash(RedisConstants.USER_INFO_HASH, String.valueOf(obj.getId()));



                });



            }, 2, TimeUnit.SECONDS);



            //测试Redis延时双删，Hash方式 end



        }



    }



 



}
```

查询操作代码：

```plaintext
  @ApiOperation(value = "根据用户Id查询用户详情信息")



    @ApiImplicitParam(value = "用户Id", name = "userId", required = true)



    @GetMapping("/getUserInfoById")



    public CommonResult<UserInfo> getUserInfoById(Integer userId) {



        //Hash方式 start



        Object userInfoObj = redisUtil.getHash(RedisConstants.USER_INFO_HASH, String.valueOf(userId));



        if(ObjectUtil.isNotNull(userInfoObj)){



            return success(userInfoObj);



        }



        UserInfo userInfo = userInfoService.getById(userId);



        Optional.ofNullable(userInfo).ifPresent(obj -> {



            redisUtil.putHash(RedisConstants.USER_INFO_HASH, String.valueOf(userId), obj);



        });



        //Hash方式 end



        return success(userInfo);



    }
```

说明：

以上就是实现缓存和数据表一致的延时双删策略Demo，经过测试可以保持数据一致

四、小结

1、使用延时双删策略是为了保持Redis缓存与数据表一致

2、第一次删除不再赘述，为了清除缓存中的旧数据

3、主要是第二次删除，第二次删除为什么要延时呢？延时1是为了等待更新数据表动作完成、2是为了等待更新数据表之前查询查到的旧数据写入缓存动作完成，最后再把写入缓存的旧数据删除

4、延时双删依然无法保证一致，只能减轻出现脏数据的情况，所以对一致性要求较高的数据尽量不要放入缓存

### 遇到难点：

导包版本，前后端交互使用postman，怎么展示全部评论：存储帖子的数据结构循环存储评论和评论的评论。Elasticsearch 和 Redis 底层均依赖于 netty，存在冲突。在CommunityApplication 做系统配置修改解决冲突。

### 亮点：

点赞、关注、缓存用户的信息，通过redis这一个非关系型数据库，将数据存储在内存中，读写速度快，满足高性能的要求。



1. 注册：
2. 会话管理 cookie， session
3. 登录

![截屏2023-03-11 上午7.07.55](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-11 上午7.07.55.png)

- 登录凭证，设计一张表(id, user_id, ticket, status, expired)

![截屏2023-03-11 上午7.50.25](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-11 上午7.50.25.png)

拦截器作用：在访问所有页面时，拦截器先检查有没有登陆用户，如果有登陆用户则使用登陆用户的信息，如果没有登陆用户符合条件，则不使用登陆信息（头像消息设置等），此时临时用户可以点击页面上方的登陆按钮进行登陆。**此网站支持临时访客登陆。**

问题：多线程并发访问时，用户凭证的存储问题，使用threadlocal

****\*一句话理解ThreadLocal，threadlocl是作为当前线程中属性ThreadLocalMap集合中的某一个Entry的key值Entry（threadlocl,value），虽然不同的线程之间threadlocal这个key值是一样，但是不同的线程所拥有的ThreadLocalMap是独一无二的，也就是不同的线程间同一个ThreadLocal（key）对应存储的值(value)不一样，从而到达了线程间变量隔离的目的，但是在同一个线程中这个value变量地址是一样的。\*****

问题：seesion会出现并发问题吗，session并发控制可以实现服务端控制客户端某个用户同时在线的数量。![截屏2023-03-13 上午7.35.45](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-13 上午7.35.45.png)

![截屏2023-03-13 上午7.22.33](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-13 上午7.22.33.png)

自定义注解的使用逻辑：拦截所有请求，找到带注解的进行处理。也可以拦截指定路径，在拦截器配置里加路径而不用注解。

![截屏2023-03-13 上午7.22.31](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-13 上午7.22.31.png)

[问题](https://www.zhihu.com/question/512822388/answer/2322252278):ajax请求，controller返回不跳转

![截屏2023-03-14 上午7.46.14](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-14 上午7.46.14.png)

![截屏2023-03-14 上午7.12.42](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-14 上午7.12.42.png)

![截屏2023-03-14 上午7.12.56](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-14 上午7.12.56.png)

![截屏2023-03-14 上午7.33.42](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-14 上午7.33.42.png)

![截屏2023-03-14 上午7.28.52](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-14 上午7.28.52.png)

![截屏2023-03-14 上午7.38.46](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-14 上午7.38.46.png)

![截屏2023-03-16 上午7.33.25](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-16 上午7.33.25.png)

![截屏2023-03-16 上午7.06.33](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-16 上午7.06.33.png)

![截屏2023-03-16 上午7.25.48](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-16 上午7.25.48.png)

![截屏2023-03-16 上午7.50.08](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-16 上午7.50.08.png)

![截屏2023-03-16 上午7.28.34](/Users/LJP/Desktop/Typora/javaLearning/config/截屏2023-03-16 上午7.28.34.png)



   

除了模态间和模态内的相似性约束外，我们进一步正则化两种模态的归一化哈希表示和哈希码之间的二值化差异，从而推导出哈希码的最优连续代理。具体来说，我们首先对视觉和文本模态的哈希码向量进行归一化，然后直接利用归一化哈希表示和哈希的内积来估计它们之间的相似性。通过这种方式，我们可以获得视觉和文本模态的 1× N 自监督相似度矩阵，即 Cbv 和 Cbt。它们的详细信息如下



