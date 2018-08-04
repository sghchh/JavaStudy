#Java线程  
###一 认识线程  
java线程转换图：
![](http://static.zybuluo.com/kiraSally/ouapg7k7ijvkbgjpby6y814u/Java%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2.jpg)

* 线程的状态转换  
	* 新建(new)：新创建一个线程对象:在JAVA中的表现就是Thread thread = new Thread()  
	* 就绪(runnable)：线程创建后，其他线程调用该对象的start方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU时间分片使用权：在JAVA中的表现就是thread.start()  
	* 运行(running)：就绪态线程获取到CPU时间分片之后，就可以执行任务：在JAVA中的表现就是thread.run()，但需要注意的是，此时线程不一定是立即执行，这跟系统调度有关  
	* 阻塞(block)：阻塞状态是指线程因为某种原因放弃CPU使用权，让出CPU时间片，暂时停止运行（注意此时线程在内存中是存在的，并没有被GC），直到线程进入就绪态，才有机会再次获取时间片转成运行态。阻塞分三种情况：  
		*  常规阻塞：运行态线程在发出I/O请求、执行Thread.sleep方法、t.join方法时，JVM会将该线程设置为阻塞状态；当I/O处理完毕并发出响应、sleep方法超时、join等待线程终止或超时，线程重新转入就绪态  
		*  同步阻塞：运行态线程在获取对象的同步锁时，当该同步锁已被其他线程占用，JVM会将该线程暂时放入同步队列中，当其他线程放弃同步锁同时该线程竞争到该同步锁时，该线程转为就绪态  
		*  等待阻塞：运行态线程执行wait方法，JVM会将该线程放入等待队列中，直到被其他线程唤醒或其他线程中断或超时，再次获取同步锁标识，重新进入就绪态`  
	* 死亡(dead)：线程main方法、run方法执行完毕或出现异常，则该线程生命周期结束。已死亡线程不可复生，当然可以使用线程池机制来提高线程的复用性，避免线程被直接杀死  
###二 线程控制  
* sleep方法：  
	* 该方法是一个**静态方法**，所以调用的时候使**当前线程**进入睡眠状态  
	* 该方法执行后会使线程陷入**阻塞状态**  
	* 该方法执行后**不会释放锁**
* yield方法：  
	* 该方法也是一个**静态方法**，调用后会让当前线程进入**就绪状态**，即让线程调度器重新调度一次。完全有可能出现执行完yield方法后该线程继续执行了。 
	* 该方法执行后**会释放锁** 
* suspend方法：  
	* 该方法是一个**实例方法**，没有参数，调用后线程进入**暂停**状态  
	* 解除**暂停**状态需要**调用resume（）方法**显示解除  
	* 该方法执行前必须获得锁
	* 该方法执行后**不会释放锁**
* join方法：  
	* 该方法是一个**实例方法**。比如在main线程中调用方法，“新线程.join()"。则main线程会停止，直到新线程的run方法执行完。  
	* 同时也可以添加long参数，表示最多等待的时间（毫秒）。  
	* 该方法**底层使用wait实现的**，所以该方法执行后**会释放锁**
* resume方法：  
    * 该方法是一个实例方法，可以让经过suspend方法暂停的线程重新恢复进入到就绪态

> 注意到线程控制的解决方案都只是一种“主动停止”的思想：不管是sleep、yield、suspend，他们实现线程控制的方式都是**使得当前执行的线程暂停**，试想当调用Thread.sleep(),Thread.yield()以及thread.suspend()的时候，调用这些方法的线程本身就是正在执行的线程啊，所以，这些方法的效果都是*让调用该方法的线程暂停*

---  
####线程暂停的实现  
* **1. suspend()+resume()方法实现**  

		public class Test {
		public static boolean isThread1=true;
		public static int count=0;
		public static Thread thread1,thread2;
		public Test(){
			thread1=new Thread(new Runnable(){
				public void run() {
					while(Test.count!=10)
				// TODO Auto-generated method stub
					{
						Test.count++;
						System.out.println(Thread.currentThread().getName()+"滴答");
						Test.isThread1=false;
						Test.thread2.resume();
						Thread.currentThread().suspend();
					}
				}
			
			},"thread-1");
			thread2=new Thread(new Runnable(){

				public void run() {
				// TODO Auto-generated method stub
					while(Test.count!=100)
						// TODO Auto-generated method stub
						{
					    	Test.count++;
							System.out.println(Thread.currentThread().getName()+"滴答");
							Test.isThread1=true;
							Test.thread1.resume();
							Thread.currentThread().suspend();
						
						}
				}
			
			},"thread-2");
		}
		public static void main(String[] args)
		{
			final Test test=new Test();
			Test.thread1.start();
			Test.thread2.start();
		}
		}
		该段代码实现的功能使两个线程轮流打印输出：  
		thread-1滴答
		thread-2滴答
		thread-1滴答
		thread-2滴答
		thread-1滴答
		thread-2滴答
		thread-1滴答
		thread-2滴答
		thread-1滴答
		thread-2滴答  
使用这种方式有两种危险：  
**1. 独占**

		//如果使用不当，很容易造成公共的同步对象的独占，是其他线程无法访问公共同步对象
		public class ThreadDemo {
    		synchronized void echo(){
        		System.out.println("开始任务");
        		if (Thread.currentThread().getName().equals("sally")){
            		System.out.println("sally线程执行suspend方法并永远暂停");
            		Thread.currentThread().suspend();
        		}
    		}
    		public static void main(String[] args) throws InterruptedException {
        		final ThreadDemo threadDemo = new ThreadDemo();
        		Thread t1 = new Thread(new Runnable() {
            		@Override
            		public void run() {
                		threadDemo.echo();
            		}
        		}, "sally");
        		t1.start();
        		Thread t2 = new Thread(new Runnable() {
            				@Override
            				public void run() {
                		System.out.println("kira线程启动，但无法执行echo方法");
                		//因为echo方法被sally线程锁定并且永远suspend暂停了，无法释放，即独占
                		threadDemo.echo();
            		}
        		}, "kira");
        		t2.start();
    		}
		}
		-------------
		//输出：
		开始任务
		sally线程执行suspend方法
		kira线程启动，但无法执行echo方法
>占有同步资源，在suspend（）方法调用之前无法及时释放资源  

**2. 不同步**  

		//如果使用不当，也容易出现因为线程的暂停而导致线程不同步的问题
		public class ThreadDemo {
    		private String value = "战狼1";
    		public String getValue() {
        		return value;
    		}
    		public void setValue(String value) {
        		if (Thread.currentThread().getName().equals("sally")){
            		System.out.println("停止sally线程");
            		Thread.currentThread().suspend();
        		}
        		this.value = value;
    		}
    		public static void main(String[] args) throws InterruptedException {
        		final ThreadDemo threadDemo = new ThreadDemo();
        		Thread t1 = new Thread(new Runnable() {
            		@Override
            		public void run() {
                		threadDemo.setValue("战狼2");
            		}
        		}, "sally");
        		t1.start();
        		Thread.sleep(1000);
        		Thread t2 = new Thread(new Runnable() {
            		@Override
            		public void run() {
                		System.out.println(threadDemo.getValue());
            		}
        		}, "kira");
        		t2.start();
    		}
		}
		//输出：
		停止sally线程
		战狼1 --> 很明显，由于线程暂停，得到的不是想要的战狼2  
>被暂停线程的run()方法没有执行完，导致在里面进行的修改没有及时实现  

* **2. sleep()方法实现线程暂停**  
* **3. yield()方法提出让出CPU，重新进行一次分配（可能执行完还是这个线程被调用）** 
###三 线程同步  
由于多线程的并发操作，线程的同步就会出现问题，解决方法就是引入了**同步监视器和同步锁**：  
####3.1 synchronized关键字  

**同步关键字synchronized用于对共享资源的修饰，如果不是共享资源，没有必要使用同步关键字。**
##### 3.1.1 同步方法  

同步方法是指一个方法用synchronize修饰，这样的话**同步锁就是该类的一个实例对象**：

	class MyObject{
		synchronized public void method() throws InterruptedException{
			System.out.println("当前执行的线程是："+Thread.currentThread().getName());
			Thread.sleep(500);
			System.out.println("从"+Thread.currentThread().getName()+"线程处返回");
		}
	}
	class MyThreadA extends Thread{
		private MyObject o;
		MyThreadA(MyObject o){
			this.o=o;
		}
		
		public void run(){
			try {
				o.method();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	class MyThreadB extends Thread{
		private MyObject o;
		MyThreadB(MyObject o){
			this.o=o;
		}
		
		public void run(){
			try {
				o.method();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	public class Run{
		public static void main(String[] args){
			MyObject o=new MyObject();
			MyThreadA thread=new MyThreadA(o);
			thread.start();
			MyThreadB threadb=new MyThreadB(o);
			threadb.start();
		}
	}  
	--------
	输出：
		当前执行的线程是：Thread-0
		从Thread-0线程处返回
		当前执行的线程是：Thread-1
		从Thread-1线程处返回  

从输出的结果来看，虽然thread-0中间有调用sleep(500)但是thread-1并没有执行o.method()方法，从而实现了方法的同步调用。  

而synchronized关键字的同步锁必须是同一个对象才能够约束住多线程，如果将main方法改成下面这样，结果就不一样了：

	public class Run{
		public static void main(String[] args){
			MyObject o=new MyObject();
			MyThreadA thread=new MyThreadA(o);
			thread.start();
			MyThreadB threadb=new MyThreadB(new MyObject());
			threadb.start();
		}
	}
	----
	当前执行的线程是：Thread-0
	当前执行的线程是：Thread-1
	从Thread-0线程处返回
	从Thread-1线程处返回  

而synchronized关键字也可以作用到静态方法上，意思就是为Class上锁，Class锁可以对类的所有对象的实例起作用。  

	class MyObject{
		synchronized public static void methodA() throws InterruptedException{
			System.out.println("当前执行的线程是："+Thread.currentThread().getName());
			Thread.sleep(500);
			System.out.println("从"+Thread.currentThread().getName()+"线程处返回");
		}
		synchronized public static void methodB() throws InterruptedException{
			System.out.println("当前执行的线程是："+Thread.currentThread().getName());
			System.out.println("从"+Thread.currentThread().getName()+"线程处返回");
		}
		synchronized public void methodC(){
			System.out.println("当前执行的线程是："+Thread.currentThread().getName());
			System.out.println("从"+Thread.currentThread().getName()+"线程处返回");
		}
	}
	class MyThreadA extends Thread{
		private MyObject o;
		MyThreadA(MyObject o){
			this.o=o;
		}
		
		public void run(){
			try {
				o.methodA();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	class MyThreadB extends Thread{
		private MyObject o;
		MyThreadB(MyObject o){
			this.o=o;
		}
		
		public void run(){
			try {
				o.methodB();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	class MyThreadC extends Thread{
		private MyObject o;
		MyThreadC(MyObject o){
			this.o=o;
		}
		
		public void run(){
				o.methodC();
		}
	}
	
	
	public class Run{
		public static void main(String[] args){
			MyObject o=new MyObject();
			MyThreadA thread=new MyThreadA(o);
			thread.start();
			MyThreadB threadb=new MyThreadB(new MyObject());
			threadb.start();
			MyThreadC threadc=new MyThreadC(o);
			threadc.start();
		}
	}  
	------  
	当前执行的线程是：Thread-0
	当前执行的线程是：Thread-2
	从Thread-2线程处返回
	从Thread-0线程处返回
	当前执行的线程是：Thread-1
	从Thread-1线程处返回  

可以看到，即使threadb和threada传入的对象不一样，这两个线程调用的方法也不是同一个方法，但是他们两个的调用顺序也是同步的；而threadc和threada传入的对象时同一个，但是他们的调用是异步的。就是因为threada和threadb调用的都是static方法，所以**即使threadb传入的对象和threada传入的对象不是同一个，那也没什么帮助，因为他们两个的同步锁是Class类**；但是threadc调用的是一个对象级别的锁，所以和threada就发生了异步情况

> 另外，synchronized的同步锁具有**重入性，意思就是当一个线程得到一个对象锁后，再次请求此对象锁时是可以得到该对象的锁的，也就是说在synchronized修饰的方法内部还可以调用另一个synchronized修饰的方法。**否则的话会发生死锁（持有锁，但是却在同一个对象的另一个synchronized方法的等待队列里面等待）。
> **可重入锁支持在子类的synchronized方法中调用父类的synchronized方法。**


#####3.1.2 同步代码块    
		synchronized（obj）{
			...  
			//此处的代码块就是同步代码块 
		}  
synchronized后面括号中的obj就是**同步监视器**，上面代码的含义是：线程开始执行同步代码块之前，必须先获得**对同步监视器的锁定**。在多线程并发操作临界区（共享资源）的时候，任意时刻只有一个线程可以获得对同步监视器的锁定，**当同步代码块执行完毕后释放锁定**。如果想像同步方法一样让类的对象作为同步监视器，那么**括号中就填this**；同样，如果**括号中的是一个XXX.class那么就和类同步方法一样了**。同步代码块儿就像是**一半同步，一半异步**的类的实例方法一样。  
和同步方法一样，如果不同调用对应的对象不同则就是两个不同的同步监视器，并不会产生同步效果；只有对应的对象相同，才会产生同步效果。

####3.2 同步锁  
同步锁需要显示定义一个Lock对象，**每次只能有一个线程对Lock对象加锁**，线程开始访问共享资源之前必须获得Lock对象。Lock与ReadWriteLock（读写锁）是两个**根接口**。ReentrantLock（可重入锁，可以嵌套似的加锁）和ReentrantReadWriteLock分别是两个的实现类。  

		class X{
			//定义锁对象  
			private final ReentrantLock lock=new ReentrantLock();
			...
			void methord(...){
				lock.lock();
				...
				finally
				{  
					lock.unlock();  
				}  
			}  
		}  
这就是同步锁的使用  
###四 线程通信   
####4.1 传统的线程通信  
![](http://static.zybuluo.com/kiraSally/w92amiesxtqq1jn9zlj08jzj/image_1bng05nfa14ij3as3eab3ode8l.png)  
值得注意的是这三种方法都是**Object类提供的方法，由Object对象来调用（也就是同步监视器）**，也就是这种传统的通信方法**适用于使用synchronized关键字来保持同步，拥有同步监视器的情况**。  
#####4.1.1 wait（）方法  
>* wait方法使当前线程进行等待，该方法是Object类的方法，用来将当前线程放入"等待队列"中，并在wait所在的代码处停止执行，直到收到通知被唤醒或被中断或超时  
* 调用wait方法之前，线程必须获得该对象的对象级别锁，即只能在同步方法或同步块中调用wait方法  
* 在执行wait方法后，当前线程释放锁，在从wait方法返回前，线程与其他线程竞争重新获得锁  
* 如果调用wait方法时没有持有适当的锁，则抛出运行期异常类IllegalMonitorStateException
#####4.1.2 notify（）方法  
* notify方法**使线程被唤醒**，该方法是Object类的方法，用来将线程**从"等待队列中"移出到"同步队列中"**，线程状态重新变成**阻塞状态**，notify方法所在同步块释放锁后，从wait方法返回继续执行
* 在执行notify方法之后，**当前线程不会马上释放对象锁**，等待线程也并不能马上获取该对象锁，需要等到执行notify方法的线程将程序执行完，即**退出同步代码块之后当前线程才能释放锁**，而等待线程才可以有机会获取该对象锁  
* 如果有多个线程都在此同步监视器上等待，则会选择唤醒其中一个线程  
* 调用notify()或notifyAll()方法之后，等待线程不会从wait()返回，需要notify()方法所在同步块代码执行完毕而释放锁之后，等待线程才可以**获取到该对象锁并从wait()返回**  
####4.2 使用join方法实现线程通信  
join方法也可以实现线程通信 
  
		Thread t1 = new Thread(new Runnable() {
    		@Override
    		public void run() {
       			 for (int i = 0;i<100;i++){
          		  System.out.println(Thread.currentThread().getName()+"线程值为:sally" + i);                 
				 }
    		}
		},"sally");
		Thread t2 = new Thread(new Runnable() {
   			 @Override
    		public void run() {
       			 for (int i = 0;i<2;i++){
          		 	System.out.println(Thread.currentThread().getName()+"线程值为:kira" + i);
       			 }
   			}
		},"kira");
		t1.start();
		t1.join();//让t2线程和后续线程无限等待直到sally线程执行完毕
		t2.start();
		-------------
		//输出：
		......
		sally线程值为:sally97
		sally线程值为:sally98
		sally线程值为:sally99
		kira线程值为:kira0 //可以发现直到sally线程执行完毕，kira线程才开始执行
		kira线程值为:kira1  
###五 线程中断  
线程终止都会释放同步监视器。  
####5.1 Java线程中断的相关方法  
* 使用退出标识，使线程正常退出，即当run方法执行完成后线程自动终止  
* 使用stop方法强行终止，已被弃用  
* 使用interrupt（）方法中断线程  
####5.2 interrupt方法  
表示中断一个线程  

* 只能被自己中断，否则有可能抛出异常  
* 若当前线程已经被Object.wait()，Thread的join(),sleep方法阻塞，则调用该方法会抛出异常，**且中断状态会被清除**  

**调用interrupt()会遵循如下准则：**

1. 当前线程**只能被自己中断**，否则抛出SecurityException异常
2. **若当前线程已被Object.wait()、thread.join()或thread.sleep()阻塞，将抛出InterruptedException，同时清除中断状态**
3. 若当前线程在InterruptibleChannel上发生IO阻塞，该通道要被关闭并抛出ClosedByInterruptException异常，同时设置中断状态
4. 若当前线程被Selector阻塞，将线程状态设置为中断同时从选取方法中立即返回（通常是）非0值
5. 非上述情况，仅仅只是修改状态位，不会抛异常，针对异常情况可自行捕获处理
6. 中断非活动状态线程无意义
####5.3 isInterrupted方法  
判断是否处于中断状态  

* 调用的线程不清除状态标识  
isInterrupted(boolean):带参数的方法，参数表示是否清除状态标识，false标识不清除，true标识清除  
####5.4 interrupted方法    
返回当前正在执行的线程是否处于中断状态，**底层是currentThread().isInterrupted(false);**所以这个判断的是当前正在执行的线程的状态，也就是 Thread1.interrupted（）方法调用后可能返回的是Thread2是否处于中断状态的布尔值，如果是当前线程自己执行该方法，那么就一定返回的true了。  
####5.5 中断线程的方法 
**interrupt()方法调用后只是更改了那个布尔值，只是建议中断该线程，并没有真的为我们实现中断** 
#####5.5.1 异常法  
线程在抛出异常后会主动终止，所以该方法就是手动抛出异常  

	class MyThread extends Thread{
		public void run(){
			super.run();
			try{
				for (int i=0;i<5000000;i++){
					if (this.interrupted()){
						System.out.println("已经是停止状态了！我要退出了！当前的i为"+i);
						throw new InterruptedException();
					}
				}
			}catch(InterruptedException e){
				System.out.print("进入MyThead.java类run方法中的catch了");
				e.printStackTrace();
			}
		}
	}
	
	public class Run{
		public static void main(String[] args) throws InterruptedException{
			MyThread thread=new MyThread();
			thread.start();
			Thread.sleep(5);
			thread.interrupt();
			System.out.println("执行完interrupt了，end");
		}
	}
		-------------
		//执行完interrupt了，end
		已经是停止状态了！我要退出了！当前的i为147312
		进入MyThead.java类run方法中的catch了java.lang.InterruptedException
			at MyThread.run(Run.java:22) 
#####5.5.2 沉睡法  
利用interrupt方法中的**若当前线程已经被Object.wait()，Thread的join(),sleep方法阻塞，则调用该方法会抛出异常**，在沉睡中调用interrupt方法，让其抛出异常 
 
	class MyThread extends Thread{
		public void run(){
			super.run();
			try{
				System.out.println("MyThread的run方法开始了");
				Thread.sleep(2000);
			}catch(InterruptedException e){
				System.out.print("进入MyThead.java类run方法中的catch了,在沉睡中被中断发生异常");
				e.printStackTrace();
			}
		}
	}
	
	public class Run{
		public static void main(String[] args){
			MyThread thread=new MyThread();
			thread.start();
			try {
				Thread.sleep(5);
				thread.interrupt();
			} catch (InterruptedException e) {
				System.out.println("main方法中的catch块儿，thread中断异常");
				e.printStackTrace();
			}
			
			System.out.println("执行完interrupt了，end");
		}
	}
		-------------
		//输出：MyThread的run方法开始了
			执行完interrupt了，end
			进入MyThead.java类run方法中的catch了,在沉睡中被中断发生异常java.lang.InterruptedException: sleep interrupted
				at java.lang.Thread.sleep(Native Method)
				at MyThread.run(Run.java:20)  

而且可以看到，是在线程的run方法中捕获了异常。
			  
#####5.5.3 Return法  
	class MyThread extends Thread{
		public void run(){
			super.run();
				while(true){
					System.out.println("正在执行");
					if (this.interrupted()){
						System.out.println("停止了");
						return;
					}
				}
		}
	}
	
	public class Run{
		public static void main(String[] args){
			MyThread thread=new MyThread();
			thread.start();
			try {
				Thread.sleep(50);
				thread.interrupt();
			}catch(InterruptedException e){
				e.printStackTrace();
			}
			System.out.println("执行完interrupt了，end");
		}
	}
		-------------
		//输出：
		正在执行
		正在执行
		正在执行
		正在执行
		正在执行
		正在执行
		正在执行
		正在执行
		正在执行
		停止了
		执行完interrupt了，end  

从我们以上实现中断的方法来看，中断了的线程和暂停时不一样的，中断了的线程是没法主动的进入到就绪或者运行态了，这算是**中断和暂停的区别来看待**    

### 六、线程通信再学习  
#### 6.1 轮询法  
**Sleep+While(true):**  

	final List<Integer> list = new ArrayList<Integer>();
	Thread t1 = new Thread(new Runnable() {
    	@Override
    	public void run() {
        	try {
            	for (int i = 0 ; i < 6 ;i++){
                	list.add(i);
                	System.out.println("添加了" + (i+1) + "个元素");
            	}
            	Thread.sleep(1000);
        	} catch (InterruptedException e) {
            	e.printStackTrace();
        	}
    	}
	},"sally");
	Thread t2 = new Thread(new Runnable() {
    	@Override
    	public void run() {
        	try {
	            while (true){
	                if (list.size() == 3){
	                    System.out.println("已添加5个元素,kira线程需要退出");
	                    throw new InterruptedException();
	                }
	            }
	        } catch (InterruptedException e) {
	            e.printStackTrace();//这里只是打印堆栈信息，不是真的停止执行
        	}
    	}
	},"kira");
	t1.start();
	t2.start();
	-------------
	//输出：
	添加了1个元素
	添加了2个元素
	添加了3个元素
	已添加3个元素,kira线程需要退出
	java.lang.InterruptedException
	    at concurrent.Main$2.run(Main.java:98)
	    at java.lang.Thread.run(Thread.java:748)
	添加了4个元素
	添加了5个元素
	添加了6个元素  

>分析：由于调用**sleep()方法**的线程不会释放任何监视器，所以如果while的条件是**临界区**的资源，那么该方法恐怕无法奏效；但是如果是普通的线程共享的资源(比如这里的list)，那么便能够实现效果，但是轮询的时间要把握恰当，否则造成时间的浪费、难以保证准时(时间过长)；或者是资源的浪费、开销的增大(轮询时间过短)。

#### 6.2 Object.wait/Object.notify  
![](http://static.zybuluo.com/kiraSally/w92amiesxtqq1jn9zlj08jzj/image_1bng05nfa14ij3as3eab3ode8l.png)  

##### 6.2.1 Object.wait()方法  

* wait方法**使当前线程进行阻塞等待**，该方法是Object类的方法，用来将当前线程**放入"等待队列"中**，并在wait所在的代码处停止执行，直到收到通知**被唤醒或被中断或超时**
* **调用wait方法之前，线程必须获得该对象的对象级别锁**，即只能在同步方法或同步块中调用wait方法
* 在执行wait方法后，**当前线程释放锁**，在从wait方法返回前，线程与其他线程竞争重新获得锁
* 如果调用wait方法时没有持有适当的锁，则抛出运行期异常类IllegalMonitorStateException  

##### 6.2.2 Object.notify()  

* notify方法**使线程被唤醒**，该方法是Object类的方法，用来将当前线程**从"等待队列中"移出到"同步队列中"**，线程状态重新变成**阻塞状态**，notify方法所在同步块释放锁后，从wait方法返回继续执行
* 调用notify方法之前，线程必须获得该对象的对象级别锁，即**只能在同步方法或同步块中调用notify方法**
* 该方法用来通知那么可能等待该对象的对象锁的其他线程，如果有多个线程等待，则由线程规划器从等待队列中随机选择一个WAITING状态线程，对其发出通知转入同步队列并使它等待获取该对象的对象锁
* 在执行notify方法之后，**当前线程不会马上释放对象锁**，等待线程也并不能马上获取该对象锁，需要等到执行notify方法的线程将程序执行完，即**退出同步代码块之后当前线程才能释放锁**，而等待线程才可以有机会获取该对象锁
* 如果调用notify方法时没有持有适当的锁，则抛出运行期异常类IllegalMonitorStateException  

##### 6.2.3 **等待/通知的经典范式（生产者-消费者模式）**  

	final List<Integer> list = new ArrayList<Integer>();
	Object lock = new Object();
	Thread t1 = new Thread(new Runnable() {
	    @Override
	    public void run() {
	        //加锁，拥有lock的Monitor
	        //wait方法必须在synchronized方法或方法块中执行，否则抛出IllegalMonitorStateException
	        synchronized (lock){
	            System.out.println(Thread.currentThread().getName() + "线程开始执行");
	            if (list.size() != 3){
	                System.out.println("wait 开始:" + System.currentTimeMillis());
	                try {
	                    lock.wait();//将当前sally线程放入等待队列中，进入等待状态
	                    System.out.println("wait 结束:" + System.currentTimeMillis());
	                    System.out.println(Thread.currentThread().getName() + "线程结束执行");
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	},"sally");
	Thread t2 = new Thread(new Runnable() {
	    @Override
	    public void run() {
	        //加锁，拥有lock的Monitor
	        //notify方法必须在synchronized方法或方法块中执行，否则抛出IllegalMonitorStateException
	        //由于wait方法会释放锁，所以kira线程可以获取到lock同步锁
	        //同时notify方法不会释放锁，直到该同步块执行完毕
	        synchronized (lock){
	            try {
	                System.out.println(Thread.currentThread().getName() + "线程开始执行");
	                for (int i = 0; i< 6;i++){
	                    list.add(i);
	                    if (list.size() == 3){
	                        System.out.println("notify 开始:" + System.currentTimeMillis());
	                        //随机唤醒一个等待线程，这里因为就只有sally线程被等待，所有就唤醒它
	                        lock.notify();
	                        // lock.notifyAll(); 会一次性唤醒所有的等待线程
	                        System.out.println("notify方法已执行，发出通知");
	                    }
	                    System.out.println("已添加" + ( i +1 ) + "个元素");
	                    Thread.sleep(500);//为了让效果明显一些，我们先暂停500毫秒
	                }
	                System.out.println("notify 结束:" + System.currentTimeMillis());
	                System.out.println(Thread.currentThread().getName() + "线程结束执行");
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	},"kira");
	t1.start();
	t2.start();
	-------------
	//输出：
	sally线程开始执行
	wait 开始:1502699526681
	执行wait方法    
	kira线程开始执行 //注意：wait方法会释放锁，所有kira线程获得锁从而执行
	已添加1个元素
	已添加2个元素
	notify 开始:1502699527682
	notify已方法，发出通知
	已添加3个元素
	已添加4个元素
	已添加5个元素
	已添加6个元素 //注意：notify方法不会释放锁，直到该同步方法/块执行完毕才会释放锁
	notify 结束:1502699529682
	kira线程结束执行
	wait 结束:1502699529682
	sally线程结束执行  

>join的源码非常简单，是用wait实现的；猜测：比如在main线程中新建一个t线程，调用t.join()，则对应的源码中wait()的调用者是main线程，而不是t线程。  

> **综上所述：能够释放锁的只有yield方法和Object.wait()方法。**
