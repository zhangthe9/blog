---
layout: post
title: java 多线程
description: java 多线程
---
通常多线程程序结构如下
<pre class="prettyprint" >
while (true) {
	if (no tasks)	{
		wait for a task;
		execute the task; 
	} 
}
</pre>

<pre class="prettyprint" >
class WorkerThread extends Thread {
	@Override
	public void run() {
		/* do work */
	}
}

Thread t = new WorkerThread();
t.start();
或者： 
Thread t = new Thread(new Runnable() {
				public void run() {
								/* do work */
			}
});
t.start();
</pre>

###使用线程池解决问题
负载不可预知后，不能都用new Thread(runnable).start() ，可能很快耗尽资源了

###java.util.concurrent
JDK5新增软件包java.util.concurrent，提供大量并发编程实用工具类
java.util.concurrent.Executor是个执行器。执行已提交的 Runnable 任务的对象
将任务提交与每个任务将如何运行的机制（包括线程使用的细节、调度等）分离开来。使用 Executor 而不是显式地创建线程
	
###Executor 框架
管理实现 Runnable 的任务
<pre class="prettyprint" >
public interface Executor { 
	void execute(Runnable command);
}
</pre>
	
调整执行策略（队列限制、池大小、优先级排列等等）容易

ExecutorService还管理生命周期
<pre class="prettyprint" >
public interface ExecutorService extends Executor {
	void shutdown();
	List<Runnable> shutdownNow(); 
	boolean isShutdown(); 
	boolean isTerminated();
	boolean awaitTermination(long timeout, TimeUnit unit);
	// other convenience methods for submitting tasks
	} 
}
</pre>
<pre class="prettyprint" >	
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestThreadPool {
	public static void main(String args[]) throws InterruptedException {
		// only two threads
		ExecutorService exec = Executors.newFixedThreadPool(2);
		for (int index = 0; index < 100; index++) {
			Runnable run = new Runnable() {
				public void run() {
					long time = (long) (Math.random() * 1000);
					System.out.println("Sleeping " + time + "ms");
					try {
						Thread.sleep(time);
					} catch (InterruptedException e) {
					}
				}
			};
			exec.execute(run);
		}
		// must shutdown
		exec.shutdown();
	}
}
</pre>
线程池必须使用shutdown来显式关闭，否则主线程就无法退出。shutdown也不会阻塞主线程
	
###Executor 实现的不同执行策略
何时在哪个线程中运行任务，执行任务可能消耗的资源级别（线程、内存等等），以及如果执行程序超载该怎么办
	
- Executors.newCachedThreadPool() 
- Executors.newFixedThreadPool(int n) 
- Executors.newSingleThreadExecutor() 
	
<pre class="prettyprint" >	
public static void main(String[] args) {
		final ScheduledExecutorService scheduler = Executors
				.newScheduledThreadPool(2);
		final Runnable beeper = new Runnable() {
			int count = 0;

			public void run() {
				System.out.println(new Date() + " beep " + (++count));
			}
		};
		// 1秒钟后运行，并每隔2秒运行一次
		final ScheduledFuture beeperHandle = scheduler.scheduleAtFixedRate(
				beeper, 1, 2, SECONDS);
		// 2秒钟后运行，并每次在上次任务运行完后等待5秒后重新运行
		final ScheduledFuture beeperHandle2 = scheduler.scheduleWithFixedDelay(
				beeper, 2, 5, SECONDS);
		// 30秒后结束关闭任务，并且关闭Scheduler
		scheduler.schedule(new Runnable() {
			public void run() {
				beeperHandle.cancel(true);
				beeperHandle2.cancel(true);
				scheduler.shutdown();
			}
		}, 30, SECONDS);
	}
</pre>

###CyclicBarrier
多个线程同时工作以完成一件事，而且完成过程中，往往会等待其他线程都完成某一阶段后再执行，等所有线程都到达某一个阶段后再统一执行

CyclicBarrier有参与者个数属性和await()方法。当所有线程都调用了await()后，就表示这些线程都可以继续执行，否则就会等待
	
比如有几个旅行团需要途经深圳、广州、韶关、长沙最后到达武汉。旅行团中有自驾游的，有徒步的，有乘坐旅游大巴的；这些旅行团同时出发，并且每到一个目的地，都要等待其他旅行团到达此地后再同时出发，直到都到达终点站武汉	

<pre class="prettyprint" >
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestCyclicBarrier {
	// Shenzhen, Guangzhou, Shaoguan, Changsha, Wuhan

	// 自驾游
	private static int[] timeSelf = { 1, 3, 4, 4, 5 };
	// 旅游大巴
	private static int[] timeBus = { 2, 4, 6, 6, 7 };
	// 徒步
	private static int[] timeWalk = { 5, 8, 15, 15, 10 };

	static String now() {
		SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
		return sdf.format(new Date()) + ": ";
	}

	static class Tour implements Runnable {
		private int[] times;
		private CyclicBarrier barrier;
		private String tourName;

		public Tour(CyclicBarrier barrier, String tourName, int[] times) {
			this.times = times;
			this.tourName = tourName;
			this.barrier = barrier;
		}

		public void run() {
			try {
				Thread.sleep(times[0] * 1000);
				System.out.println(TestCyclicBarrier.now() + tourName
						+ " Reached Shenzhen");
				barrier.await();
				Thread.sleep(times[1] * 1000);
				System.out.println(TestCyclicBarrier.now() + tourName
						+ " Reached Guangzhou");
				barrier.await();
				Thread.sleep(times[2] * 1000);
				System.out.println(TestCyclicBarrier.now() + tourName
						+ " Reached Shaoguan");
				barrier.await();
				Thread.sleep(times[3] * 1000);
				System.out.println(TestCyclicBarrier.now() + tourName
						+ " Reached Changsha");
				barrier.await();
				Thread.sleep(times[4] * 1000);
				System.out.println(TestCyclicBarrier.now() + tourName
						+ " Reached Wuhan");
				barrier.await();
			} catch (InterruptedException e) {
			} catch (BrokenBarrierException e) {
			}
		}
	}

	public static void main(String[] args) {
		// 三个旅行团
		CyclicBarrier barrier = new CyclicBarrier(3);
		ExecutorService exec = Executors.newFixedThreadPool(3);
		exec.submit(new Tour(barrier, "WalkTour", TestCyclicBarrier.timeWalk));
		exec.submit(new Tour(barrier, "SelfTour", TestCyclicBarrier.timeSelf));
		exec.submit(new Tour(barrier, "BusTour", TestCyclicBarrier.timeBus));
		exec.shutdown();
	}

}
</pre>


###BlockingQueue
- put()
放到队列中，如果队列已经满了，就等待直到有空闲节点
- take()
从head取，没有就等待直到有可取的
	

###原子整型
AtomicInteger可以在并发情况下达到原子化更新，避免使用了synchronized，而且性能非常高

###CountDownLatch
倒数计数的锁，倒数到0触发事件，开锁，其他人就可以进入了
countDown()倒数一次
await() 等待倒数到0，如果没有到达0，就只有阻塞等待了

CountDouwnLatch实例不能重复使用，一次性的，锁一经被打开就不能再关闭使用，重复使用用CyclicBarrier

模拟100米赛跑，10名选手已经准备就绪，只等裁判一声令下。当所有人都到达终点时，比赛结束 

<pre class="prettyprint" >
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestCountDownLatch {
	public static void main(String[] args) throws InterruptedException {
		// 开始的倒数锁
		final CountDownLatch begin = new CountDownLatch(1);
		// 结束的倒数锁
		final CountDownLatch end = new CountDownLatch(10);
		// 十名选手
		final ExecutorService exec = Executors.newFixedThreadPool(10);
		for (int index = 0; index < 10; index++) {
			final int NO = index + 1;
			Runnable run = new Runnable() {
				public void run() {
					try {
						begin.await();
						Thread.sleep((long) (Math.random() * 10000));
						System.out.println("No." + NO + " arrived");
					} catch (InterruptedException e) {
					} finally {
						end.countDown();
					}
				}
			};
			exec.submit(run);
		}
		System.out.println("Game Start");
		begin.countDown();
		end.await();
		System.out.println("Game Over");
		exec.shutdown();
	}
}
</pre>				
####ThreadPoolExecutor
newFixedThreadPool 和 newCachedThreadPool 返回的 Executor 是ThreadPoolExecutor 的实例，可定制
###RejectedExecutionHandler 	
ThreadPoolExecutor.setRejectedExecutionHandler（默认在抛异常时)
在Executor不工作时用，如已关闭，队列已满
放弃任务，在调用者的线程中执行任务，或放弃队列中最早的任务以为新任务腾出空间
还可 扩展 ThreadPoolExecutor，重写 beforeExecute 和 afterExecute，进行其他执行定制

###Future 接口
表示已经完成、正在执行或者尚未开始执行的任务
可以取消尚未完成，查询任务已经完成还是取消了，以及提取（或等待）任务的结果值
	
FutureTask 实现 Future，用构造函数将 Runnable 或 Callable（会产生结果的 Runnable）和 Future 接口封装
因为 FutureTask 也实现 Runnable，所以可以只将 FutureTask 提供给 Executor
ExecutorService.submit()等提交方法还将返回 Future 接口
Future.get() 方法检索任务计算的结果（或如果任务完成，但有异常，则抛出 ExecutionException）。如果任务尚未完成，那么 Future.get() 将被阻塞，直到任务完成；如果任务已经完成，那么它将立即返回结果
get(timeout) 在timeout时间内没有取到就失败返回，而不再阻塞
	
<pre class="prettyprint" >
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class TestFutureTask {
	public static void main(String[] args) throws InterruptedException,
			ExecutionException {
		final ExecutorService exec = Executors.newFixedThreadPool(5);
		Callable call = new Callable() {
			public String call() throws Exception {
				Thread.sleep(1000 * 5);
				return "Other less important but longtime things";
			}
		};
		Future task = exec.submit(call);
		// 重要的事情
		Thread.sleep(1000 * 3);
		System.out.println("Let’s do important things.");
		// 其他不重要的事情
		String obj = (String) task.get();
		System.out.println(obj);
		// 关闭线程池
		exec.shutdown();
	}
}
</pre>	
	
###CompletionService
- submit()
提交runnable或者callable，一般会提交给一个线程池处理
- take()
	取出已经执行完毕runnable或者callable实例的Future对象，如果没有满足要求的，就等待了
- poll()
	与take类似，只是不会等待，如果没有满足要求，就返回null对象
<pre class="prettyprint" >
import java.util.concurrent.Callable;
import java.util.concurrent.CompletionService;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorCompletionService;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class TestCompletionService {
	public static void main(String[] args) throws InterruptedException,
			ExecutionException {
		ExecutorService exec = Executors.newFixedThreadPool(10);
		CompletionService serv =
				new ExecutorCompletionService(exec);
		for (int index = 0; index < 5; index++) {
			final int NO = index;
			Callable downImg = new Callable() {
				public String call() throws Exception {
					Thread.sleep((long) (Math.random() * 10000));
					return "Downloaded Image" + NO;
				}
			};
			serv.submit(downImg);
		}
		Thread.sleep(1000 * 2);
		System.out.println("Show web content");
		for (int index = 0; index < 5; index++) {
			Future task = serv.take();
			String img = (String) task.get();
			System.out.println(img);
		}
		System.out.println("End");
		// 关闭线程池
		exec.shutdown();
	}
}
</pre>
	
###Semaphore
信号量，控制某个资源可被同时访问的个数
	
- acquire()
	
获取许可，如果没有就等待
	
- release()
	
释放许可
比如在Windows下可以设置共享文件的最大客户端访问个数

Semaphore维护了当前访问的个数，提供同步机制，控制同时访问的个数。在数据结构中链表可以保存“无限”的节点，用Semaphore可以实现有限大小的链表。另外重入锁ReentrantLock也可以实现该功能，但实现上要负责些，代码也要复杂些

只有5个许可的Semaphore，而有20个线程要访问这个资源，通过acquire()和release()获取和释放访问许可
<pre class="prettyprint" >
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class TestSemaphore {
	public static void main(String[] args) {
		// 线程池
		ExecutorService exec = Executors.newCachedThreadPool();
		// 只能5个线程同时访问
		final Semaphore semp = new Semaphore(5);
		// 模拟20个客户端访问
		for (int index = 0; index < 20; index++) {
			final int NO = index;
			Runnable run = new Runnable() {
				public void run() {
					try {
						// 获取许可
						semp.acquire();
						System.out.println("Accessing: " + NO);
						Thread.sleep((long) (Math.random() * 10000));
						// 访问完后，释放
						semp.release();
					} catch (InterruptedException e) {
					}
				}
			};
			exec.execute(run);
		}
		// 退出线程池
		exec.shutdown();
	}
}
</pre>

- java.util.concurrent.atomic 
 包含不用加锁情况下就能改变值的原子变量
    
- java.util.concurrent.locks 
 包含锁定的工具

###ReentrantLock