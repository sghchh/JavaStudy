#JavaIO  
声明：本文章主要参考  
>https://juejin.im/entry/59291f1a44d9040064213c02

先说一说电脑上的数据的储存方式，一种是外存（磁盘，硬盘，U盘等），一种是内存（内存条），还有一种是缓存（位于cpu中）。  
##1.数据流  
一组有序，有起点终点的字节的数据序列就是数据流。包括了输入流和输出流。  
![](https://user-gold-cdn.xitu.io/2017/5/27/7b7033d593400a8091d9bfdc0b298085?imageView2/0/w/1280/h/960)  
从图中可见，数据流的终点是程序的内存区的话就是输入流，起点是程序的内存区的话就是输出流。  
###数据流的分类  
流序列中的数据既可以是未经加工的原始的二进制数据，也可以是经过了一定编码处理后符合某种格式规定的特定数据。因此Java中的流分为两种：  

* 字节流：数据的最小单元是字节（**万能，任何数据都能读写**）  
* 字符流：数据的最小单元是字符，Java中的字符是Unicode编码的，一个字符占两个字节（**只能够读写字符**）  
为什么会有这种分类呢，直接全部都用字节流不就行了？首先要知道，从磁盘文件，网络等中读取数据是一个不小的消耗，所以能够尽量的减少读取的次数是再好不过了。而字符流的读取单位比字节流要大，虽然只能够读取文本内容，但好在文本内容平时不少和我们打交道，所以Java设计出字符流只赚不亏。  
##2.Java中基本的流  
###2.1 四个基本的类  
* 字节流：InputStream，OutputStream  
* 字符流：Reader，Writer  
这四个基本的类是抽象类，不能够实例化，但是可以使用他们的派生类：  
![](https://user-gold-cdn.xitu.io/2017/5/27/1f764a6578ce587dd4b375a0f49bbd27?imageView2/0/w/1280/h/960)  
![](https://user-gold-cdn.xitu.io/2017/5/27/7a9562993319dad9e678215999a79337?imageView2/0/w/1280/h/960)  
![](https://user-gold-cdn.xitu.io/2017/5/27/844ff03073d5f5f77d6bbd79907bfbb6?imageView2/0/w/1280/h/960)  
![](https://user-gold-cdn.xitu.io/2017/5/27/993ecfaa94913626dde52f7328fba43a?imageView2/0/w/1280/h/960)  
###2.2 具体分类  
* **Memory**：  
	* 从/向内存数组读写数据: CharArrayReader，CharArrayWriter，ByteArrayInputStream、ByteArrayOutputStream  
	* 从/向内存字符串读写数据 StringReader、StringWriter、StringBufferInputStream  
* **Pipe管道**  实现管道的输入和输出（ 进程间通信）: PipedReader、PipedWriter、PipedInputStream、PipedOutputStream
* **File 文件流** 对文件进行读、写操作 ：FileReader、FileWriter、FileInputStream、FileOutputStream 
* ObjectSerialization 对象输入、输出 ：ObjectInputStream、ObjectOutputStream
* **DataConversion数据流** 按基本数据类型读、写（处理的数据是Java的基本类型（如布尔型，字节，整数和浮点数））：DataInputStream、DataOutputStream 
* **Printing** 包含方便的打印方法 ：PrintWriter、PrintStream
* **Buffering缓冲**  在读入或写出时，对数据进行缓存，以减少I/O的次数：BufferedReader、BufferedWriter、BufferedInputStream、BufferedOutputStream 
* **Filtering 滤流**，在数据进行读或写时进行过滤：FilterReader、FilterWriter、FilterInputStream、FilterOutputStream过
* **Concatenation合并输入** 把多个输入流连接成一个输入流 ：SequenceInputStream 
* **Counting计数**  在读入数据时对行记数 ：LineNumberReader、LineNumberInputStream
* **Peeking Ahead** 通过缓存机制，进行预读 ：PushbackReader、PushbackInputStream
* **Converting between Bytes and Characters** 按照一定的编码/解码标准将字节流转换为字符流，或进行反向转换（Stream到Reader,Writer的转换类）：InputStreamReader、OutputStreamWriter 
###2.3  根据数据来源和去向分类  
* **File文件**： FileInputStream, FileOutputStream, FileReader, FileWriter 
* **byte[]:**ByteArrayInputStream, ByteArrayOutputStream   
* **Char[]:** CharArrayReader, CharArrayWriter
* **String:**StringBufferInputStream, StringReader, StringWriter   
* **网络数据流:**InputStream, OutputStream, Reader, Writer  
###2.4 标准流  
Java自带的标准数据流：java.lang.System  

		java.lang.System   
		public  final  class System  extends  Object{    		
			static   PrintStream  err;//标准错误流（输出）   
			static   InputStream  in;//标准输入(键盘输入流)   
			static   PrintStream  out;//标准输出流(显示器输出流)   
		}  
这下知道System.out.print()的原理就是调用了PrintStream.print()方法  
###2.5 数据流提供方法  
这里只说InputStream，其他的三个流方法都和这个大同小异，只是参数由byte类的换成了char类的，read改成write等变化。  
InputStream是输入字节数据用的类，提供了三种重载的read方法：  

*   public abstract int read( )：读取一个byte的数据，返回值是高位补0的int类型值。**若返回值=-1**说明没有读取到任何字节读取工作结束。  
*   public int read(byte b[ ])：读取b.length个字节的数据放到b数组中。返回值是读取的字节数。该方法实际上是调用下一个方法实现的  
*   public int read(byte b[ ], int off, int len)：从输入流中最多读取len个字节的数据，存放到偏移量为off的b数组中。   
*   public int available( )：返回输入流中可以读取的字节数。注意：若输入阻塞，当前线程将被挂起，如果InputStream对象调用这个方法的话，它只会返回0，这个方法必须由继承InputStream类的子类对象调用才有用，  
*   public long skip(long n)：忽略输入流中的n个字节，返回值是实际忽略的字节数, 跳过一些字节来读取  
*   public int close( ) ：我们在使用完后，必须对我们打开的流进行关闭.  
最常见的就是三个read方法和close方法了。  
*这里就简单说一下缓冲的事，read方法无参的时候就代表一个一个的读，当传入一个byte[]数组的时候，就是先将数据读入到数组中，在一次性的将数据从数组中读入内存，这，就是一种缓冲，这种方法一次性的读取多个数据，显然比无参的时候一个一个的读取的开销要小。*   
###2.6 缓冲流  
**计算机访问外部设备非常耗时。访问外存的频率越高，造成CPU闲置的概率就越大。为了减少访问外存的次数，应该在一次对外设的访问中，读写更多的数据。为此，除了程序和流节点间交换数据必需的读写机制外，还应该增加缓冲机制。缓冲流就是每一个数据流分配一个缓冲区，一个缓冲区就是一个临时存储数据的内存。这样可以减少访问硬盘的次数,提高传输效率。缓冲流的缓冲机制就是通过一个数组实现的**  
BufferedInputStream:当向缓冲流写入数据时候，数据先写到缓冲区，待缓冲区写满后，系统一次性将数据发送给输出设备。  
BufferedOutputStream :当从向缓冲流读取数据时候，系统先从缓冲区读出数据，待缓冲区为空时，系统再从输入设备读取数据到缓冲区.  
**增加缓冲功能后需要使用flush（）方法（强制清空缓冲区）才能将缓冲区的内容写到实际的物理节点（BufferedWriter会自动调用这个方法，不用程序员显式调用）。**

#JavaNIO  
上面所说的Java输入输出流等都是阻塞式的输入输出，也就是说**当这些流执行read方法读取数据的时候，如果数据源中没有数据，则就会阻塞线程。**而且传统的输入输出流都是通过字节移动来处理的（即使是字符流，在底层也是通过字节的移动来实现的），也就是说，**面向流的输入输出系统一次只能处理一个字节，**因此面向流的输入输出系统通常效率不高。  
Java中的新IO则以不同的方式来处理输入输出，NIO采用内存映射文件的方式来处理输入输出，**NIO将文件或文件的一段区域映射到内存中，这样就可以像访问内存一样来访问文件了（这种处理方式模拟了操作系统上的虚拟内存的概念）**，通过这种方式来进行输入输出比传统的方式要快的多。  
##1.核心概念  
###1.1 Channel（通道）  
NIO中的所有数据都要通过通道传输，Channel的map（）方法则可以将“一块数据”直接映射到内存中。如果说传统的输入输出是面向流的处理，那么NIO就是面向块的处理。  
###1.2 Buffer（缓冲）  
顾名思义，作为一个缓冲，Buffer的作用就是一个容器，Buffer的本质是一个数组，就像传统的缓冲实现一样，所有数据要想写进Channel中，先得存进Buffer中，然后由Buffer一块全部写进Channel。**Buffer也允许Channel直接将一块数据直接映射成Buffer**  
###1.3 Charset（字符集）  
Channel的作用就像是传统输入输出流中的各种Stream和Writer，Reader；顾名思义，字符集适用于char，String等。因为只有文本数据才有可能用到字符集（将二进制解码成字符，或者将字符编码成二进制），其他形式的数据（声音，图像等）都用不到字符集。  
##2.Buffer  
Buffer是一个抽象类，它的最常用的实现类是ByteBuffer，除了布尔类型外，其他的**基本类型**都有相应的Buffer实现XxxBuffer**（其中ByteBuffer还有一个子类MappedByteBuffer，会在Channel中见到）**。  
各XxxBuffer都有一个静态方法用来创建并返回一个XxxBuffer的实例  

	static XxxBuffer allocate（int capacity）  
该方法创建一个容量为capacity的XxxBuffer对象。  
###2.1 核心参数  
* 容量（capacity）：该值代表了Buffer缓存的最大数据容量，创建后不可改变  
* 界限（limit）：顾名思义，表示数据读取的上界，也就是说该位置后的数据不能够读写  
* 位置（position）：用于表名当前在Buffer中读写的位置。Buffer中存放的数据position是从0开始的，也就是说读到第二个数据的时候，position为2，表示改读第三个数据了。  
###2.2 Buffer的三种状态  
* 刚开始的时候Buffer的position在起点，也就是position为0；而limit为capacity，也就是说默认Buffer容量全部可读写  
* 当通过Buffer的put（）方法往Buffer中写入一个数据的时候，position会往后移动一格，当所有的数据装入Buffer完成后，调用Buffer的flip（）方法，则limit移动到当前的position位置，也就是说limit准确标出了Buffer中真实的数据量的大小，防止进行超出这个范围的读取。调用完flip（）方法之后，Buffer就做好了输出数据的准备。**因此flip（）方法一般用于往Buffer中写入数据的时候**  
* 当Buffer输出数据结束后，Buffer调用clear（）方法，这个方法的意思是将position移动到起始的0位置，limit移动到capacity位置，也就是回到了初始的状态，又做好了输入数据的准备了。  
###2.3 Buffer的读写操作  
Buffer的子类都实现了put（）和get（）方法来进行读写操作。put方法表示将数据放入Buffer中，get方法表示取出。同时也支持批量取和写（以数组做为参数）。  
###2.4 特殊  
ByteBuffer还提供了一个allocateDirect（）方法来直接创建一个Buffer，该方法创建的成本高，但是效率也高，因此适用于ByteBuffer长期使用的情况。  
##3.Channel  
Channel的作用就想传统输入输出流中的各种Stream，Reader和Writer。但是**Channel可以通过map方法将一块数据直接映射到内存，或者Buffer（创建一个缓存，那么就会将程序内存中的一块区域划出来作为缓存）。因此将数据写入内存就先当于Channel将数据写入Buffer；这就类比于输入输出流想数据写入内存，只不过输入输出流写入的时候“内存”是没有具体的对象来显示的，而NIO中“内存”是Buffer来担当的。**  
**由于Channel是针对于传统的输入输出流来创造的，因此，Channel也有各种Channel（FileChannel等）。Channel对象的创建是通过各种流的getChannel（）方法来实现的。也就是说，是什么流的getChannel（）方法，产生的就是什么Channel。**  
###3.1 方法  
* read（Buffer）：**这个方法的意思是将Channel中的数据读到Buffer中** 
* write（Buffer）：**这个方法的意思是将Buffer中的数据写到Channel中**  
* MappedByteBuffer map(FileChannel.MapMode mode,long position,long size):（这里是以FileChannel作为示例的）实现将数据映射为Buffer  
	* 第一个参数代表执行映射的模式，分别有只读，只写，读写等  
	* 第二个第三个参数决定映射的数据是哪一块  
##4. Charset  
###4.1 Encode和Decode  
Encode代表编码，是指将字符序列编码成二进制序列  
Decode代表解码，是指将二进制序列解码成字符序列  
可以调用System类的getProperties（）方法来看本地系统的文件编码格式，看到有file.encoding=GBK表示是以GBK字符集进行的编码。  
知道了编码的字符集的名称后就可以产生一个Charset对象了：  

	Charset charset=Charset.forName("GBK");  
获得了Charset对象之后就可以调用该对象的newDecode(),newEncode()方法来获得CharsetDecoder和CharsetEncoder对象了，这两个对象分别是解码器和编码器，可以分别调用这两个对象的decode(CharBuffer/String)将CharBuffer或者String转化成字节序列ByteBuffer和encode(ByteBuffer)方法将一个字节序列转化成CharBuffer字符序列。  
另外，Charset类就提供了三个相应的方法来实现解码器和编码器的方法的效果：  
* CharBuffer decode(ByteBuffer)
* ByteBuffer encode(CharBuffer)
* ByteBuffer encode(String)
 
传统的IO和NIO之间的效率差别：  

	public class Compare {

	//通过IO进行一个.mp3文件的拷贝，返回的是拷贝的时间
	public long getIOCopyTime(File file) 
	{
		FileInputStream input=null;
		try {
			input = new FileInputStream(file);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		BufferedInputStream bufferInput=new BufferedInputStream(input);
		File f=new File("D://IOmusic.mp3");
		FileOutputStream out=null;
		try {
			out = new FileOutputStream(f);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		BufferedOutputStream output=new BufferedOutputStream(out);
		byte[] bytes=new byte[1024];
		long curr=System.currentTimeMillis();
		try {
			while(bufferInput.read(bytes, 0, bytes.length)!=-1)
			{
				output.write(bytes, 0, bytes.length);
				output.flush();
				bytes=new byte[1024];
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return System.currentTimeMillis()-curr;
	}
	
	//NIO方式的拷贝，返回的也是拷贝的时间
	public long getNIOCopyTime(File file)
	{
		FileInputStream in=null;
		try {
			in = new FileInputStream(file);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		FileChannel channel=in.getChannel();
		File f=new File("D://NIOmusic.mp3");
		FileOutputStream out=null;
		try {
			out = new FileOutputStream(f);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		FileChannel outChan=out.getChannel();
		long curr=System.currentTimeMillis();
		try {
			outChan.write(channel.map(FileChannel.MapMode.READ_ONLY, 0, file.length()));
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return System.currentTimeMillis()-curr;
	}
	
	public static void main(String[] args) throws IOException
	{
		Compare c=new Compare();
		System.out.println(c.getIOCopyTime(new File("C://Users/as/Downloads/Burning.mp3")));
		System.out.print(c.getNIOCopyTime(new File("C://Users/as/Downloads/Burning.mp3")));
	}
	}
	输出：
	------------
	47
	6
可以看出，NIO确实更加快