#Java学习笔记---面向对象  
##类  
1. 类的成员包括构造器，方法，成员变量，代码块。
**使用static修饰的成员变量和方法表明这个成员变量或者方法属于这个类本身，而不属于该类的实例，因此又称为静态方法，静态变量或者类方法，类变量；而且JAVA语法规定，类成员只能访问类成员，实例成员只能访问实例成员，二者不能互相访问**
######this关键字---使代码更加简洁  
this关键字总是指向调用该方法的对象，因此**只能在实例方法中使用**，它的主要引用场合有：  

* 构造器中引用正在初始化的对象
* 在方法中引用正在调用该方法的对象  
this关键字的主要作用就是在一个方法中调用另一个方法，JAVA语法支持对象的一个成员直接访问另一个成员，实际上只不过是省略了this罢了，默认还是调用了this。但有时却不能省略this关键字，在JAVA的方法中，对于一个变量来说，寻找一个变量的优先级别是先从循环代码块中找，其次是方法中的局部变量，最后才是成员变量（直接使用成员变量正是因为this关键字的支持），所以如果在方法中定义了一个和成员变量重名的局部变量，如果想访问这个成员变量则不能省略this关键字了。

		public class DefaultText {

	      public int count=9;
	      public void output()
	      {
              //此处定义了一个和成员变量重名的局部变量
		      int count=0;
		      System.out.println(count);
		      System.out.println(this.count);
	      }
	      public static void main(String[] args)
	      {
		      new DefaultText().output();
	      }
        }  
运行程序发现第一个输出0，第二个输出9；  
**this关键字还可以作为返回值**使用this关键字可以是代码更加简洁。  
##方法详解  
几个概念：  
*方法重载*：方法名相同形参列表不同叫做方法重载  
*参数的传递机制*：值传递，传递给形参的只是实参的值，方法体重对形参的操作并不会影响实参，但是如果形参要求是一个引用类型的变量，由于传递的是值，所以方法中实际操作的形参和实参都是指向了同一个地址，相当于操作是的是实际的数据。
关于值传递的理解看如下两个代码：  

    public class SwapText {
	public void swap(int a,int b){
		int c=a;
		a=b;
		b=c;
		System.out.println("swap方法中：a是"+a+",b是"+b);
	}
	public static void main(String[] args)
	{
		int a=9;
		int b=8;
		new SwapText().swap(a, b);
		System.out.println("swap方法交换后：a是"+a+",b是"+b);
	}
    }  
    输出结果是：swap方法中：a是8,b是9
               swap方法交换后：a是9,b是8  
可见swap方法并没有“真的”将两个数交换，那么如果形参是一个引用类型变量呢：  
    public class DefaultText {

	public int c=9;
	public int b=8;
	public void swap(DefaultText a)
	{
		int count=a.c;
		a.c=a.b;
		a.b=count;
		System.out.println("swap方法中:"+a.b+","+a.c);
		a=null;
	}
	public static void main(String[] args)
	{
		DefaultText a=new DefaultText();
		a.swap(a);
		System.out.print("交换后:"+a.b+","+a.c);
	}
    }
    输出结果是：swap方法中:9,8
               交换后:9,8
JAVA也支持形参数目可变的方法，语法是形参的类型后面加上...即可，传参数的时候也可以直接传一个数组，实际上可以把这种参数数目可变的语法就看成是传递一个数组，只不过这样写要简单一些，省去了new一个数组类型的变量，但是也降低了代码的可读性，不建议使用这种形式。*注意若有形参数目可变的用法，则只能形参的最后使用，因此一个方法最多包含一个数目可变的形参，从这也可看出这个用法的局限性很多，还不如传递一个数组来的直接*  
例如：public void test(int a,String... book){}  
##JAVA中的包和封装  
###包机制
Java允许将一组功能相关的类放在同一个package下，从而组成逻辑上的类库单元。如果希望将一个类放在指定的包结构下，应该在Java源程序的第一个非注释行放置如下代码：  

	package packagename;
位于包中的类的完整类名都应该是包名和类名的组合，因此想要使用其他包中的类的时候就必须得加很多包的前缀繁琐至极，因此JAVA支持import关键字来导入包中的一个或者全部类。  
导入全部类的语法是：  
  
	import packagename.*;
    如：import java.util.*;
导入单个类的语法是：

    import packagename.classname;  
###封装  
**定义**：封装就是将对象的状态信息隐藏在对象的内部，不允许外部程序直接访问对象的内部信息，而是通过该类所提供的方法来实现对内部信息的操作和访问，实际上封装包括两部分：把该隐藏的隐藏起来，该暴露的暴露出来。  
**实现方法**：通过访问修饰符来实现
这里就来粗略的介绍一下这几个访问修饰符：  

* **private**：当前类访问权限，只能在当前类的内部访问  
* **default**:包访问权限，什么都不加就是包访问权限，可以被相同包下的其他类访问  
* **protected**：子类访问权限，可以被不同包下的子类访问  
* **public**：公共访问权限，被所有类访问     
对于外部类而言，她可以使用的访问修饰符只有public和default两个级别，因为外部类没有处于任何类的内部，也就没有其所在类的内部，所在类的子类两个范围，因此没有private protected两个访问修饰符。  
**要想访问访问其他类中private修饰的成员，则需要通过该类提供的setter和getter方法。**
##继承  
* 重写父类方法：子类重写父类的方法则完全覆盖父类的方法，想要调用父类被覆盖的方法需要通过super关键字调用（静态的方法则需要类名），子类中的实例方法只能覆盖父类的实例方法，静态方法也是这样。*注：子类重写父类的方法要满足“两同，两小，一大”的规则：两同指的是方法名和形参列表相同，两小指的是子类方法声明抛出异常类型和子类方法的返回值类型应该比父类的范围更小或者相等，一大指定是子类方法的访问修饰符范围要大于等于父类的，如果一个方法想要被子类重写那么这个方法必须至少是protected修饰的，如果是private或者不加，即使在子类中有一个和父类名字，返回值，形参列表都相同的方法，那也不是子类重写了父类的方法只是子类定义了另一个方法，没有覆盖父类的方法*  
* 重新定义一个与父类中重名的成员变量：定义一个重名的成员变量并不会覆盖父类的变量，只是隐藏起来了，要想访问父类的这个变量同样也要super关键字。因为只是隐藏不是覆盖，所以会出现以下情况：  
	
	    class Person{
			int a=10;
		}
		class People extends Person{
			int a=9;
		}
		public class Hello {
			public static void main(String[] args)
		{
			People pp=new People();
			System.out.println(((Person)pp).a);
		}
		}
        输出是10
##多态  
JAVA**引用类型的变量**有两个类型：一个是编译时类型，一个是运行时类型。**编译时类型由声明该变量时使用的类型决定，运行时类型由实际赋给该变量的对象决定**如果编译时类型和运行时类型不一样，就会出现多态。  
看示例：  

	class Animal{
		public void eat()
		{
			System.out.println("动物都会吃");
		}
	}
	class People extends Animal{
		public void eat()
		{
			System.out.println("人也会吃");
		}
		public void write()
		{
			System.out.println("书写文字是人类特有的技能");
		}
	}
	public class Hello {
		public static void main(String[] args)
		{
			Animal animal=new People();
			animal.eat();
			//The method write() is undefined for the type Animal;
			//animal.write();
		}
	}
	会看到输出的是：人也会吃
从这个代码实例中我们来认识一下什么是多态：由于People是Animal的子类，这种向上转换Java是自动完成的，不用强制类型转化。因此，animal的编译时类型是Animal类型，运行时类型是People类型。程序执行到animal.eat();的时候因为animal中有该方法，因此编译能够通过，但因为运行时类型是People类型，所以方法执行的People中的方法；程序执行到animal.write()时就不行了，因为编译时类型是Animal类型，该类型中没有定义write方法，所以编译不能通过，报错。  
**相同类型的变量，调用同一个方法时呈现出多种不同的行为特征，这就是多态**  
可见多态之所以存在，是因为子类重写父类的方法时，完全覆盖了父类的方法；但子类的同名成员变量并不会完全覆盖父类的成员变量，所以成员变量之间不存在多态。
##初始化块  
初始化块也是进行对象的初始化操作的，*执行在构造器之前*。初始化块只有static一个修饰符，也可不加修饰符。同一类型的（静态的和普通的）可以有多个，但这么做没有意义，完全可以在一个初始化块中完成。一般初始化块用来把这个类的所有实例的共性的变量进行初始化。
如定义一个Animal类，一个planet属性，则所有的实例该属性都应该是earth，这样就可以把这个属性的初始化工作放到代码块中来。
##类成员  
###单例类  
只能创建一个实例的类叫做单例类，单例类应该具有以下特点：  

* 构造器使用private访问修饰符  
* 提供一个**public static方法**用来作为其他类访问这个类的点，该方法要返回该类的实例，即方法内部调用构造器创建一个对象  
* 要有一个成员变量储存这个单例类的实例，由于该成员变量需要被上面的静态方法访问，所以该成员变量必须是static修饰  
下面就看一个示例代码：  

		class Single{
			//作为保存单例的变量
			private static Single instance;
			public int a=9;
			public static int b=8;
			private Single(){};
			public static Single getSingle(){
				if(instance==null)
				{
					instance=new Single();
				}
				return instance;
			}
		}
		public class DefaultText {
			public static void main(String[] args)
			{
				Single sin=Single.getSingle();
				System.out.println(sin.a);
			}
		}
instance是静态的成员变量，是属于类的，但是在getSingle方法中，instance是通过构造器创建的一个类的实例
。也就是说instance是一个实例，在instance中没有private static Single instance这个属性，因为这是个静态成员。同样instance中有a成员，但是没有b成员。  
##final修饰符  
final修饰的类，方法，变量是不可改变的。  
###final修饰成员变量  
由于系统会为成员变量进行隐式的初始化，而final修饰后只能进行一次初始化操作，这么一来的话就没有什么意义了。所以，**规定final修饰的成员变量必须显式初始化**。

* 类变量：静态初始化块，声明变量的时候指定初始值  
* 实例变量：初始化块，声明变量的时候指定初始值，构造器中指定初始值  
*注意*：final修饰引用类型变量的时候有些特殊情况，由于引用类型变量保存的是地址值，final修饰后只是保证这个地址值不变，但是地址所对应的内容是可以变的。  
###final变量的宏替换  
某些情况下变量不再是一个变量，系统会把他当成一个常量对待，则需要满足以下几个条件：  

* 使用final修饰  
* 在声明变量的时候指定初始值  
* 该初始值在编译的时候就可以被确定下来  
一旦满足同时以上三个条件，则系统就会把这个变量当成常量看待，常量就是指定的初始值，用到该变量的地方就自动替换成该变量的值。  

		public class DefaultText {
			public static void main(String[] args)
			{
				final String str="hello world";
				final String str2=new String("hello world");
				String str3="hello world";
				String a="hello world";
		
				String s1="hello";
				String s2=" world";
				final String s3="hello";
				final String s4=" world";
		
				String b=str;
				String c=str2;
				String d=s1+s2;
				String e=s3+s4;
				
				System.out.println(a==str3);
				System.out.println(a==b);
				System.out.println(a==c);
				System.out.println(a==e);
				System.out.println(a==d);
			}	
			}
    	输出是：true
		       true
			   false
			   true
			   false
下面就来分析一下，首先代码中的所有在声明变量后直接赋值的值（“hello world”,"hello","world",不算new String（“hello world"))由于这些字符串的值都可以在编译的时候确定下来，所以这些字符串都是有Java中的常量池来储存的，也就是说a引用在运行时指向常量池中的hello world，str3引用运行时也指向常量池中的字符串，所以*a==str3*输出是true；由于str满足宏替换的三个条件，所以系统把str当成常量处理，所以第二个*a==b*为true；由于str2的值无法在编译的时候确定下来，所以str2地址运行时并不是指向常量池中的字符串，而是指向堆内存中的String对象，所以*a==c*返回的是false；s3和s4都满足宏替换的条件，系统把他们当成常量处理，所以e在编译阶段就可以确定值，所以*a==e*返回的是true；而s1和s2不满足宏替换的第一个条件，系统不把他们当成常量处理，所以d的值在编译阶段不能确定，即d在运行时并不是指向常量池中的字符串，所以最后一个返回false。  
##接口  
成员：  

* 成员变量（只能是静态变量）默认使用public static final修饰，访问修饰符如果要显式加只能加public。
* 方法（抽象方法，类方法（默认且只能是public），默认方法（默认且只能是public访问权限，default修饰）），接口里的普通方法默认加上public abstract修饰符  
* 内部类  
由于接口定义的是一种规范，所以其成员都默认是public访问权限的，显示加只能是加public访问权限，我认为由于接口中不能存在普通方法，默认方法就是来弥补这个不足的。  
###接口的作用  

* 定义变量  
* 调用接口中定义的常量（因为都是final修饰的静态成员变量，必须显示初始化，所以满足宏替换的条件）  
* 被其他类实现，其他类必须实现接口中全部的抽象方法  
##Lambda表达式  
Java8新增的Lambda表达式简化了使用匿名内部类时候的一些繁琐的创建匿名内部类，重写方法等代码。我们使用匿名内部类的情景都是要使用匿名内部类的重写的方法，如果一个普通方法的形参是一个匿名内部类，这时候就可以使用Lambda表达式了。表达式由三部分组成：  

* 形参列表。如果只有一个形参，则这个形参列表的括号也可以省略。  
* 箭头->  
* 代码块：就是匿名内部类想要重写的方法的方法体  

　


