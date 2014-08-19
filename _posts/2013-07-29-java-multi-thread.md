---
layout: post
title: Java多线程发展简史
description: Java多线程发展简史
---
http://www.raychase.net/698
   如果从历史的角度去了解一门语言一个特性的演进，或许能有不同收获

##进程/线程调度方式-抢占式和协作式

###抢占式
   操作系统非常适合使用抢占式方式来调度它的进程，它给不同的进程分配时间片，对于长期无响应的进程，它有能力剥夺它的资源，甚至将其强行停止（如果采用协作式的方式，需要进程自觉、主动地释放资源，也许就不知道需要等到什么时候了）

###协作式
   Java语言一开始就采用协作式的方式，并且在后面发展的过程中，逐步废弃掉了粗暴的stop/resume/suspend这样的方法，它们是违背协作式的不良设计，转而采用wait/notify/sleep这样的两边线程配合行动的方式
   1996年1月的JDK1.0版本就确定,至今并未发生实质性的变更，可以说是一个具有传承性的良好设计
<pre class="prettyprint">
public class InterruptCheck extends Thread {
 
    @Override
    public void run() {
        System.out.println("start");
        while (true)
            if (Thread.currentThread().isInterrupted())
                break;
        System.out.println("while exit");
    }
 
    public static void main(String[] args) {
        Thread thread = new InterruptCheck();
        thread.start();
        try {
            sleep(2000);
        } catch (InterruptedException e) {
        }
        thread.interrupt();
    }
}
</pre>

这是中断的一种使用方式，看起来就像是一个标志位，线程A设置这个标志位，线程B时不时地检查这个标志位
<pre class="prettyprint">
public class InterruptWait extends Thread {
    public static Object lock = new Object();
 
    @Override
    public void run() {
        System.out.println("start");
        synchronized (lock) {
            try {
                lock.wait();
            } catch (InterruptedException e) {
                System.out.println(Thread.currentThread().isInterrupted());
                Thread.currentThread().interrupt(); // set interrupt flag again
                System.out.println(Thread.currentThread().isInterrupted());
                e.printStackTrace();
            }
        }
    }
 
    public static void main(String[] args) {
        Thread thread = new InterruptWait();
        thread.start();
        try {
            sleep(2000);
        } catch (InterruptedException e) {
        }
        thread.interrupt();
    }
}
</pre>
在这种方式下，如果使用wait方法处于等待中的线程，被另一个线程使用中断唤醒，于是抛出InterruptedException，同时，中断标志清除，这时候通常会在捕获该异常的地方重新设置中断，以便后续的逻辑通过检查中断状态来了解该线程是如何结束的

<pre class="prettyprint">
import java.util.concurrent.atomic.AtomicInteger;
 
public class Atomicity {
 
    private static volatile int nonAtomicCounter = 0;
    private static volatile AtomicInteger atomicCounter = new AtomicInteger(0);
    private static int times = 0;
 
    public static void caculate() {
        times++;
        for (int i = 0; i < 1000; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    nonAtomicCounter++;
                    atomicCounter.incrementAndGet();
                }
            }).start();
        }
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
        }
    }
 
    public static void main(String[] args) {
        caculate();
        while (nonAtomicCounter == 1000) {
            nonAtomicCounter = 0;
            atomicCounter.set(0);
            caculate();
        }
 
        System.out.println("Non-atomic counter: " + times + ":"
                + nonAtomicCounter);
        System.out.println("Atomic counter: " + times + ":" + atomicCounter);
    }
}
</pre>
也许需要跑几次才能看到效果，使用非原子性的++操作，结果经常小于1000
<pre class="prettyprint">
public class Lock {
    private static Object o = new Object();
    static Lock lock = new Lock();
 
    // lock on dynamic method
    public synchronized void dynamicMethod() {
        System.out.println("dynamic method");
        sleepSilently(2000);
    }
 
    // lock on static method
    public static synchronized void staticMethod() {
        System.out.println("static method");
        sleepSilently(2000);
    }
 
    // lock on this
    public void thisBlock() {
        synchronized (this) {
            System.out.println("this block");
            sleepSilently(2000);
        }
    }
 
    // lock on an object
    public void objectBlock() {
        synchronized (o) {
            System.out.println("dynamic block");
            sleepSilently(2000);
        }
    }
 
    // lock on the class
    public static void classBlock() {
        synchronized (Lock.class) {
            System.out.println("static block");
            sleepSilently(2000);
        }
    }
 
    private static void sleepSilently(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    public static void main(String[] args) {
 
        // object lock test
        new Thread() {
            @Override
            public void run() {
                lock.dynamicMethod();
            }
        }.start();
        new Thread() {
            @Override
            public void run() {
                lock.thisBlock();
            }
        }.start();
        new Thread() {
            @Override
            public void run() {
                lock.objectBlock();
            }
        }.start();
 
        sleepSilently(3000);
        System.out.println();
 
        // class lock test
        new Thread() {
            @Override
            public void run() {
                lock.staticMethod();
            }
        }.start();
        new Thread() {
            @Override
            public void run() {
                lock.classBlock();
            }
        }.start();
 
    }
}
</pre>
上面的例子可以反映对一个锁竞争的现象，结合上面的例子，理解下面这两条，就可以很容易理解synchronized关键字的使用：


非静态方法使用synchronized修饰，相当于synchronized(this)
静态方法使用synchronized修饰，相当于synchronized(Lock.class)