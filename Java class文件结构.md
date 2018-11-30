# Java的class文件格式  
## 一、常量池  
class文件中的常量池中有很多**info**，之前一直被迷惑，再次看书有所觉悟，在此记录一下：  

### 1. 数据的真正记录者  
这里的**数据的真正记录者**指的是常量池中不需要通过**index这种指向常量池的其他项**的数据来作为自己的部分数据的**info**。比如：CONSTANT_Class_info中的name_index就是引用常量池中的Utf8_info的值来记录的。因此刨去这种**info**，常量池中数据的真正记录者只有如下的几个：  

#### 1.1 CONSTANT_Integer_info为代表的数据  
结构为：  

	CONSTANT_Integer_info{
		u1 tag;
		u4 bytes;
	}  
其中的u4就直接保存了数据常量，并没有通过index来引用常量池中其他地方的数据  

当然还有：  

	CONSTANT_Float_info{
		u1 tag;
		u4 bytes;
	}
	
	CONSTANT_Long_info{
		u1 tag;
		u4 high_bytes;
		u4 low_bytes;
	}

	CONSTANT_Double_info{
		u1 tag;
		u4 high_bytes;
		u4 low_bytes;
	}  

上面的**info**用来记录**数据**常量：  

	private int a=100;
	private float b=200;

	上面的100和200就被储存在一个integer_info和Float_info中  

#### 1.2 CONSTANT_Utf8_info  
这个表保存的数据最多、最杂了，保存除了**数字以外的一切字符**；因此，其他各种表格中的各种**index归根结底都是引用的Utf8_info数据**  

	CONSTANT_Utf8_info{
		u1 tag;
		u2 length;
		u1 bytes[length];
	}  

该表中保存的数据有：**类的全限定名字符串、字段和方法的名称字符串、各种属性的名称字符串(如Code、SourceFile)、方法和字段的描述符字符串、String常量(如：private String name = "hello"中的hello)、字段的类型字符(如：int数据为I)**  

实例：  

	class Parent{}
	public final class Test extends Parent{
	  public static final String world="world";
	  private short a=9;
	  private int b=10;
	  private String name="hello";
	  public String sex=null;
	  public static void main(String[] args){
	    Test t=new Test();
	    float c=1.0f;
	    int d=1;
	    byte e=1;
	    System.out.println(c-d);
	    System.out.println(d-e);
	  }
	}  

其生成的class文件，使用javap查看后：  

	Constant pool:
	   #1 = Methodref          #12.#31        // Parent."<init>":()V
	   #2 = Fieldref           #7.#32         // Test.a:S
	   #3 = Fieldref           #7.#33         // Test.b:I
	   #4 = String             #34            // hello
	   #5 = Fieldref           #7.#35         // Test.name:Ljava/lang/String;
	   #6 = Fieldref           #7.#36         // Test.sex:Ljava/lang/String;
	   #7 = Class              #37            // Test
	   #8 = Methodref          #7.#31         // Test."<init>":()V
	   #9 = Fieldref           #38.#39        // java/lang/System.out:Ljava/io/PrintStream;
	  #10 = Methodref          #40.#41        // java/io/PrintStream.println:(F)V
	  #11 = Methodref          #40.#42        // java/io/PrintStream.println:(I)V
	  #12 = Class              #43            // Parent
	  #13 = Utf8               world
	  #14 = Utf8               Ljava/lang/String;
	  #15 = Utf8               ConstantValue
	  #16 = String             #13            // world
	  #17 = Utf8               a
	  #18 = Utf8               S
	  #19 = Utf8               b
	  #20 = Utf8               I
	  #21 = Utf8               name
	  #22 = Utf8               sex
	  #23 = Utf8               <init>
	  #24 = Utf8               ()V
	  #25 = Utf8               Code
	  #26 = Utf8               LineNumberTable
	  #27 = Utf8               main
	  #28 = Utf8               ([Ljava/lang/String;)V
	  #29 = Utf8               SourceFile
	  #30 = Utf8               Test.java
	  #31 = NameAndType        #23:#24        // "<init>":()V
	  #32 = NameAndType        #17:#18        // a:S
	  #33 = NameAndType        #19:#20        // b:I
	  #34 = Utf8               hello
	  #35 = NameAndType        #21:#14        // name:Ljava/lang/String;
	  #36 = NameAndType        #22:#14        // sex:Ljava/lang/String;
	  #37 = Utf8               Test
	  #38 = Class              #44            // java/lang/System
	  #39 = NameAndType        #45:#46        // out:Ljava/io/PrintStream;
	  #40 = Class              #47            // java/io/PrintStream
	  #41 = NameAndType        #48:#49        // println:(F)V
	  #42 = NameAndType        #48:#50        // println:(I)V
	  #43 = Utf8               Parent
	  #44 = Utf8               java/lang/System
	  #45 = Utf8               out
	  #46 = Utf8               Ljava/io/PrintStream;
	  #47 = Utf8               java/io/PrintStream
	  #48 = Utf8               println
	  #49 = Utf8               (F)V
	  #50 = Utf8               (I)V  

可以看到Utf8的数据就是我们上面总结的那样  

### 2. 其他  
除了上面的两个**info**之外，其他的表格的数据都是引用其他的表格得到的。  

#### 2.1 Class_info  
该结构用于表示一个类或者接口：  

	CONSTANT_Class_info{
		u1 tag;
		u2 name_index;
	}

name_index引用一个Utf8数据，用于表示类的名字。

#### 2.2 CONSTANT_NameAndType_info
用于**表示字段或者方法**，注意，从该info的名字完全看不出来，它是用来表示字段或者方法的，但是从它的结构就可以看出来，确实是这样的：  

	CONSTANT_NameAndType_info{
		u1 tag;
		u2 name_index;
		u2 descriptor_index;
	}

name_index引用一个Utf8数据，用来表示该字段或者方法的名字；descriptor_index也是引用一个Utf8数据，用来表示字段或者方法的描述符。  

#### 2.3 Fieldref_info、Methodref_info、InterfaceMethodref_info  
看名字就知道这三个表格对应的是：**字段、方法、接口方法的引用**。  

三个表格的格式都一样：	  
	
	CONSTANT_XXX_info{
		u1 tag;
		u2 class_index;
		u2 name_and_type_index;
	}

**其中的class_index是引用一个Class_info数据，代表该字段或者方法或者接口方法所属于的类；name_and_type_index则是引用一个NameAndType_info数据，代表字段或者方法的描述符**  

上面提到过，**NameAndType_info才是表示字段或者方法**，那么这里的三个是什么呢？上面说的**字段、方法、接口的引用**又是什么意思呢？  

经过测试，**方法只有被调用了、字段只有被访问了，才会在class文件中生成Methodref、Fieldref的对应项**；难道说，只有方法或者字段被访问才能称作是方法或者字段吗？这肯定不是，所以这三个是**字段、方法、接口方法的引用**才更恰当；而**NameAndType_info**才代表的是字段或者方法。  

#### 2.4 CONSTANT_String_info  
这个表象很简单，就是代表一个字符串常量数据：  

	CONSTANT_String_info{
		u1 tag;
		u2 string_index;
	}

string_index引用一个Utf8数据，表示该String对象的值，比如private String name=“hello”，hello被写进一个Utf8_info中，而后会在创建一个String_info来指向这个Utf8_info。  

## 二、字段和方法  
常量池后面的两个重要的东西就是**字段和方法**的**info**了。  

常量池中的**NameAndType_info**我们说代表了字段和方法，那里只是代表了字段和方法的名字和类型，然而，定义一个字段或者方法除了这两个基本的元素外，还有很多其他的信息：**访问权限、final、static、volatile(字段)、方法还会有方法体里面的代码等**。因此，字段和方法真正的完整描述都是在**field_info和method_info**这两个**和常量池同一级别的info**中。  

	field_info{
		u2 access_flags;
		u2 name_index;
		u2 descriptor_index;
		u2 attributes_count;
		attribute_info attribute[attributes_count];
	}

	method_info{
		u2 access_flags;
		u2 name_index;
		u2 descriptor_index;
		u2 attribute_count;
		attribute_info attributes[attributes_count];
	}

这两个**info**中的**access_flags都是表示访问权限的一种掩码**，但是，字段和方法的这个值会有一些不同，比如字段中有ACC_VOLATILE，而方法中会有ACC_NATIVE等。**name_index和descriptor_index都分别表示名字和描述符**，都是引用常量池中的Utf8中的数据；**attribute_info**则是属性表，包含了一些其他的**附加信息**，比如方法会有一个Code属性用来保存方法体中用到的字节码指令。  

## 三、attribute_info  
jvm中的属性有很多。  

### 1. ConstantValue  
该属性是一个定长属性，其结构为：  

	ConstantValue_info{
		u2 attribute_name_index;
		u4 attribute_length;
		u2 constantvalue_index;
	}

其中的**attribute_name_index**指向一个常量池中的**Utf8**数据，而第二个**attribute_length**固定为2，**constantvalue_index**也是一个指向常量池中的一个与该字段相同类型的数据，由于后者中只有基本类型和String才会在常量池中生成常量，所以**ConstantValue**属性也只限于基本类型和String。  

而该属性的意义是**通知虚拟机自动为静态变量赋值**。也就是说，在JVM的规范中，只要字段被static修饰了就会生成这个属性，并没有final啥事儿；但是javac对这个做了特殊的处理，之后同时被static和final修饰的字段才会通过**ConstantValue**来进行初始化，而不是通过**类构造器<clinit>**  

### 2. Code  
这个属性是方法才有的属性，里面封装了方法体编译后的字节码指令以及一些其他的信息：  

	Code_attribute{
		u2 attribute_name_index;
		u4 attribute_length;
		u2 max_stack;
		u2 max_locals;
		u4 code_length;
		u1 code[code_length];
		u2 exception_table[exception_length];
		{
			u2 start_pc;
			u2 end_pc;
			u2 handler_pc;
			u2 catch_type;
		}  exception_table[exception_table_length];
		u2 attributes_count;
		attribute_info attributes[attribute_count];
	}   

其中的**attribute_name_index**是一个指向常量池的Utf8的数据，用来表示该属性的名字(Code)；这里的**attribute_length**表示当前属性的长度，不包括初始的6个字节；**max_stack**表示当前操作数栈在方法执行的任何时间点的最大深度；**max_locals**表示分配在当前方法引用的局部变量表中的局部变量个数；**code_length**给出了当前方法**code[]**数组的字节数，应该就是方法体中jvm指令的个数；**exception_table_length**给出了exception_table表的成员个数；**exception_table[]**数组的每个成员表示**code[]数组中的一个异常处理器**；而exception_table中的每一个元素都有下面几项：**start_pc和end_pc**，其中start_pc必须是对当前code[]中某一项指令操作码的有效索引，end_pc的值要么是code[]中某一个操作吗的有效索引，要么等于code[]的length；**handler_pc**的值表示一个异常处理器的起点，其值必须同时是对当前code[]和其中某一指令操作码的有效索引；**catch_type**如果该项的值不为0，那么它必须是对常量池表的一个有效索引，该索引必须是**CONSTANT_Class_info**结构，用来**表示当前异常处理器需要捕捉的异常类型**。  

### 3. Exception  
该属性位于method_info结构中，指出了一个方法可能抛出的受检异常  

	Exception_attribute{
		u2 attribute_name_index;
		u4 attribute_length;
		u2 number_of_exceptions;
		u2 exception_index_table[number_of_exceptions];
	}

前两个就不说了，和前面的属性是一样的；第三个也很简单，第四个属性中的每一项都是一个**引用自常量池中的Class_info数据，表示要抛出的异常所属的类的类型**  

