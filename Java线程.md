#Java线程    
### 背景补充  
#### 1. CPU密集型和IO密集型  
之前一直疑惑，在单核情况下，对于不同的线程，同时只能运行一个，那么和单线程相比，所有的任务完成所花费的总时间是一样的，而多线程情况下还会有线程切换上的开销，多线程最终应该比单线程更费时才对啊？要解决这个疑惑，只需要了解一下**CPU密集型**和**IO密集型**即可。  

##### CPU密集型  
CPU密集型也叫计算密集型，指的是系统的硬盘、内存性能相对CPU要好很多，此时，系统运作大部分的状况是CPU Loading 100%，CPU要读/写I/O(硬盘/内存)，I/O在很短的时间就可以完成，而CPU还有许多运算要处理，CPU Loading很高。  

在多重程序系统中，大部份时间用来做计算、逻辑判断等CPU动作的程序称之CPU bound。例如一个计算圆周率至小数点一千位以下的程序，在执行的过程当中绝大部份时间用在三角函数和开根号的计算，便是属于CPU bound的程序。  

CPU bound的程序一般而言CPU占用率相当高。这可能是因为任务本身不太需要访问I/O设备，也可能是因为程序是多线程实现因此屏蔽掉了等待I/O的时间。  

##### IO密集型  
IO密集型指的是系统的CPU性能相对硬盘、内存要好很多，此时，系统运作，大部分的状况是CPU在等I/O (硬盘/内存) 的读/写操作，此时CPU Loading并不高。

I/O bound的程序一般在达到性能极限时，CPU占用率仍然较低。这可能是因为任务本身需要大量I/O操作，而pipeline做得不是很好，没有充分利用处理器能力。  

##### CPU密集型 vs IO密集型  
我们可以把任务分为计算密集型和IO密集型。

计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。

计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

第二种任务的类型是IO密集型，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。对于IO密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是IO密集型任务，比如Web应用。

IO密集型任务执行期间，99%的时间都花在IO上，花在CPU上的时间很少，因此，用运行速度极快的C语言替换用Python这样运行速度极低的脚本语言，完全无法提升运行效率。对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言，脚本语言是首选，C语言最差。

总之，计算密集型程序适合C语言多线程，I/O密集型适合脚本语言开发的多线程。

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

> 阻塞队列和同步队列的区别：同步队列是因为线程由于锁被其他的线程所占用而陷入等待的状态；线程进入等待队列的方式只有一种方式，主动调用Object.wait()方法

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

>被暂停线程的run()方法没有执行完，导致在里面进行的修改没有及时实现；**由于suspend方法和resume方法的特殊性，可以发现，对于同步资源，suspend方法一定会造成独占，所以这一对儿只能用在非临界资源的情况，但这种情况下要注意数据不同步的问题**  

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

**经检验，当线程运行到同步代码块儿的位置是才可能被阻塞，之前的代码是异步访问的，同样，之后的代码再阻塞期间也是没法执行的**

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

> 同步锁和前面说的synchronized关键字的一个优化就是：Lock可以随意的控制加锁的位置和范围，而同步关键字只能往一个属性上、整个方法上加锁；而Lock可以实现在一个方法中的一部分中加锁。

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
  
> **经过检验：当线程真的如上所说return或者抛出异常而中断后，其中断状态会重置为false**
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

## 线程池  
> 主要参考链接：[https://www.zybuluo.com/kiraSally/note/990993](https://www.zybuluo.com/kiraSally/note/990993)  

### 1. 线程池是如何按照原理实现的  
原理图：  

![](http://static.zybuluo.com/kiraSally/9mmvv0fwbyf6exxmk68p70yz/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20171219150356.jpg)  

#### 1.1 提交任务与执行  
任务的提交是在**execute()方法**(当然还有submit方法)中完成的。首先，按照流程图，提交一个任务后需要先判断是否满足**核心池容量约束**：  

	/**
         * 情况一：当实际工作线程数 < 核心工作线程数时
         * 执行方案：会创建一个新的工作线程去执行该任务
         * 注意：此时即使有其他空闲的工作线程也还是会新增工作线程，
         *      直到达到核心工作线程数为止
         */
        if (workerCountOf(c) < corePoolSize) {
            /**
             * 新增工作线程，true表示要对比的是核心工作线程数
             * 一旦新增成功就开始执行当前任务
             * 期间也会通过自旋获取队列任务进行执行
             */
            if (addWorker(command, true))
                return;
            /**
             * 需要重新获取控制器状态，说明新增线程失败
             * 线程失败的原因可能有两种：
             *  - 1.线程池已被关闭，非RUNNING状态的线程池是不允许接收新任务的
             *  - 2.并发时，假如都通过了workerCountOf(c) < corePoolSize校验，但其他线程
             *      可能会在addWorker先创建出线程，导致workerCountOf(c) >= corePoolSize，
             *      即实际工作线程数 >= 核心工作线程数，此时需要进入情况二
             */
            c = ctl.get();
        }  

以上是execute源码的片段，很明显这第一种情况就是流程图中的**核心池未满**的情况，注意其中说明的添加任务失败的情况，情况一很明了，情况二需要提醒的是**只有其他地方的并发操作导致了核心池的容量约束被破坏了addWork才会返回false，如果只是并发提前添加了一个word但是并没有破坏核心池的容量约束的话，在addWork源码(下面会说到)内部会继续进行添加work的操作**  

对于核心池未满的情况，确实做了相应的处理，接下来看看需要入队的情况：  

	/**
         * 情况二：当实际工作线程数>=核心线程数时，新提交任务需要入队
         * 执行方案：一旦入队成功，仍需要处理线程池状态突变和工作线程死亡的情况
         */
        if (isRunning(c) && workQueue.offer(command)) {
            //双重校验
            int recheck = ctl.get();
            /**
             * recheck的目的是为了防止线程池状态的突变 - 即被关闭
             * 一旦线程池非RUNNING状态时，除了从队列中移除该任务(回滚)外
             * 还需要执行任务拒绝策略处理新提交的任务
             */
            if (!isRunning(recheck) && remove(command))
                //执行任务拒绝策略
                reject(command);
            /**
             * 若线程池还是RUNNING状态 或 队列移除失败(可能正好被一个工作线程拿到处理了)
             * 此时需要确保至少有一个工作线程还可以干活
             * 补充一句：之所有无须与核心工作线程数或最大线程数相比，而只是比较0的原因是
             *          只要保证有一个工作线程可以干活就行，它会自动去获取任务
             */
            else if (workerCountOf(recheck) == 0)
                /**
                 * 若工作线程都已死亡，需要新增一个工作线程去干活
                 * 死亡原因可能是线程超时或者异常等等复杂情况
                 *
                 * 第一个参数为null指的是传入一个空任务，
                 * 目的是创建一个新工作线程去处理队列中的剩余任务
                 * 第二个参数为false目的是提示可以扩容到最大工作线程数
                 */
                addWorker(null, false);
        }  

根据if的条件可以看出，如果**workQueue.offer(command)**通过就说明队列没有满，这时候就将**任务入队**了，**符合流程图**，然后进入了if块儿中进行**双重校验**。第一个校验很容易理解，可是当if条件没有被选择的时候(也就是注释中说的**若线程池还是RUNNING状态 或 队列移除失败(可能正好被一个工作线程拿到处理了)**)为什么还要判断一下else if的条件？  

接下来是第三个情况，**最大线程池容量约束：**  

	/**
         * 情况三：一旦线程池被关闭 或者 新任务入队失败(队列已满)
         * 执行方案：会尝试创建一个新的工作线程，并允许扩容到最大工作线程数
         * 注意：一旦创建失败，比如超过最大工作线程数，需要执行任务拒绝策略
         */
        else if (!addWorker(command, false))
            //执行任务拒绝策略
            reject(command);  

很简单的逻辑，如果添加失败就执行拒绝操作。  

上面的约束中，**核心池容量约束**和**最大线程池容量约束**的处理代码中都涉及到了**addWork**的调用，但是addWork在什么时候才能真正被添加呢(该方法返回true)？接下来就看看该方法的源码：  

	/**
	 * 新增工作线程需要遵守线程池控制状态规定和边界限制
	 *
	 * @param core core为true时允许扩容到核心工作线程数，否则为最大工作线程数
	 * @return 新增成功返回true，失败返回false
	 */
	private boolean addWorker(Runnable firstTask, boolean core) {
	    //重试标签
	    retry:
	    /***
	     * 外部自旋 -> 目的是确认是否能够新增工作线程
	     * 允许新增线程的条件有两个：
	     *   1.满足线程池状态条件 -> 条件一
	     *   2.实际工作线程满足数量边界条件 -> 条件二
	     * 不满足条件时会直接返回false，表示新增工作线程失败
	     */
	    for (;;) {
	        //读取原子控制量 - 包含workerCount(实际工作线程数)和runState(线程池状态)
	        int c = ctl.get();
	        //读取线程池状态
	        int rs = runStateOf(c);
	        /**
	         * 条件一.判断是否满足线程池状态条件
	         *  1.只有两种情况允许新增线程：
	         *    1.1 线程池状态==RUNNING
	         *    1.2 线程池状态==SHUTDOWN且firstTask为null同时队列非空
	         *
	         *  2.线程池状态>=SHUTDOWN时不允许接收新任务，具体如下：
	         *    2.1 线程池状态>SHUTDOWN，即为STOP、TIDYING、TERMINATED
	         *    2.2 线程池状态==SHUTDOWN，但firstTask非空
	         *    2.3 线程池状态==SHUTDOWN且firstTask为空，但队列为空
	         *  补充：针对1.2、2.2、2.3的情况具体请参加后面的"小问答"环节
	         */
	        if (rs >= SHUTDOWN &&
	            !(rs == SHUTDOWN && firstTask == null && ! workQueue.isEmpty()))
	            return false;
	        /***
	         * 内部自旋 -> 条件二.判断实际工作线程数是否满足数量边界条件
	         *   -数量边界条件满足会对尝试workerCount实现CAS自增，否则新增失败
	         *   -当CAS失败时会再次重新判断是否满足新增条件：
	         *       1.若此期间线程池状态突变(被关闭)，重新判断线程池状态条件和数量边界条件
	        *        2.若此期间线程池状态一致，则只需重新判断数量边界条件
	        */
	        for (;;) {
	            //读取实际工作线程数
	            int wc = workerCountOf(c);
	            /**
	             * 新增工作线程会因两种实际工作线程数超标情况而失败：
	             *  1.实际工作线程数 >= 最大容量
	             *  2.实际工作线程数 > 工作线程比较边界数(当前最大扩容数)
	             *   -若core = true，比较边界数 = 核心工作线程数
	             *   -若core = false，比较边界数 = 最大工作线程数
	             */
	            if (wc >= CAPACITY || wc >= (core ? corePoolSize : maximumPoolSize))
	                return false;
	            /**
	             * 实际工作线程计数CAS自增:
	             *   1.一旦成功直接退出整个retry循环，表明新增条件都满足
	             *   2.因并发竞争导致CAS更新失败的原因有三种: 
	             *      2.1 线程池刚好已新增一个工作线程
	             *        -> 计数增加，只需重新判断数量边界条件
	             *      2.2 刚好其他工作线程运行期发生错误或因超时被回收
	             *        -> 计数减少，只需重新判断数量边界条件
	             *      2.3 刚好线程池被关闭 
	             *        -> 计数减少，工作线程被回收，
	             *           需重新判断线程池状态条件和数量边界条件
	             */
	            if (compareAndIncrementWorkerCount(c))
	                break retry;
	            //重新读取原子控制量 -> 原因是在此期间可能线程池被关闭了
	            c = ctl.get();
	            /**
	             * 快速检测是否发生线程池状态突变
	             *  1.若状态突变，重新判断线程池状态条件和数量边界条件
	             *  2.若状态一致，则只需重新判断数量边界条件
	             */
	            if (runStateOf(c) != rs)
	                continue retry;
	        }
	    }
	    /**
	     * 这里是addWorker方法的一个分割线
	     * 前面的代码的作用是决定了线程池接受还是拒绝新增工作线程
	     * 后面的代码的作用是真正开始新增工作线程并封装成Worker接着执行后续操作
	     * PS:虽然笔者觉得这个方法其实可以拆分成两个方法的(在break retry的位置)
	     */
	    //记录新增的工作线程是否开始工作
	    boolean workerStarted = false;
	    //记录新增的worker是否成功添加到workers集合中
	    boolean workerAdded = false;
	    Worker w = null;
	    try {
	        //将新提交的任务和当前线程封装成一个Worker
	        w = new Worker(firstTask);
	        //获取新创建的实际工作线程
	        final Thread t = w.thread;
	        /**
	         * 检测是否有可执行任务的线程，即是否成功创建了新的工作线程
	         *   1.若存在，则选择执行任务
	         *   2.若不存在，则需要执行addWorkerFailed()方法
	         */
	        if (t != null) {
	            /**
	             * 新增工作线程需要加全局锁
	             * 目的是为了确保安全更新workers集合和largestPoolSize
	             */
	            final ReentrantLock mainLock = this.mainLock;
	            mainLock.lock();
	            try {
	                /**
	                 * 获得全局锁后，需再次检测当前线程池状态
	                 * 原因在于预防两种非法情况：
	                 *  1.线程工厂创建线程失败
	                 *  2.在锁被获取之前，线程池就被关闭了
	                 */
	                int rs = runStateOf(ctl.get());
	                /**
	                 * 只有两种情况是允许添加work进入works集合的
	                 * 也只有进入workers集合后才是真正的工作线程，并开始执行任务
	                 *  1.线程池状态为RUNNING(即rs<SHUTDOWN)
	                 *  2.线程池状态为SHUTDOWN且传入一个空任务
	                 *  (理由参见：小问答之快速检测线程池状态?) 
	                 */
	                if (rs < SHUTDOWN ||
	                    (rs == SHUTDOWN && firstTask == null)) {
	                    /**
	                     * 若线程处于活动状态时，说明线程已启动，需要立即抛出"线程状态非法异常"
	                     * 原因是线程是在后面才被start的，已被start的不允许再被添加到workers集合中
	                     * 换句话说该方法新增线程时，而线程是新的，本身应该是初始状态(new)
	                     * 可能出现的场景：自定义线程工厂newThread有可能会提前启动线程
	                     */
	                    if (t.isAlive())
	                        throw new IllegalThreadStateException();
	                    //由于加锁，所以可以放心的加入集合
	                    workers.add(w);
	                    int s = workers.size();
	                    //更新最大工作线程数，由于持有锁，所以无需CAS
	                    if (s > largestPoolSize)
	                        largestPoolSize = s;
	                    //确认新建的worker已被添加到workers集合中  
	                    workerAdded = true;
	                }
	            } finally {
	                //千万不要忘记主动解锁
	                mainLock.unlock();
	            }
	            /**
	             * 一旦新建工作线程被加入工作线程集合中，就意味着其可以开始干活了
	             * 有心的您肯定发现在线程start之前已经释放锁了
	             * 原因在于一旦workerAdded为true时，说明锁的目的已经达到
	             * 根据最小化锁作用域的原则，线程执行任务无须加锁，这是种优化
	             * 也希望您在使用锁时尽量保证锁的作用域最小化
	             */
	            if (workerAdded) {
	                /**
	                 * 启动线程，开始干活啦
	                 * 若您看过笔者的"并发番@Thread一文通"肯定知道start()后，
	                 * 一旦线程初始化完成便会立即调用run()方法
	                 */
	                t.start();
	                //确认该工作线程开始干活了
	                workerStarted = true;
	            }
	        }
	    } finally {
	        //若新建工作线程失败或新建工作线程后没有成功执行，需要做新增失败处理
	        if (!workerStarted)
	            addWorkerFailed(w);
	    }
	    //返回结果表明新建的工作线程是否已启动执行
	    return workerStarted;
	}  

这就来简单的捋一捋：这个方法看成两个部分，第一部分由两个循环组成，目的是判断是否满足新增一个Work的条件，如果不满足就直接return false了，也就没有第二部分什么事儿了；第二部分的作用是如果满足新增线程的条件，那就开始干吧！  

第一部分很容易理解，第二部分有一点需要注意，那就是在真正的往works中添加线程的时候(也就是正在工作线程数加一)的时候是**获取了全局锁的**。而且，这个方法返回true的时候新加入的线程已将调用了start方法了。  

### 2. 线程池是如何实现线程复用的  
使用线程池的一大目的是为了实现线程的复用来节省开销，那么线程池是如何实现复用的呢？  
首先，看看work是如何实现run方法的：  

	/**
	 * 工作线程运行
	 * runWorker方法内部会通过轮询的方式
	 * 不停地获取任务和执行任务直到线程被回收
	 */
	public void run() {
	    runWorker(this);
	}  
runWorker方法又是如何实现的呢？  

	final void runWorker(Worker w) {
	    //读取当前线程 -即调用execute()方法的线程(一般是主线程)
	    Thread wt = Thread.currentThread();
	    //读取待执行任务
	    Runnable task = w.firstTask;
	    //清空任务 -> 目的是用来接收下一个任务
	    w.firstTask = null;
	    /**
	     * 注意Worker本身也是一把不可重入的互斥锁！
	     * 由于Worker初始化时state=-1，因此此处的解锁的目的是：
	     * 将state-1变成0，因为只有state>=0时才允许中断；
	     * 同时也侧面说明在worker调用runWorker()之前是不允许被中断的，
	     * 即运行前不允许被中断
	     */
	    w.unlock();
	    //记录是否因异常/错误突然完成，默认有异常/错误发生
	    boolean completedAbruptly = true;
	    try {
	        /**
	         * 获取任务并执行任务，取任务分两种情况：
	         *   1.初始任务：Worker被初始化时赋予的第一个任务(firstTask)
	         *   2.队列任务：当firstTask任务执行好后，线程不会被回收，而是之后自动自旋从任务队列中取任务(getTask)
	         *     此时即体现了线程的复用
	         */
	        while (task != null || (task = getTask()) != null) {
	            /**
	             * Worker加锁的目的是为了在shutdown()时不要立即终止正在运行的worker，
	             * 因为需要先持有锁才能终止，而不是为了处理并发情况(注意不是全局锁)
	             * 在shutdownNow()时会立即终止worker，因为其无须持有锁就能终止
	             * 关于关闭线程池下文会再具体详述
	             */
	            w.lock();
	            /**
	             * 当线程池被关闭且主线程非中断状态时，需要重新中断它
	             * 由于调用线程一般是主线程，因此这里是主线程代指调用线程
	             */
	            if ((runStateAtLeast(ctl.get(), STOP) ||
	                 (Thread.interrupted() &&
	                    runStateAtLeast(ctl.get(), STOP))) &&
	                        !wt.isInterrupted())
	                wt.interrupt();
	            try {
	                /**
	                 * 每个任务执行前都会调用"前置方法"，
	                 * 在"前置方法"可能会抛出异常，
	                 * 结果是退出循环且completedAbruptly=true，
	                 * 从而线程死亡，任务未执行(并被丢弃)
	                 */
	                beforeExecute(wt, task);
	                Throwable thrown = null;
	                try {
	                    //执行任务
	                    task.run();
	                } catch (RuntimeException x) {
	                    thrown = x; throw x;
	                } catch (Error x) {
	                    thrown = x; throw x;
	                } catch (Throwable x) {
	                    thrown = x; throw new Error(x);
	                } finally {
	                    /**
	                     * 任务执行结束后，会调用"后置方法"
	                     * 该方法也可能抛异常从而导致线程死亡
	                     * 但值得注意的是任务已经执行完毕
	                     */
	                    afterExecute(task, thrown);
	                }
	            } finally {
	                //清空任务 help gc
	                task = null;
	                //无论成功失败任务数都要+1，由于持有锁所以无须CAS
	                w.completedTasks++;
	                //必须要主动释放锁
	                w.unlock();
	            }
	        }
	        //无异常时需要清除异常状态
	        completedAbruptly = false;
	    } finally {
	        /**
	         * 工作线程退出循环的原因有两个：
	         *  1.因意外的错误/异常退出
	         *  2.getTask()返回空 -> 原因有四种，下文会详述
	         * 工作线程退出循环后，需要执行相对应的回收处理
	         */
	        processWorkerExit(w, completedAbruptly);
	    }
	}  

结合注释也能看懂，真正实现复用的是醒目的**while循环中的task.run()**，而当**getTask()不为空**的时候while循环就没可能停下来，也就是说只要任务对列中还有任务，那么这个线程就会一直从里面取出来执行。  

同样**getTask()**也是由很多的约束的：  

	private Runnable getTask() {
	    // 记录任务队列的poll()是否超时，默认未超时
	    boolean timedOut = false; 
	    //自旋获取任务
	    for (;;) {
	        /**
	         * 线程池会依次判断五种情况，满足任意一种就返回null：
	         *    1.线程池被关闭，状态为(STOP || TIDYING || TERMINATED)
	         *    2.线程池被关闭，状态为SHUTDOWN且任务队列为空
	         *    3.实际工作线程数超过最大工作线程数
	         *    4.工作线程满足超时条件后，同时符合下述的任意一种情况：
	         *      4.1 线程池中还存在至少一个其他可用的工作线程
	         *      4.2 线程池中已没有其他可用的工作线程但任务队列为空
	         */
	        int c = ctl.get();
	        int rs = runStateOf(c);
	        /**
	         * 判断线程池状态条件，有两种情况直接返回null
	         *  1.线程池状态大于SHUTDOWN(STOP||TIDYING||TERMINATED)，说明不允许再执行任务
	         *    - 因为>=STOP以上状态时不允许接收新任务同时会中断正在执行中的任务，任务队列的任务也不执行了         
	         *  
	         *  2.线程池状态为SHUTDOWN且任务队列为空，说明已经无任务可执行
	         *    - 因为SHUTDOWN时还需要执行任务队列的剩余任务，只有当无任务才可退出
	         */
	        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
	            /**
	             * 减少一个工作线程数
	             * 值得注意的是工作线程的回收是放在processWorkerExit()中进行的
	             * decrementWorkerCount()方法是内部不断循环执行CAS的，保证最终一定会成功
	             * 补充：因线程池被关闭而计数减少可能与addWorker()的
	             *      计数CAS自增发生并发竞争
	             */
	            decrementWorkerCount();
	            return null;
	        }
	        //读取实际工作线程数
	        int wc = workerCountOf(c);
	        /**
	         * 判断是否需要处理超时：
	         *   1.allowCoreThreadTimeOut = true 表示需要回收空闲超时的核心工作线程
	         *   2.wc > corePoolSize 表示存在空闲超时的非核心工作线程需要回收
	         */
	        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;
	         /**
	          * 有三种情况会实际工作线程计数-1且直接返回null
	          *
	          *    1.实际工作线程数超过最大线程数
	          *    2.该工作线程满足空闲超时条件需要被回收：
	          *       2.1 当线程池中还存在至少一个其他可用的工作线程
	          *       2.2 线程池中已没有其他可用的工作线程但任务队列为空
	          *  
	          * 结合2.1和2.2我们可以推导出：
	          *
	          *   1.当任务队列非空时，线程池至少需要维护一个可用的工作线程，
	          *     因此此时即使该工作线程超时也不会被回收掉而是继续获取任务
	          *
	          *   2.当实际工作线程数超标或获取任务超时时，线程池会因为
	          *     一直没有新任务可执行，而逐渐减少线程直到核心线程数为止；
	          *     若设置allowCoreThreadTimeOut为true，则减少到1为止；
	          *
	          * 提示：由于wc > maximumPoolSize时必定wc > 1，因此无须比较
	          * (wc > maximumPoolSize && workQueue.isEmpty()) 这种情况
	          */
	        if ((wc > maximumPoolSize || (timed && timedOut))
	            && (wc > 1 || workQueue.isEmpty())) {
	            /**
	             * CAS失败的原因还是出现并发竞争，具体参考上文
	             * 当CAS失败后，说明实际工作线程数已经发生变化，
	             * 必须重新判断实际工作线程数和超时情况
	             * 因此需要countinue
	             */
	            if (compareAndDecrementWorkerCount(c))
	                return null;
	           /**        
	            */                
	            continue;
	        }
	        //若满足获取任务条件，根据是否需要超时获取会调用不同方法
	        try {
	           /**
	            * 从任务队列中取任务分两种：
	            *  1.timed=true 表明需要处理超时情况
	            *   -> 调用poll()，超过keepAliveTime返回null
	            *  2.timed=fasle 表明无须处理超时情况
	            *   -> 调用take()，无任务则挂起等待
	            */
	            Runnable r = timed ?
	                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
	                workQueue.take();
	            //一旦获取到任务就返回该任务并退出循环
	            if (r != null)
	                return r;
	            //当任务为空时说明poll超时
	            timedOut = true;
	            /**
	             * 关于中断异常获取简单讲一些超出本章范畴的内容
	             * take()和poll(long timeout, TimeUnit unit)都会throws InterruptedException
	             * 原因在LockSupport.park(this)不会抛出异常但会响应中断；
	             * 但ConditionObject的await()会通过reportInterruptAfterWait()响应中断
	             * 具体内容笔者会在阻塞队列相关番中进一步介绍
	             */
	        } catch (InterruptedException retry) {
	            /**
	             * 一旦该工作线程被中断，需要清除超时标记
	             * 这表明当工作线程在获取队列任务时被中断，
	             * 若您不对中断异常做任务处理，线程池就默认
	             * 您希望线程继续执行，这样就会重置之前的超时标记
	             */
	            timedOut = false;
	        }
	    }
	}