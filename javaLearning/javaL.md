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

# 项目

权限模块主要实现了注册登录和权限管理等功能。
帖子模块主要实现了发帖，评论和私信等功能。
性能模块使用Redis实现点赞，关注和网站数据统计等功能。
通知模块使用Kafka实现系统通知功能。
搜索模块使用Elasticsearch实现帖子搜索功能

遇到难点：导包版本，前后端交互使用postman，怎么展示全部评论：存储帖子的数据结构循环存储评论和评论的评论。Elasticsearch 和 Redis 底层均依赖于 netty，存在冲突。在CommunityApplication 做系统配置修改解决冲突。

亮点：点赞、关注、缓存用户的信息，通过redis这一个非关系型数据库，将数据存储在内存中，读写速度快，满足高性能的要求。

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



