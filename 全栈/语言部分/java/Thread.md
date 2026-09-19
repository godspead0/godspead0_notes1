[toc]



## 进程和线程

### 区别

1. 进程是包含线程的. 每个进程至少有⼀个线程存在，即主线程。
2. 进程和进程之间不共享内存空间. 同⼀个进程的线程之间共享同⼀个内存空间.
3. 进程是系统分配资源的最小单位，线程是系统调度的最小单位。
4. ⼀个进程挂了⼀般不会影响到其他进程. 但是⼀个线程挂了, 可能把同进程内的其他线程⼀起带走(整 个进程崩溃)

### 为什么要有线程

1. 单核 CPU 的发展遇到了瓶颈. 要想提高算力, 就需要多核 CPU. 而并发编程能更充分利用多核 CPU 资源.

2. 有些任务场景需要 “等待 IO”, 为了让等待 IO 的时间能够去做⼀些其他的工作, 也需要用到并发编程. 其次,
   虽然多进程也能实现 并发编程, 但是线程比进程更轻量.

3. 创建线程比创建进程更快.

4. 销毁线程比销毁进程更快.

5. 调度线程比调度进程更快.

## 创建线程

1. 继承Thread类

```Java
class MyThread extends Thread { 
    @Override
    public void run() {
        System.out.println("这⾥是线程运⾏的代码");
    }
}
public class Test {
    public static void main(String[] args)  {
        MyThread t = new MyThread();
        t.start();
    }
}
```

2. 实现Runnable接口

```Java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("这⾥是线程运⾏的代码");
    }
}
public class Test {
    public static void main(String[] args)  {
        Thread t = new Thread(new MyRunnable());
        t.start();
    }
}
```

3. 匿名内部类创建Thread子类对象

```Java
public class Test {
    public static void main(String[] args)  {
        // 使⽤匿名类创建 Thread ⼦类对象
        Thread t1 = new Thread() {
            @Override
            public void run() {
                System.out.println("使⽤匿名类创建 Thread ⼦类对象");
            }
        };
    }
}
```

4. 匿名内部类创建Runnable子类对象

```Java
public class Test {
    public static void main(String[] args)  {
        // 使⽤匿名类创建 Runnable ⼦类对象
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("使⽤匿名类创建 Runnable ⼦类对象");
            }
        });
    }
}
```

5. lambda表达式创建Runnable子类对象

```Java
public class Test {
    public static void main(String[] args)  {
        // 使⽤匿名类创建 Runnable ⼦类对象
        // 使⽤ lambda 表达式创建 Runnable ⼦类对象
        Thread t3 = new Thread(() -> System.out.println("使⽤匿名类创建 Thread ⼦类对象"));
        Thread t4 = new Thread(() -> {
            System.out.println("使⽤匿名类创建 Thread ⼦类对象");
        });
    }
}
```

6. 实现callable接口

* 与Runnable接口类似，只是该方式有返回值，但Runnable没有返回值
* 需要使用一个中介FutureTask

```Java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class Test {
	public static void main(String[] args) {
		//返回值是int类型
		Callable callable=()->{
			int result=0;
			for(int i=0;i<100;i++) {
				result+=i;
			}
			return result;
		};
		//Thread thread=new Thread(callable); 不能直接像创建Runnable接口一样
		//知道返回值是int性。使用泛型约束
		FutureTask<Integer> task=new FutureTask<> (callable);
		
		Thread thread=new Thread(task);
		thread.start();
		
		//获取计算结果
		Integer integer = null;
		try {
			integer = task.get();  //该方法会抛出两个异常，需要手动处理
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}  
		System.out.println(integer);
	}
}
```



## Thread类及其方法

### 构造方法

| 方法                                 | 说明                                |
| ------------------------------------ | ----------------------------------- |
| Thread()                             | 创建线程对象                        |
| Thread(Runnable target)              | 使用Runnable对象创建线程对象        |
| Thread(String name)                  | 创建线程对象,并命名                 |
| Thread(Runnable target, String name) | 使用Runnable对象创建线程对象,并命名 |

### 常见属性

| 属性         | 获取方法        |
| ------------ | --------------- |
| ID           | getId()         |
| 名称         | getName()       |
| 状态         | getState()      |
| 优先级       | getPriority()   |
| 是否后台线程 | isDaemon()      |
| 是否存活     | isAlive()       |
| 是否被中断   | isInterrupted() |

**补充**:

- ID是线程的唯⼀标识，不同线程不会重复
- 名称是各种调试工具用到
- 状态表示线程当前所处的⼀个情况，下面我们会进⼀步说明
- 优先级高的线程理论上来说更容易被调度到

### 常用方法

#### 线程的命名setName

- 实例化一个线程，使用setName()方法
- 实例化一个线程的同时，通过构造方法对线程进行命名
- 使用用户自定义的线程类，在实例化的同时，进行名字的赋值，需要给自定义线程类添加对应的构造方法`

```Java
class MyThread extends Thread{
	public MyThread() {}
	public MyThread(String name) {
		this.setName(name);     //使用setName()方法
		//super(name);  //直接调用父类的构造方法
	}
}
public class ThreadClass {
	public static void main(String[] args) {
		//1、实例化一个线程，使用setName()方法
		Thread t=new Thread();
		t.setName("用户线程1");
		System.out.println(t.getName());

		//2、实例化一个线程的同时，通过构造方法对线程进行命名
		//  构造方法：Thread(Runnable r,String name);
		Thread t2=new Thread(()->{},"用户线程2"); 
		System.out.println(t2.getName());
		
		//3、使用用户自定义的线程类，在实例化的同时，进行名字的赋值
		//   需要给自定义线程类添加对应的构造方法
		MyThread t3=new MyThread("用户线程3");
		System.out.println(t3.getName());
	}
}
```

#### 线程休眠sleep（Run->Interrupt）

* 调用**sleep()**方法，参数：以毫秒为单位的时间差
* 会抛出**InterruptedException异常**，需要处理
* 使得线程由运行状态变为阻塞状态，当休眠时间到达时，才会重新变为就绪状态。即使此时系统中没有其他可执行的线程，处于sleep的线程也依然不会执行

```Java
class MyThread extends Thread{
	//重写run方法
	@Override
	public void run() {
		for(int i=0;i<5;i++) {
			System.out.println(+i);
			//线程休眠
			//参数：以毫秒为单位
			//需要捕获异常
			try {
				Thread.sleep(1000);  //休眠1秒
			} 
			catch (InterruptedException e) {
				e.printStackTrace();
			}   
		}
	}
}
public class ThreadClass {
	public static void main(String[] args) {
		//调用threadSleep方法
		threadSleep();
	}
    /****线程休眠****/
	public static void threadSleep() {
		//实例化一个线程
		MyThread mt=new MyThread();
		mt.start();
	}
}
//输出形式：每隔1秒输出一个i值
AI写代码
java
运行

```

#### 线程的优先级setPriority

* 调用**setPriority()**方法，参数：[0,10]范围内的一个整数，默认是5
* 设置优先级，只是设置这个线程可以抢到CPU时间片的概率，并不是优先级高的线程一定能抢到CPU时间片（不是优先级高的线程一定先执行，也不是优先级高的线程执行完再执行其他线程）
* 设置优先级必须要放在线程开始（start）之前

```Java
public class ThreadClass {
	public static void main(String[] args) {
		threadPriority();
	}
    /****设置线程的优先级***/
	public static void threadPriority() {
		Runnable r=()->{
			for(int i=0;i<5;i++){
					System.out.println(Thread.currentThread().getName()+":"+i);
			}
		};
        //1、线程实例化
		Thread t1=new Thread(r,"Thread-1");
		Thread t2=new Thread(r,"Thread-2");
		
		//2、设置优先级, 必须要将该操作放在线程开始（start）之前
		t1.setPriority(10);
		t2.setPriority(1);
		
        //3、线程启动
		t1.start();
		t2.start();
	}
}
//输出结果：交替执行
```

#### 线程的礼让yield（Run->Ready）

* 调用**yield()**方法，类方法
* 线程礼让是指让当前运行的线程释放自己的CPU资源，由运行状态，回到就绪状态。**但并不意味着一定去执行另一个线程，**此时依然是两个线程进行CPU时间片的抢夺

```Java
public class ThreadClass {
	public static void main(String[] args) {
		threadYield();
	}
	/***线程的礼让***/
	public static void threadYield() {
		Runnable r=()->{
			for(int i=0;i<10;i++) {
				System.out.println(Thread.currentThread().getName()+":"+i);
				//线程礼让
				if(i==3) {
					Thread.yield();
				}
			}
		};
		Thread t1=new Thread(r,"Thread-1");
		Thread t2=new Thread(r,"Thread-2");
		
		t1.start();
		t2.start();
	}
}
/*输出结果：
Thread-2:0
Thread-2:1
Thread-2:2
Thread-2:3   //Thread-2礼让，CPU被Thread-1抢到
Thread-1:0
Thread-1:1
Thread-1:2
Thread-1:3  //Thread-1礼让，但是CPU还是被Thread-1抢到，Thread-1继续执行
Thread-1:4
Thread-1:5
Thread-1:6
Thread-1:7
Thread-1:8
Thread-1:9   //Thread-1执行完毕，Thread-2接着执行
Thread-2:4
Thread-2:5
Thread-2:6
Thread-2:7
Thread-2:8
Thread-2:9
*/
```

#### 线程合并join

* 执行join的线程，在该过程中，其他线程阻塞，待此线程执行完毕，再执行其他线程。（插队）
* 抛出InterruptException异常

```Java
public class JoinTest {

	public static void main(String[] args) {
		Runnable runnable=()->{
			for(int i=0;i<100;i++) {
				System.out.println("vip线程"+i);
			}
		};
		
		Thread thread=new Thread(runnable);
		thread.start();
		//主线程输出100次
		for(int i=0;i<100;i++) {
			/*
			 * 当主线程运行到第50次时，调用join方法，那么此时会等join方法加入的线程执行完毕，在执行主线程
			 * */
			if(i==50) {
				try {
					thread.join();
				} 
				catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			System.out.println("main"+i);
		}
	}
}
/*输出：在50之前，主线程和子线程交替执行，但是等到主线程为50时，此时子线程会执行直到100结束，然后主线程才执行
*/
AI写代码
java
运行

1
```

#### 守护线程setDaemon

- 如果所有的用户线程结束，那么守护线程会自动死亡；虚拟机不需要等待守护线程执行结束
- setDaemon默认是false，如果要设置一个线程为守护线程，则改为true即可

```Java
public class DaemonTest {
	public static void main(String[] args) {
		Runnable r1=()->{
			while(true) {
				System.out.println("守护线程");
			}
		};
		
		for(int i=0;i<10;i++) {
			System.out.println("主线程"+i);
		}
		
		Thread thread=new Thread(r1);
		thread.setDaemon(true);  //默认是false，表示用户线程
		thread.start();
	}
}
//守护线程是一个死循环，但是等待主线程执行结束后，该线程会自动停止
```



### 获取当前线程的引用

| 方法                                  | 说明                   |
| ------------------------------------- | ---------------------- |
| public static Thread currentThread(); | 返回当前线程对象的引用 |

```Java
public class ThreadDemo {
    public static void main(String[] args) {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName());
    }
}
```

### 休眠当前线程

* 这个方法只能保证实际休眠时间是大于等于参数设置的休眠时间的。

| 方法                                                         | 说明               |
| ------------------------------------------------------------ | ------------------ |
| public staticvoid sleep(long millis) throws interruptedException | 休眠当前线程       |
| public staticvoid sleep(long millis, int nanos) throws interruptedException | 可以更高精度的休眠 |

```Java
public class ThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        System.out.println(System.currentTimeMillis());
        Thread.sleep(3 * 1000);
        System.out.println(System.currentTimeMillis());
    }
}
```



## 线程的状态

### 线程的所有状态

| 状态名        | 具体状态                                    |
| ------------- | ------------------------------------------- |
| NEW           | 安排了工作,还未开始行动                     |
| RUNNABLE      | 可工作的.又可以分成正在工作中和即将开始工作 |
| BLOCKED       | 这几个都表示排队等着其他事情                |
| WAITING       | 这几个都表示排队等着其他事情                |
| TIMED_WAITING | 这几个都表示排队等着其他事情                |
| TERMINATED    | 工作完成了                                  |

## 线程安全

### 产生的原因

* 当一个线程在访问并操作某个资源的过程中，还没来得及完全修改该资源，CPU时间片就被其他线程抢走

```Java
//用四个线程模拟四个售票员卖票，仓库中的余票即为临界资源
class TicketCenter{
	//描述剩余票的数量
	public static int restCount=100;
}
public class SourseProblem {
	public static void main(String[] args) {
		Runnable r=()->{
			//当余票大于0时，可以继续售票
			while(TicketCenter.restCount>0) {
				System.out.println(Thread.currentThread().getName()+"卖出一张票，剩余"+ --TicketCenter.restCount+"张");
			}
		};
		//四个线程模拟四个售票员，线程名模拟售票员名
		Thread t1=new Thread(r,"Thread-1");
		Thread t2=new Thread(r,"Thread-2");
		Thread t3=new Thread(r,"Thread-3");
		Thread t4=new Thread(r,"Thread-4");
		
		t1.start();
		t2.start();
		t3.start();
		t4.start();
	}
}
```

出现临界资源问题，这是因为一个线程在计算余票的过程中，还没来的及将计算、或计算后的结果还没来得及赋给restCount，CPU就被其他线程抢走，此时其他线程中的余票是当前抢到时刻的余票值

### 解决方法

> JVM实现的synchronized

> JDK实现的ReentrantLock

#### 方式一：使用同步代码块

* 用synchronized修饰多线程需要访问的代码

```Java
class TicketCenter{
	//描述剩余票的数量
	public static int restCount=100;
}
public class SourseProblem {
	public static void main(String[] args) {
		Runnable r=()->{
			//当余票大于0时，可以继续售票
			while(TicketCenter.restCount>0) {
				//同步监视器
				synchronized("") {
					if(TicketCenter.restCount<=0) {
						return;
					}
					System.out.println(Thread.currentThread().getName()+"卖出一张票，剩余"+ --TicketCenter.restCount+"张");
				}
			}
		};
		//四个线程模拟四个售票员，线程名模拟售票员名
		Thread t1=new Thread(r,"Thread-1");
		Thread t2=new Thread(r,"Thread-2");
		Thread t3=new Thread(r,"Thread-3");
		Thread t4=new Thread(r,"Thread-4");
		
		t1.start();
		t2.start();
		t3.start();
		t4.start();
	}
}
```

#### 方法二：同步方法：使用关键字synchronized修饰的方法

将上面的同步代码段用一个方法实现

1. 静态方法：同步监视器就是：当前类.class
2. 非静态方法：同步监视器是 this

```Java
class TicketCenter{
	//描述剩余票的数量
	public static int restCount=100;
}
public class SourseProblem {
	public static void main(String[] args) {
		Runnable r=()->{
			while(TicketCenter.restCount>0) {
				soldTicket();
			}
		};
		Thread t1=new Thread(r,"Thread-1");
		Thread t2=new Thread(r,"Thread-2");
		Thread t3=new Thread(r,"Thread-3");
		Thread t4=new Thread(r,"Thread-4");
	
		t1.start();
		t2.start();
		t3.start();
		t4.start();
	}
	//同步方法
	public synchronized static void soldTicket(){
		if(TicketCenter.restCount<=0) {
			return;
		}
		System.out.println(Thread.currentThread().getName()+"卖出一张票，剩余"+ --TicketCenter.restCount+"张");
	}
}
```

#### 方式三：同步锁

* 显式定义同步锁对象来实现同步

```Java
class TicketCenter{
	//描述剩余票的数量
	public static int restCount=100;
}
public class SourseProblem {
	public static void main(String[] args) {
		//实例化一个锁对象
		ReentrantLock rt=new ReentrantLock();
		
		Runnable r=()->{
			while(TicketCenter.restCount>0) {
				//对临界资源上锁
				rt.lock();
				
				if(TicketCenter.restCount<=0) {
					return;
				}
				System.out.println(Thread.currentThread().getName()+"卖出一张票，剩余"+ --TicketCenter.restCount+"张");
				
				//对临界资源解锁
				rt.unlock();
			}
		};
		Thread t1=new Thread(r,"Thread-1");
		Thread t2=new Thread(r,"Thread-2");
		Thread t3=new Thread(r,"Thread-3");
		Thread t4=new Thread(r,"Thread-4");
		
		t1.start();
		t2.start();
		t3.start();
		t4.start();
	}
}
```

### 死锁

* 多个线程彼此持有对方所需要的锁，而不释放自己的锁

```Java
//线程A、B互相等待对方释放拥有的锁
public class DeadLock {
	public static void main(String[] args) {
        
		Runnable runnable1=()->{
			synchronized("A"){
				System.out.println("A线程持有了A锁，等待B锁");
				//此时A线程已经持有A锁了，让它继续持有B锁
                /*为了确保产生死锁
                try {
					Thread.sleep(1000);
				} 
				catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}*/
                
				synchronized("B"){
					System.out.println("A线程持有了A锁和B锁");
				}
			}
		};
		
		Runnable runnable2=()->{
			synchronized("B"){
				System.out.println("B线程持有了B锁，等待A锁");
				//此时B线程已经持有B锁了，让它继续去持有A锁
				synchronized("A"){
					System.out.println("B线程持有了A锁和B锁");
				}
			}
		};
		
		Thread t1=new Thread(runnable1);
		Thread t2=new Thread(runnable2);
		
		t1.start();
		t2.start();
	}
}
/*输出结果：
B线程持有了B锁，等待A锁
A线程持有了A锁，等待B锁
(程序未结束)
*/
```

* 上述代码其实不能完全产生死锁，如果在A线程获取B锁之前，B线程都没有获得执行机会，那么B线程就不会获取到B锁，此时程序依然会执行，不会产生死锁。为了一定产生死锁情况，可以在A线程执行过程中调用一个sleep方法。

### 线程通信：解决死锁的办法

#### 方式一：synchronized下的通信

* wait()：等待，当前的线程释放对同步监视器的锁定，并且让出CPU资源，使得当前的线程进入等待队列中
* notify()：通知，唤醒在此同步监视器上等待的一个线程（具体哪一个由CPU决定），使这个线程进入锁池
* notifyAll()：通知，唤醒在此同步监视器上等待的所有线程，使这些线程进入锁池

```Java
public class DeadLock {
	public static void main(String[] args) {
        
		Runnable runnable1=()->{
			synchronized("A"){
				System.out.println("A线程持有了A锁，等待B锁");
				//A线程释放A锁(捕获异常)
				try {
					"A".wait();
				} 
				catch (InterruptedException e) {
					e.printStackTrace();
				}
				
				synchronized("B"){
					System.out.println("A线程持有了A锁和B锁");
				}
			}
		};
		
		Runnable runnable2=()->{
			synchronized("B"){
				System.out.println("B线程持有了B锁，等待A锁");
				
				synchronized("A"){
					System.out.println("B线程持有了A锁和B锁");
					//此时B线程已经执行完成了，但是A线程任然还在等待，因此需要唤醒A线程
					"A".notify();
				}
			}
		};
		
		Thread t1=new Thread(runnable1);
		Thread t2=new Thread(runnable2);
		
		t1.start();
		t2.start();
	}
}
/*输出结果：
A线程持有了A锁，等待B锁
B线程持有了B锁，等待A锁
B线程持有了A锁和B锁
A线程持有了A锁和B锁
*/
AI写代码
java
运行

```

#### 方式二：Lock锁下的通信，采用Condition控制通信。JUC中的类（java.util.comcurrent类）

- await()：等价于wait()
- signal()：等价于notify()
- signalAll()：等价于notifyAll()

### 多线程下的单例类

- 懒汉式单例类会出现问题

```Java
//定义一个单例类
class Boss{
	//构造器私有化
	private Boss() {
		System.out.println("一个Boss对象被实例化了");
	}
	private static Boss instance=null;
	//外部类只能通过该方法获取Boss类的实例
	public static Boss getBoss() {
		if(instance==null) {
			instance=new Boss();
		}
		return instance;
	}
}
public class SingletonTest {
	public static void main(String[] args) {
		Runnable runnable=()->{
			Boss.getBoss();
		};
		//开辟了100条线程去获取这Boss实例
		for(int i=0;i<100;i++) {
			new Thread(runnable).start();
		}
	}
}
```

* 当多线程去执行这个单例类时，还是希望只产生一个实例对象，但程序输出结果明显不是，这是由于多线程导致的

#### 修改方式1：对临界资源上锁，使用同步代码

```Java
//定义一个单例类
class Boss{
	//构造器私有化
	private Boss() {
		System.out.println("一个Boss对象被实例化了");
	}
	private static Boss instance=null;

	public static Boss getBoss() {
        //同步代码段
		synchronized("") {
			if(instance==null) {
				instance=new Boss();
			}
		}
		return instance;
	}

}

public class SingletonTest {
	public static void main(String[] args) {
		Runnable runnable=()->{
			Boss.getBoss();
		};
		//开辟了100条线程去获取这Boss实例
		for(int i=0;i<100;i++) {
			new Thread(runnable).start();
		}
	}
}
```

#### 修改方式2：对临界资源上锁，使用同步方法

```Java
class Boss{
	//构造器私有化
	private Boss() {
		System.out.println("一个Boss对象被实例化了");
	}
	private static Boss instance=null;
	//同步方法
	public static synchronized Boss getBoss() {
		if(instance==null) {
			instance=new Boss();
		}
		return instance;
	}
}

public class SingletonTest {
	public static void main(String[] args) {
		Runnable runnable=()->{
			Boss.getBoss();
		};
		//开辟了100条线程去获取这Boss实例
		for(int i=0;i<100;i++) {
			new Thread(runnable).start();
		}
	}
}
```

## 线程池

> 线程池在系统启动时就创建大量空闲的线程。提前创建多个线程，放入线程池，使用时直接从线程池中获取，使用完放回池中

### 作用

可以避免频繁创建销毁线程的过程，实现充分利用

- corePoolSize:核心池的大小（可以放多少个线程）
- maximumPoolSize:最大线程数（一次可以同时运行的线程数量）
- keepAliveTime:线程没有任务时最多保持多长时间后会终止

### 创建方式

* ExecutorService接口：线程池真正的接口
* Executor：创建线程的工具类，调用该类的newFixedThreadPool(corePoolSizesize)方法来创建线程池
* execute：执行Runnable接口的，无返回值
* Future submit：执行Callable接口的，有返回值
* shutdown:关闭连接

```Java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolTest {
	public static void main(String[] args) {
		Runnable r=()->{
			System.out.println(Thread.currentThread().getName());
		};
		//创建线程池，设置大小为10
		ExecutorService service=Executors.newFixedThreadPool(10);
		//执行
		service.execute(r);
		service.execute(r);
		service.execute(r);
		service.execute(r);
        //关闭连接
		service.shutdown();
	}
}
/*输出结果：
pool-1-thread-3
pool-1-thread-4
pool-1-thread-2
pool-1-thread-1
*/
```

## JUC组件

### 未来任务FutureTask

* 利用Callable创建线程时，有返回值，该值由Future进行封装，FutureTask实现了RunnableFuture接口，而该接口继承自Runnable和Future接口，因此FutureTask既可以当做一个任务执行，也可以有返回值。

* 当计算一个任务需要很长时间时，可使用FutureTask来封装这个任务，使得主线程在完成自己的任务后在去获取这个计算结果

```Java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class FutureTaskTest {
	public static void main(String[] args) {
		//创建一个Clallable接口，有返回值，给子线程执行
		Callable<Integer> cla=()->{
			int result=0;
			for(int i=0;i<100;i++) {
				Thread.sleep(10);  //每一次计算时都让让主线程执行一段时间
				result+=i;
			}
			return result;
		};
		//新建一个FutureTask实例
		FutureTask<Integer> futureTask=new FutureTask<>(cla);
		//执行计算任务的线程
		Thread t1=new Thread(futureTask);
		t1.start();
		
        //创建Runnable接口，给主线程执行
		Runnable runnable=()->{
			System.out.println("主线程任务正在执行");
			try {
				Thread.sleep(10);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		};
		
		Thread t2=new Thread(runnable);
		t2.start();
		
		//得到有返回值的输出
		try {
			System.out.println(futureTask.get());
		} catch (InterruptedException | ExecutionException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
/*输出结果：
另一个线程任务正在执行
4950
*/
//如果将Callable执行体中的Thread.sleep(10);去掉，则执行结果为：4950  另一个线程任务正在执行。
```

### 阻塞队列BlockingQueue

* 利用BlockingQueue作为线程同步的工具，主要用来实现消费者生产者设计模式

### 叉链接ForkJoin

* 主要用于并行计算中，将大的任务分成小的任务进行计算，再把小任务的结果合并成总的计算结果
