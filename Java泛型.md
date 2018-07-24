#Java泛型  
方法中的形参代表变量，常量，表达式等数据，把他们称为形参，或者是数据形参。定义方法时可以声明数据形参，再调用方法时必须为这些数据形参传入实际的数据。所谓泛型就是允许在定义接口，类，方法的时候使用**类型形参**，这个类型形参在声明变量，创建对象，调用方法的时候必须指明。也就是说，在定义的时候使用“模糊”的类型，在使用的时候就必须制定确切的类型了。  
看示例：  

	//定义的时候可以使用泛型
	public class People<T>{
		public T t;
	//虽然定义类的时候用到了泛型，但是定义构造器的时候不能用<>菱形语法
		public People(T t){
			this.t=t;
		}
	}
	//这是使用的时候，使用的时候就必须为类型形参指定确切的值
	class Person extends People<String>{
		public Person(String s){
			super(s);
		}
	}  
是指泛型参数的上限是：<T extends Object>
通配符的上限也是：<? extends Object>  
下限分别是：<T super String>,<? super String>  

**泛型使用的核心是：定义的时候可以使用泛型，使用的时候必须确定这个泛型的确切类型**  
##定义的时候  
在定义一个接口或者类的时候，如果这个接口或者类的成员中（包括静态和非静态）要用到泛型，那么这个类或者接口的后面必须用“菱形语法”  

	//因为People的成员中有一个T类型，使用到了泛型，所以People后面必须用菱形语法
	public class People<T>{
		public T t;
	//虽然定义类的时候用到了泛型，但是定义构造器的时候不能用<>菱形语法
		public People(T t){
			this.t=t;
		}
	}
	//这是使用的时候，使用的时候就必须为类型形参指定确切的值
	class Person extends People<String>{
		public Person(String s){
			super(s);
		}
	}   
定义接口和定义类一样。但是在定义构造器的时候**一定不能使用菱形语法**，可以理解为构造器内部在初始化成员的时候可能不涉及到使用到了泛型的成员；但是在调用构造器的时候**可以使用菱形语法**，这里也仅仅是*可以*，说明也是有可能初始化的时候没有初始化那个泛型的成员，所以就没必要用菱形语法了。  
同样也可以定义一个泛型方法，当一个类或者接口没有使用类型参数，但定义方法的时候想自己定义类型形参，也是可以的。实现的方法是在**方法的返回值类型之前，访问修饰符之后使用菱形语法**：  

	public <T>　void getName(T t);
##使用的时候  
泛型在使用的时候必须指明类型才行，常见的“使用”情况：  

* 继承具有泛型的父类的时候，父类必须在菱形语法中指明泛型的具体类型：class Person extends People<String>  
* 使用带泛型的类或者接口来创建变量的时候:People<String> p;  
   
# 泛型擦除
**泛型擦除具体来说就是在编译成字节码时首先进行类型检查，接着进行类型擦除（即所有类型参数都用他们的限定类型替换，包括类、变量和方法），接着如果类型擦除和多态性发生冲突时就在子类中生成桥方法解决，接着如果调用泛型方法的返回类型被擦除则在调用该方法时插入强制类型转换。**  擦除时会转换为类型的上界，一般为Object类型，如果不想擦除成Object，就需要使用extends。

## 泛型擦除带来的问题

### 1. 运行时类型信息无法获取  
由于泛型擦除在类型检查完毕后就会将具体的类型擦除掉，因此在代码中任何获取泛型类型的信息的代码都会报错：  

	T test=new T()  

上面创建泛型类的实例就会报错，因为T的具体类型在编译完成后就被擦除了，而在运行时jvm虚拟机需要从方法区中获取该类的数据，这显然是不能办到的。  

由于泛型擦除导致运行时没有了类型的信息，所以也可以干一些奇怪的事情：  

	ArrayList<Integer> arraylist=new Arraylist<Integer>();
	arraylist.add(1);
	arraylist.getClass.getMethod("add",Object.class).invoke(arraylist,"abc");
	
由于运行时已经丢失了泛型类型的数据，所以通过反射添加一个字符串是可以的。

	ArrayList arraylist=new ArrayList<String>();
	arraylist.add(1);
	arraylist.add("hello");
	Object a=arraylist.get(0);

由于在运行时无法获取泛型的类型信息，所以arraylist添加int和string都可以，因为此时都是Object的子类型

### 2. 泛型下的继承问题  
泛型擦除并没有解决继承的问题：  

	ArrayList<Object> list1;
	ArrayList<String> list2=new ArrayList<String>();
	list1=list2;   //编译不通过

这里的编译不通过是泛型的缺陷：因为理所当然的String是Object的子类型，所以将ArrayList<String>赋值给ArrayList<Object>显得很正常，但是这两个实例没有丝毫的继承关系，因为**泛型检查的时候只会看是否是统一类型，并不会关注是否有继承关系**
	
带泛型的继承：  

	import java.lang.*;
	import java.util.*;
	class Creater<T>{
		private T value;
		public T getValue(){
			return value;
		}
		
		public void setValue(T t){
			this.value=t;
		}
	}
	
	public class Test extends Creater<String>{
		@Override
		public void setValue(String value){
			super.setValue(value);
		}
		@Override
		public String getValue(){
			return super.getValue();
		}
		public static void main(String[] args){
			Test t=new Test();
			t.setValue("hello");
			t.setValue(new Object());
		}
	}  

对上面的代码执行javac得到 ：

	Test.java:26: 错误: 对于setValue(Object), 找不到合适的方法
                t.setValue(new Object());
                 ^
    方法 Creater.setValue(String)不适用
      (参数不匹配; Object无法转换为String)
    方法 Test.setValue(String)不适用
      (参数不匹配; Object无法转换为String)
	1 个错误

提示**t.setValue(new Object());**无法编译。  
*首先由于泛型擦除的作用，Creater类的setValue()的参数应该是Object，而子类的setValue参数为String，看起来应该是子类重载了父类的setValue方法才对，那么t.setValue(new Object());就应该没有问题啊？*  

注释掉无法编译的那句话执行javap命令：  

	public void setValue(java.lang.String);
	    descriptor: (Ljava/lang/String;)V
	    flags: ACC_PUBLIC
	    Code:
	      stack=2, locals=2, args_size=2
	         0: aload_0
	         1: aload_1
	         2: invokespecial #2                  // Method Creater.setValue:(Ljava/lang/Object;)V
	         5: return
	      LineNumberTable:
	        line 17: 0
	        line 18: 5
	
	public java.lang.String getValue();
	    descriptor: ()Ljava/lang/String;
	    flags: ACC_PUBLIC
	    Code:
	      stack=1, locals=1, args_size=1
	         0: aload_0
	         1: invokespecial #3                  // Method Creater.getValue:()Ljava/lang/Object;
	         4: checkcast     #4                  // class java/lang/String
	         7: areturn
	      LineNumberTable:
	        line 21: 0

	public void setValue(java.lang.Object);
	    descriptor: (Ljava/lang/Object;)V
	    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
	    Code:
	      stack=2, locals=2, args_size=2
	         0: aload_0
	         1: aload_1
	         2: checkcast     #4                  // class java/lang/String
	         5: invokevirtual #8                  // Method setValue:(Ljava/lang/String;)V
	         8: return
	      LineNumberTable:
	        line 14: 0
	
	  public java.lang.Object getValue();
	    descriptor: ()Ljava/lang/Object;
	    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
	    Code:
	      stack=1, locals=1, args_size=1
	         0: aload_0
	         1: invokevirtual #9                  // Method getValue:()Ljava/lang/String;
	         4: areturn
	      LineNumberTable:
	        line 14: 0  

发现子类编译后有两个setValue方法，而参数一个是String一个是Object，**参数为Object的setValue就是jvm生成的桥方法，而且桥方法的实现是调用参数为String的同名方法的**。所以，**子类在定义的时候的setValue(String)方法并不是对父类方法的重载，而是一种“伪重写”，实际上生成的桥方法是对父类方法的重载，只不过桥方法也是调用setValue(String)来实现的，所以子类的方法效果就像是重写一样(注意到这里的Override并没有引起编译报错)。**  

## 3. 泛型数组的相关问题  
泛型数组只有一个问题：**Java的泛型数组不能采用具体的泛型类型进行初始化**  

	List<String>[] lsa=new List<String>[10];
	Object o=lsa;
	Object[] oa=(Object[])o;
	List<Integer> li=new ArrayList<Integer>();
	li.add(new Integer(3));
	oa[1]=li;		//ok
	String s=lsa[0].get(0);   //run-time error

由于 JVM 泛型的擦除机制，所以上面代码可以给 oa[1] 赋值为 ArrayList 也不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现 ClassCastException  

但是下面的就可以：

	List<?>[] test=new ArrayList<?>[10];
	Object o=test;
	Object[] oa=(Object[])o;
	List<Integer> a=new ArrayList();
	a.add(new Integer(1));
	oa[0]=a;
	Integer s=(Integer)lsa[0].get(0);

所以说采用通配符的方式初始化泛型数组是允许的，因为对于通配符的方式最后取出数据是要做显式类型转换的，符合预期逻辑。综述就是说 Java 的泛型数组初始化时数组类型不能是具体的泛型类型，只能是通配符的形式，因为具体类型会导致可存入任意类型对象，在取出时会发生类型转换异常，会与泛型的设计思想冲突，而通配符形式本来就需要自己强转，符合预期    

## 4. List<Object 与 List<?> 类型之间的区别  
List<?> 是一个未知类型的 List，而 List<Object 其实是任意类型的 List，我们可以把 List<String、List<Integer 赋值给 List<?>，却不能把 List<String 赋值给 List<Object  


