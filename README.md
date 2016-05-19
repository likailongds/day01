# day01
introduce Collections 
我的java笔记之多线程并发
本章目标提供多线程并发程序的设计基础    
1.可以运行多个独立的任务  
2.考虑这些任务关闭时可能出现的问题
3.任务可能会在共享资源发生干涉。互斥是防止这种问题的方式及就是锁
4.如果设计不合理就会出现死锁
明白什么时候使用并发，什么时候避免使用并发是关键，使用他们的主要原因是
1.要处理很多事物，他们交织在一起，应用并发能够更有效的使用计算机
2.能够更好的组织代码
3.更便于用户使用
均衡资源的经典案例是在等待输入输出时使用cpu，更好的代码组织可以在仿真中看到，是用户方的经典案例是在长时间下载的过程中监听停止按钮是否被按下
线程的一个额外好处是提供了轻量级的执行上下文切换，而不是重量级切换，因为一个给定进程内的所有线程共享所有的内存空间，轻量级的上下文切换只改变程序的执行序列和局部变量，进程切换必须改变所有的内存空间
多线程的主要缺陷有
1.等待资源的时候性能降低
2.处理线程的额外cpu花费
3.糟糕的程序导致不必要的复杂度
4.有可能产生一些病态的行为如饿死，竞争，死锁，活锁
5.不同平台导致的不一定性
6.因为多个线程可能会共享一个资源，比如一个对象的内存，而且你必须确定多个线程不会同时读取和改变这个资源，这就是线程的最大难题，明智的使用加锁机制例如synchronized
关键字。
案例1建立线程任务有两种方式 实现Runnable接口这里可以使用匿名内部类来实现或者继承Thread
package cn.itcast.javasethread;
public class LiftOff implements Runnable {
    protected int countDown=10;
    private static int taskCount=0;
    private   final  int  id=taskCount++;//一旦被初始化不希望被改变
	public LiftOff() {//无参构造方法
	}
	public LiftOff(int countDown) {//带参构造方法传入时间
      this.countDown=countDown;
	}
	public String status(){
		return "#"+id+"("+( countDown>0? countDown:"LiftOff")+")";
	}
	@Override
	public void run() {
		while(countDown-->0){
			System.out.println(status());
			Thread.yield();//静态方法
		}
	}
}
package cn.itcast.javasethread;
public class MainThread {
	public static void main(String[] args) {
       // LiftOff  launch=new LiftOff();
       // launch.run();//这个是普通方法的调用不是多线程
        Thread t=new Thread(new LiftOff());
        t.start();
	}
}
#0(9)#0(8)#0(7)#0(6)#0(5)#0(4)#0(3)#0(2)#0(1)#0(LiftOff)
package cn.itcast.javasethread;
public class MoreThread {
	public static void main(String[] args) {
     for (int i = 0; i < 5; i++) 
		new Thread(new LiftOff()).start();
		System.out.println("wait for liftdown");
	}
}
线程池
package cn.itcast.javasethread;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class CachedThreadPool {
	public static void main(String[] args) {
		ExecutorService exec=Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) 
			exec.execute(new LiftOff());
			exec.shutdown();
	}
}
线程池2
package cn.itcast.javasethread;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class FixedThreadPool {
	public static void main(String[] args) {
       ExecutorService exec=Executors.newFixedThreadPool(5);//建立有限的线程。限制线程数量
       for(int i=0;i<5;i++)
    	   exec.execute(new LiftOff());
           exec.shutdown();
	}
}
package cn.itcast.javasethread;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class SingleThreadExecutor {
	public static void main(String[] args) {
		ExecutorService exec = Executors.newSingleThreadExecutor();//确保任何时候都只有一个线程在运行
		for (int i = 0; i < 5; i++)
			exec.execute(new LiftOff());
		exec.shutdown();
	}
}
package cn.itcast.javasethread;

import java.util.ArrayList;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
/**
 * submit方法会产生Future对象，他用Callable接口返回特定的结果的特定类型进行了参数化，可以用isdone方法
 * 查看任务是否玩完成
 * @author likailong
 */
public class CallableDemo {
	public static void main(String[] args) {
		ExecutorService  exec=Executors.newCachedThreadPool();
		ArrayList<Future<String>>results=new ArrayList<>();
		for(int i=0;i<10;i++)
			results.add(exec.submit(new TaskWithResult(i)));
		for(Future<String> fs:results)
			try {
				System.out.println(fs.get());
			} catch (InterruptedException e) {
				
				e.printStackTrace();
			} catch (ExecutionException e) {
				
				e.printStackTrace();
			}finally{
				exec.shutdown();
			}
	}
}
