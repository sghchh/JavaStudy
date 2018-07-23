#JVM之类加载机制  
参考文章：  
>https://mp.weixin.qq.com/s/rLooaTOU_NQTJdn28KAUFw
##1. 什么是类的加载  
类的加载是将类的.class文件中的二进制数据读取到内存中，将其放在**运行时数据区的方法区**内（关于运行时数据区，方法区等的只是将在jvm的内存模型中讲到）。**类加载的最终产品是位于堆栈内的Class对象，Class对象封装了类在方法区的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。**（感悟：这就从一方面说明了为什么Java内存模型中jvm堆的空间最大，因为，使用一个类时可能需要创建很多对象，而这些对象所封装的数据都指向了方法区中的那个）。  

--
*思考题：定义一个工具类，所有的方法和数据都是静态的，而且构造器用private修饰，那么我通过"类名.方法名"或者"类名.变量名"访问数据，是不是也是通过这个java.lang.Class对象？*
![](https://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnokxXiapDdvntH8PGwa0zGXMqaq9p1LDCF7iadMBibd685uYCy8u0yhb5dmlvLRwgzNueNdSdWdvEwww/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)  
  
####加载.class的方式  

* 从本地系统中加载  
* 通过网络下载.class文件  
* 从zip，jar等归档文件中加载.class文件  
* 从专有数据库中提取.class文件  
* 将Java源文件编译为.class文件  
##2. 类的生命周期  
![](https://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnokxXiapDdvntH8PGwa0zGXMyIBnM38m8eKia8wAVY8aXb0NhM9wFNDLVuoFKIZ0Q2SBk5yibFgXsXOw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&retryload=1)  
###2.1 加载阶段  
查找并加载类的二进制数据是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以一下三件事情：  

* 通过一个类的权限定名来获取其定义的二进制字节流  
* 将这个字节流所代表的静态储存结构转化为**方法区**的运行时数据结构  
* 在Java堆中生成一个代表这个**类的java.lang.Class对象**（注意：这个java.lang.class对象不同于new关键字创建出来的对象，涉及到反射的知识），作为**对方法区中这些数据区的访问入口**  
加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区内，而且在Java堆中也创建一个Class类的对象，这样就可以通过该对象访问方法区中的这些数据。  
###2.2 链接  
####2.2.1 验证：确保被加载的类的正确性  
验证是链接的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作：  

* **文件格式的验证**：验证字节流是否符合Class文件格式的规范；例如：是否以 0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。  
* **元数据的验证**：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了 java.lang.Object之外。 
* **字节码的验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。  
* **符号引用的验证**：确保解析动作能正确执行  
验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用 -Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。  

--  
*上面提到了常量池，了解一下常量池是属于内存里的哪一块儿区域？*
####2.2.2 准备：为类的静态变量分配内存，并为其初始化为默认值  
准备阶段是正式为**类变量**分配内存并设置类变量的**初始值**（*注：这里是初始值，不是直接就默认值*）；通常情况下初始值就是默认值（如0，null，false等），但是，如果在声明一个**静态常量**，那么初始值就是指定的那个值。  
理解：  
* 静态量和普通量的区别：准备阶段是为静态量分配内存，以及初始化的阶段。并不涉及到普通变量的事情。比如：public static int a;public int b;经过了这个阶段后a有了内存空间，并且a被赋予了0值；但是，b却啥都没有变化。  
* 静态常量和讲台变量的区别：准备阶段在为静态量初始化的时候，如果是一个静态常量，就不会初始化为默认值，而是初始化为指定的值。public static final int a=3;public static int b=3;在经过了准备阶段后，a的值变为了3，但是b的值却是0，将b赋值为3的阶段是类加载的初始化阶段。（*感悟：这就理解了为什么在声明static final修饰的量时要显示的为其赋值，因为如果不这样的话，他就被赋值为了默认值，也就没了意义*）。  
#JVM之内存结构  
先看一张图：  
![](https://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnoUSbbnzEiafyyQWUibOfnE3GicpdRQOuxWBrhB3Fic7MRf4z5ywT2RmCicibGibHNQEgUbsibLR1eLVRfo3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)   
JVM内存结构主要有三大块：**堆内存、方法区和栈。**堆内存是JVM中最大的一块由**年轻代**和**老年代**组成，而年轻代内存又被分成三部分，**Eden空间、From Survivor空间、To Survivor空间**,默认情况下年轻代按照**8:1:1**的比例来分配；

方法区**存储类信息、常量、静态变量等数据**（这里就和上面的JVM类加载相互对应上了），是线程共享的区域，为与Java堆区分，方法区还有一个别名Non-Heap(非堆)；栈又分为java虚拟机栈和本地方法栈主要用于方法的执行。 
再来看一看JVM和系统之间的关系：  
![](https://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnoUSbbnzEiafyyQWUibOfnE3G1sZG1aJZSakhFe5d6QeiciaO9ZIDfHrFS9UZx8RfWfkPk9UZLCVdcriaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)   
##Java堆（Heap）  
根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，**只要逻辑上是连续的即可**，就像我们的磁盘空间一样。Java堆内存的唯一目的就是存放对象实例，**几乎所有对象实例都在这里分配内存。**由于Java堆是垃圾收集器管理的主要区域，所以很多时候也被称为**“GC堆”**  
##方法区（Method Area）  
方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，它用于**存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。  
Java虚拟机规范对这个区域的限制非常宽松，除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的。**这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载**，一般来说这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是有必要的。方法区有时被称为**持久代（PermGen）** 

--  
*思考：这里提到了常量池，常量池和方法区的关系？*  
![](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnoL6lB6hGicsicgcT50EBbI0CicvmOXXLKcdAmJVia32ITn6gWezTRCuficFIgibcgDiaBUibjicaQyO7blCyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)  
方法的执行都是伴随着线程的。**原始类型的本地变量以及引用都存放在线程栈中。而引用关联的对象比如String，都存在在堆中。**为了更好的理解上面这段话，我们可以看一个例子：  

	public class HelloWorld{  
		private static Logger LOGGER=Logger.getLogger(HellowWorld.class.getName());  
		public void sayHello(String message){  
			SimpleDateFormat formatter=new SimpleDateFormat("dd.MM.YYYY");  
			String today=formatter.format(new Date());  
			LOGGER.info(today+":"+message);
		}
	}  
这段程序的数据在内存中的存放如下：  
![](http://mmbiz.qpic.cn/mmbiz_png/PgqYrEEtEnoL6lB6hGicsicgcT50EBbI0CMiazSMwukWpGox7ns74yo5Ke8iaPpD6bXgWnT2T87RFyEoDXAia06P34g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)