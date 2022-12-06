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











