#Java基础类库  
##系统相关类  
###System类  

* 定义：System类代表当前Java程序的运行平台，程序不能创建System类的对象，但System类提供了一些类变量和类方法，允许直接通过System类来调用这些类变量和类方法。（System类提供了代表了**标准输入，标准输出，错误输出的类变量**（System.in,System.out,System.err）,并提供了一些静态方法用于**访问环境变量，系统属性**（getenv(),getProperties()等方法），还提供了加载文件和动态链接库的方法）  
* 作用  

      * 获取平台相关属性
      * 调用平台命令来完成特定的功能
此外，System类提供了一个identityHashCode（Object x）的方法，返回指定对象的精确的hashCode值，是根据对象的地址计算得到的，如果返回值一样，那么一定是同一个对象。  

			public class DefaultText {
				public static void main(String[] args)
				{
					System.out.println(System.getenv("Path"));
					System.out.println(System.getenv("JAVA_HOME"));
				}
			}
            输出是：D:/JAVA-jdk/bin/../jre/bin/server;D:/JAVA-jdk/bin/../jre/bin;D:/JAVA-jdk/bin/../jre/lib/amd64;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files (x86)\NVIDIA Corporation\PhysX\Common;D:\Git\Git\cmd;C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;D:\JAVA-jdk\bin;C:\Users\as\AppData\Local\Microsoft\WindowsApps;;D:\Eclipse\eclipse;
			D:\JAVA-jdk
这只是最简单的用法，只是做个示范。  
###Runtime类  

* 定义：Runtime类代表了Java程序的运行时环境，每个Java程序都有一个与之对应的Runtime实例，程序可通过该对象与其运行时环境相连，同System类一样，程序也不可创建自己的Runtime实例，但可以通过getRuntime（）方法获取那个与之关联的Runtime实例。  
* 作用：访问JVM相关信息，如处理器数量，内存信息等  

		public class DefaultText {
			public static void main(String[] args)
			{
				Runtime run=Runtime.getRuntime();
				System.out.println("处理器数量"+run.availableProcessors());
				System.out.println("总内存数"+run.totalMemory());
			}
		}
        输出是：
		处理器数量4
		总内存数126877696
System和Runtime类都提供了gc（）和runFinalization（）方法来通知系统进行垃圾回收，清理系统资源。  
##ThreadLocalRandom和Random伪随机数  
Random是一个专门用来生成一个伪随机数的类，有两个构造器：  

* new Random():使用当前时间作为种子
* new Random(Long seed):自定义一个种子  
种子的作用是在nextInt（）方法中作为一个初始值，按照一个算法，生成一个随机数（0-2的32次幂）  
下面介绍几个生成随机数的方法  

* nextInt():无参方法，生成一个随机数（0-2的32次幂）  
* nextInt(int bound):生成一个0-bound范围之内的随机数  
* nextFloat():返回一个0.0-1.0之间的随机数  
* nextDouble():同上一个方法  
* nextLong():同nextInt()只是把结果转换成了Long型  
这里只是列举了一部分，其他的自行查看API  
ThreadLocalRandom类用法和Random差不多，只是获取ThreadLocalRandom对象是通过静态方法current（）获取的：ThreadLocalRandom ran=ThreadLocalRandom.current();  
##正则表达式  
作用：正则表达式是一种强大的字符串处理工具，可以对字符串进行查找，提取，分割，替换等操作。  
###单个字符的表达式  

* x：字符x（x是任何一个合法字符）  
* \0hhh:8进制数0hhh所表示的字符  
* \xhh:16进制数0xhh所表示的字符  
* \uhhhh:16进制0xhhhh所表示的Unicode字符  
* \t:制表符  
* \n:换行符  
* \r:回车符  
* \f:换页符  
* \a:报警符  
* \e:Escape符  
###通配符  

* .:可以匹配任何字符  
* \d:可以匹配0-9任何数字(d是digit的意思)  
* \D:匹配非数字  
* \w:可以匹配所有单词字符，包括0-9所有的数字，a-z所有字母，还有下划线_(w是word的意思)  
* \W:可以匹配所有非单词字符  
* \s:可以匹配所有空白字符，包括空格符，制表符，换行符，回车符，换页符(s是space的意思)  
* \S:可以匹配所有非空白字符
###有特殊意义的字符  

* $:匹配一行的结尾 
* ^:匹配一行的开头  
* ():标记字表达式的开始和结束的位置  
* []:用于确定**中括号表达式**的开始和结束位置  
* {}:用于标记前子表达式的出现频度  
* *:指定前子表达式可以出现零次或多次  
* +:指定前子表达式可以出现一次或多次   
* ?:指定前子表达式可以出现零次或一次  
* .:可以匹配除了换行符之外的所有单字符  
* \:转义字符  
* |:分隔符  
以上这些有特殊意义的字符，如果想匹配字符本身，则需要使用转义字符。如：\$表示匹配$字符  
#####中括号表达式  

* 表示枚举：[abc]表示a,b,c中任意一个字符  
* 表示范围-:[a-f]表示a到f的任意字符  
* 表示否定^:[^a]表示除了a的任意字符  
* 表示“与”运算&&:[a-f&&[^b]]表示a到f范围中除了b的所有字符  
* 表示“并”运算：[a-c[m-p]]表示a到c和m到p所有字符  
**注：中括号表达式的结果是单个字符**  
#####圆括号表达式  
用于将多个表达式组成一个子表达式，与中括号表达式区别就是可以表示一串字符，而不仅仅是单个字符。如：（dog）表示匹配dog这个字符串  
#####正则表达式的三种模式：  

* 贪婪模式（Greedy）：默认的模式，该模式表示表达式会一直匹配下去直到无法匹配  
* 勉强模式（Reluctant）：用问号（？）作为字表达式的后缀，他只会匹配最少的字符  
* 占有模式（Possessive）：很少用  
比如：x(贪婪模式)，x?（勉强模式）  
###正则表达式的使用  
Java中通过Pattern和Matcher类来使用正则表达式。  
Pattern对象时正则表达式编译后在内存中的表示形式，因此正则表达式必须先被编译成Pattern对象，然后再利用Pattern对象创建Matcher对象，各种操作方法都是Matcher对象的方法，多个Matcher对象可以共用一个Pattern对象。Matcher对象提供的几个方法是：  

* find():返回目标字符串中是否包含于Pattern匹配的子串  
* int start():返回上一次与Pattern对象匹配的子串在目标字符串中的开始位置  
* int end():返回上一次与Pattern对象匹配的子串在目标字符串中的结束位置加1  
* String group():返回上一次与Pattern匹配的子串。  
* Boolean lookingAt():返回目标字符串前面部分与Pattern是否匹配  
* Boolean matches():返回整个目标字符串与Pattern是否匹配  
* Reset():将现有的matcher对象应用与一个新的字符序列  

		public class DefaultText {
			public static void main(String[] args)
			{
				String str="优质的翻译服务开放接口 高质量翻译,28种语言支持 便捷接入,稳定服务 友情链接 百度翻译 百度开发者中心 百度翻译APP 联系我们 邮箱:translate_api@baidu.com ";
				Pattern p=Pattern.compile("([\\w&&[^\\d]]*@[\\w&&[^\\d]]*.[\\w&&[^\\d]]*)");
				Matcher m=p.matcher(str);
				while(m.find())
					System.out.print(m.group());
			}
		}
         输出结果是字符串中的那串网址。  
**注意是如何获得Pattern和Matcher对象的。**  


