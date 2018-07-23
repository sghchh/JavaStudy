# Java 细枝末节  
>**声明：**本笔记的内容参考微信公众号"码农每日一题"，旨在记录一些Java中的细节的知识。  

## 一、基本类型及其自动装箱/拆箱
### 1.1 基本类型计算中的类型转换的问题  
看看下面代码：  

	short e=(short)1;
	e=e+1;    //编译报错
    short f=(short)1;
	f+=1;     //编译通过  

关于第一个报错我们能够理解，因为1默认是int类型，e与int类型进行加减乘除的时候自动转换为int型，这时候再赋值给short 型的e就会报错了。  
第二个通过的原因是，*+=*是Java提供的操作符，为我们进行了类型的特殊处理，所以是类型安全的。  

第二个例子：  

	System.out.println(3*0.1==0.3)    //输出false  

浮点数不能完全精确的表示出来，一般都会损失精度。

### 1.2 自动拆箱和装箱  
 
我们写的代码Integer a=1;会将1自动装箱为一个Integer类型，实际上是通过**valueOf方法**；同样的，自动拆箱的过程实际上是通过**xxxValue**方法。  

	Integer a=11;
	Integer b=11;
	System.out.println(a==b);    //输出为true
	Integer c=130;
	Integer d=130;
	System.out.println(c==d);     //输出为false

woc!!!这是为啥，我们来看看这个Integer的**valueOf源码：**

	public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }  

这里的IntegerCache.low和IntegerCache.high以及IntegerCache是什么呢？  

	private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }

可以发现，**low为-128，high为127；而IntegerCache.cache是一个元素为Integer的缓冲数组。**因此，Integer为我们内置了一个数值范围[-128,127]的缓冲数组，如果通过：  

	Integer a=12;

这种方式创建Integer对象，而且数值范围在-128到127的话，对象会被缓存到该数组中，后面再次创建的话就返回数组中的该对象。这就像String字符串常量和字符串常量池的关系。  
>注意Double和Float由于浮点数精度问题，并没有内置一个缓存数组，所以并没有该特性。  

### 1.3 自增自减运算符  
看如下代码：  

	int count=0;
	for(int i=0;i<10;i++){
		count=count++;
	}
	System.out.println(count);  

最后输出的结果是**0！！**  
在Java中a++表达式是有返回值的，其运算过程是：
>计算表达式的步骤是：  
1. 把 a 的初始值存储到临时存储 a0 中；  
2. 把 a.inc() 结果赋值给 a；  
3. 把 a0 作为表达式的结果返回。  
对于 a--，步骤是完全类似的。  

### 1.4 变长参数  

一个方法只能有一个可变长参数，且这个可变长参数必须是该方法的最后一个参数，java 不允许存在一个方法具备多个变长参数或者变长参数不是方法的最后位置的情况。  
#### 1.4.1 方法识别  

	public void print(String e,Integer...integers){
	}
	
	public void print(String a,String...strings ){
	}  
	FinalTest t=new FinalTest(5);
	t.print("hello");
	t.print("hello", null);  

实例化后调用这两个方法，如上代码将会在编译器报错：报错的原因是**无法识别方法**，因为有两个方法可以匹配。  

#### 1.4.2 方法重载  

	class Test{
	public void print(String...strings){
		
	}
	}
	public class FinalTest extends Test{

	@Override
	public void print(String[] args ){
	}
	public static void main(String[] args){
		Test test=new Test();
		test.print("hello");
		FinalTest f=new FinalTest();
		f.print("hello");
	}
	}  

注释 1 能编译通过且打印为 Sub print.，因为 base 对象把子类对象 sub 做了向上转型，形参列表是由父类决定的，当然能通过。

注释 2 编译报错为传递的参数 String 类型与方法需要的 String[] 类型不匹配，因为这时编译器看到子类覆写了父类的 print 方法，所以会使用子类重新定义的 print 方法，尽管参数列表不匹配也不会再去父类匹配（因为找到了就不再找了），故有了类型不匹配的编译错误。  

### 1.5 hashCode和equals方法  
[https://mp.weixin.qq.com/s/ZZIY3hKmT-ydTti0yOrKrA](https://mp.weixin.qq.com/s/ZZIY3hKmT-ydTti0yOrKrA)  
[https://blog.csdn.net/qq_21688757/article/details/53067814](https://blog.csdn.net/qq_21688757/article/details/53067814)  

