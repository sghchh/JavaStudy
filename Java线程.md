#Java线程  
###一 认识线程  

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
	* 该方法执行后**不会释放锁** 
* suspend方法：  
	* 该方法是一个**实例方法**，没有参数，调用后线程进入**暂停**状态  
	* 解除**暂停**状态需要**调用resume（）方法**显示解除  
	* 该方法执行后**不会释放锁**
* join方法：  
	* 该方法是一个**实例方法**。比如在main线程中调用方法，“新线程.join()"。则main线程会停止，直到新线程的run方法执行完。  
	* 同时也可以添加long参数，表示最多等待的时间（毫秒）。  
	* 该方法**底层使用wait实现的**，所以该方法执行后**会释放锁**
###三 线程同步  
由于多线程的并发操作，线程的同步就会出现问题，解决方法就是引入了**同步监视器和同步锁**：  
####3.1 synchronized关键字  
#####3.1.1 同步代码块    
		synchronized（obj）{
			...  
			//此处的代码块就是同步代码块 
		}  
synchronized后面括号中的obj就是**同步监视器**，上面代码的含义是：线程开始执行同步代码块之前，必须先获得**对同步监视器的锁定**。在多线程并发操作临界区（共享资源）的时候，任意时刻只有一个线程可以获得对同步监视器的锁定，**当同步代码块执行完毕后释放锁定**。  
#####3.1.2 同步代码块  
就是用synchronized关键字修饰**实例方法**，这种情况下**同步方法的同步监视器是this，也就是调用方法的对象**。  
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
* 调用该方法的线程**会是释放对同步监视器的锁定**  
* 在从该方法返回前，线程与其他线程竞争重新获得锁  
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
###五 线程终止  
线程终止都会释放同步监视器。  
####5.1 Java终止线程的方法  
* 使用退出标识，使线程正常退出，即当run方法执行完成后线程自动终止  
* 使用stop方法强行终止，已被弃用  
* 使用interrupt（）方法中断线程  
####5.2 interrupt方法  
表示中断一个线程  

* 只能被自己中断，否则有可能抛出异常  
* 若当前线程已经被Object.wait()，Thread的join(),sleep方法阻塞，则调用该方法会抛出异常，**且中断状态会被清除**  
####5.3 isInterrupted方法  
判断是否处于中断状态  

* 调用的线程不清除状态标识  
isInterrupted(boolean):带参数的方法，参数表示是否清除状态标识，false标识不清除，true标识清除  
####5.4 interrupted方法    
返回当前正在执行的线程是否处于中断状态，**底层是currentThread().isInterrupted(false);**所以这个判断的是当前正在执行的线程的状态，也就是 Thread1.interrupted（）方法调用后可能返回的是Thread2是否处于中断状态的布尔值。  
####5.5 终止线程的方法  
#####5.5.1 异常法  
线程在抛出异常后会主动终止，所以该方法就是手动抛出异常  
		//直接选择在run方法中抛出异常，从而停止线程
		Thread t2 = new Thread(new Runnable() {
    		@Override
    		public void run() {
       		 try {
           		 throw new InterruptedException();//主动抛出异常
        	} catch (InterruptedException e) {
          	  System.out.println("成功捕获 InterruptedException异常");
           	 e.printStackTrace();
       		 }
   		 	}
		},"kira");
		t2.start();
		t2.interrupt();
		-------------
		//输出：成功捕获 InterruptedException异常
		//打印：java.lang.InterruptedException  
#####5.5.2 沉睡法  
利用interrupt方法中的*若当前线程已经被Object.wait()，Thread的join(),sleep方法阻塞，则调用该方法会抛出异常*，在沉睡中调用interrupt方法，让其抛出异常 
 
		//沉睡
		Thread t2 = new Thread(new Runnable() {
    		@Override
    		public void run() {
        		try {
         			   Thread.sleep(1000);//沉睡
       			 } catch (InterruptedException e) {
           			 System.out.println("在沉睡中 成功捕获 InterruptedException异常");
            		e.printStackTrace();
       			 }
   			 }
			},"kira");
		t2.start();
		t2.interrupt();
		-------------
		//输出：在沉睡中 成功捕获 InterruptedException异常
		//打印：java.lang.InterruptedException: sleep interrupted  
#####5.5.3 Return法  
		//j将interrupt方法与return结合使用也能停止线程
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
    			int i = 0;
    			while (true){
        			if (Thread.currentThread().isInterrupted()){
            			System.out.println("停止");
            			return;
        			}
        		System.out.println(System.currentTimeMillis());
    		}
		}
		},"kira");
		t2.start();
		Thread.sleep(1000);
		t2.interrupt();
		-------------
		//输出：
		.....
		1502463392035
		1502463392035
		1502463392035
		停止
		//然后就停止打印了