# Kotlin学习  
## 一、 基础  
### 判空  
当某个变量可以为null的时候，**必须**在声明处的类型后面添加**？**来标识该引用可以为空。  
注意：不加这个问号标识并不是说定义的引用不会为null，也可以为空；只不过这时候如果对这个引用进行操作的话编译器可能会报错；所以**？的作用**在我看来是：**对于不能为空的变量，提示你进行判空**。而在使用**==null**进行判空的时候，如果不为空会自动将引用转变为**非空值（non-nullable）***（类似于is判断是否为子类型那样，判断成功后可以当做那个类型使用）*，这么一来，就保证了非空的逻辑安全。   
>当一个基本类型的变量添加了“？”进行判空后，**就被自动装箱了，不在具有同一性**  

### 变量定义  
**val 表示不可变，类似于Java语法中的final修饰；var 表示可变，和Java常见的普通的定义变量一样**  

### ==和===  
在Kotlin中**==**判断的是两个变量的值是否相等，是**一种值比较**；而**===**则是对*同一性*的比较，类似于Java语法中的引用类型变量的比较
## 二、 基础类型  
1. 不支持八进制  
2. 不支持隐式转换
3. **Char不能与Int互用(Char在Kotlin中不算做数字类型，而是单独的一个字符类型，但是所有的数字类型都可以使用toChar()方法显示转换成Char)**
4. 对于长的数字支持下滑线分割：如1000_0000
5. 没有自动装箱（除非**使用可空引用？或者泛型**，这里的自动装箱在代码层面是没有什么可见的区别的，但是在编译为class文件后，自动装箱就会被编译为Java中的Integer类型，而平时都是Java中的int型）  
6. 每个类型都支持**toXXX()**的方法实现类型转换  

> 因为在Java中的基本数据类型，对其进行赋值只是保存了值，而其不可为空，因此，Kotlin引用Java代码时，其中的基本数据类型就被当做**不可为空的Kotlin基本类型**；同样，其包装类型会变成**可空的Kotlin基本类型**。

### 2.1 数组  
数组在Kotlin中使用**Array类**来表示，它定义了**get和set**函数**（按照运算符重载约定这会转变为 []）**  

	class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ……
	}  

我们可以使用库函数 **arrayOf()** 来创建一个数组并传递元素值给它，这样 **arrayOf(1, 2, 3)** 创建了 **array [1, 2, 3]**。 或者，库函数 **arrayOfNulls<>()** 可以用于创建一个指定大小的、所有元素都为空的数组。  
**另一个选项是用接受数组大小和一个函数参数的 Array 构造函数，用作参数的函数能够返回给定索引的每个元素初始值：**

	// 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]
	val asc = Array(5, { i -> (i * i).toString() })  

**[]** 运算符代表调用成员函数 **get() 和 set()**。  

>Kotlin 也有无装箱开销的专门的类来表示原生类型数组: ByteArray、 ShortArray、IntArray 等等。这些类和 Array 并没有继承关系，但是它们有同样的方法属性集。  

*问题1：kotlin中初始化数组，只指定大小的实现方式？类似于Java中int[] =new int[9];* 
>**使用arrayofNulls<>(int size)可以实现；** 

*问题2：kotlin实现多维数组？*

### 2.2 字符串  
Kotlin有两种类型的字符串字面值：**转义字符串和原始字符串**；转义字符串就像Java里的字符串一样，里面的**转义字符会被识别：**  

	val text="hello world \n"  

原始字符串用**三个引号(""")**分界符括起来，内部没有转义字符这一说，但是可以**包含换行和其他的任意字符**：  

	val text = """
    for (c in "foo")
        print(c)
	"""  
在原始字符串中可以调用**trimMargin()**方法**去掉前导空格**接下来就看看调用这个方法的效果：  

	fun main(args: Array<String>) {
    println("Hello, world!")
    var test="""
    |today
    |is
    |not
    |Tuesday
    """.trimMargin()
    println(test)
	}   
	output：  
	Hello, world!
	today
	is
	not
	Tuesday   

	//不加方法  
	fun main(args: Array<String>) {
    println("Hello, world!")
    var test="""
    today
    is
    not
    Tuesday
    """
    println(test)
	}  
	output：  
	Hello, world!
                 //这里有空行是紧跟着"""后面的空行
    today
    is
    not
    Tuesday

**字符串模板：**kotlin中的字符串可以包含模板表达式，即一小段代码，会求值并把结果合并到字符串中。模板表达式以美元符($)开头，由一个简单的名字构成：  

	val i=10;
	println("i=$i")  //输出i=10  

或者用花括号括起来的任意表达式：  

	val s = "abc"
	println("$s.length is ${s.length}") // 输出“abc.length is 3”  

>**理解：字符串模板中$后面的名字或者花括号中的内容会进行运算**

原始字符串和转义字符串内部都支持模板，Kotlin推荐用字符串模板代替重载的+来实现字符串的连接运算。

## 三、控制流  
### 3.1 if  
if语句与Java相比主要的功能都一样，只是扩展了一下：**kotlin中的if可以作为表达式而不是语句，而表达式是有返回值的，默认各个条件代码的最后一句就是返回的值**；正是这一功能，传统的(?:)三元操作符在kotlin中没有了，因为这用普通的if表达式就可以实现：  

	var max=if(a<b) b else a  
>想要使用if表达式而不是if语句，**必须用上else分支**，很容易想明白，因为有了else才能保证所有的情况都被考虑到了，才能对每种情况都能够赋值，不会出现if语句不满足时没有值，满足时有值可赋这种两面性的情况  

### 3.2 when  
kotlin中的when就是Java中的switch；同样在kotlin中**when可以被当做语句使用，也可以被用作表达式。**如果它被当做表达式， 符合条件的分支的值就是整个表达式的值。
  	
	when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // 注意这个块
        print("x is neither 1 nor 2")
    }
	}

>如果 when 作为一个表达式使用，则**必须有 else 分支**， 除非编译器能够检测出所有的可能情况都已经覆盖了。

如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔：  

	when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
	}  

我们可以用任意表达式（而不只是常量）作为分支条件  

	when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
	}  

我们也可以检测一个值在**（in）**或者不在**（!in）**一个区间或者集合中：  

	when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
	}  

另一种可能性是检测一个值是（**is**）或者不是（**!is**）一个特定类型的值。注意： 由于智能转换，你可以访问该类型的方法和属性而无需任何额外的检测。  

#### when结构中使用任意对象  
Kotlin的when表达式比Java强的地方还有很多，其中一个就是**when结构中可以使用任意对象**：  

	fun mix(c1:Color,c2:Color){
		when (setof(c1,c2)){
			setof(RED,YELLOw) -> ORANGE
			setof(YELLOW,BLUE) -> GREEN
		}
	}      
	// 上面的类型是枚举类

这里就体现了Kotlin的when结构的强大之处：只要setof(c1,c2)中的元素和when分支中的一样，就会满足条件

#### 不带参数的when  
when的分支

### 3.3 for语句  
for 循环可以对任何提供迭代器（iterator）的对象进行遍历，这相当于像 C# 这样的语言中的 foreach 循环。语法如下：  

	for (item in collection) print(item)  
	for (item: Int in ints) {
    // ……
	}  

如上所述，for 可以循环遍历任何提供了迭代器的**对象**。即：

* 有一个成员函数或者扩展函数 iterator()，它的返回类型
	* 有一个成员函数或者扩展函数 next()，并且
	* 有一个成员函数或者扩展函数 hasNext() 返回 Boolean。  
	
这三个函数都需要**标记为 operator**。  

上面是对循环遍历对象时候的要求，如果是循环迭代是在数字区间上进行的，**for 循环会被编译为并不创建迭代器的基于索引的循环。**  
如果你想要**通过索引遍历一个数组或者一个 list**，你可以这么做：  
	
	for (i in array.indices) {
    	println(array[i])
	}  

或者你可以用**库函数 withIndex：**  

	for ((index, value) in array.withIndex()) {
    	println("the element at $index is $value")
	}    

**遍历map：**  

	for ((k,v) in map)  
	包括上面的通过withIndex遍历数组一样，这里的k和v以及上面的index和value都是可以随便换个符号代替的

**区间：**  

	for (i in 1..100)
	for (i in 1..100 step 2)  

**倒序：**  

	for (i in 100 downTo 1)  
	for (i in 100 downTo 1 step 2)    

基本上常用的for循环都可以用区间来实现。
  
### 3.4 语句缩写  

Kotlin中还有很多简化了的缩写的语句的实现：  

* *if null:***?:**  
* *if not null:***?.**  
* *if not null else:***?.XXX?:XXX**  

**with:对一个对象实例调用多个方法**  

	val test=Persion()  
	with(test){
		method1  
		method2  
		...
	}  

>个人感觉**with更像是一个类似于for的语句，没有返回值，而且在with的花括号里面调用方法不需要使用“对象.方法”的形式**  



## 四、类和继承  

### 4.1 类的定义  
Kotlin中类的完整定义是：  

	//类名、类头（主构造函数、主构造函数的参数列表、类体）
	class className [constructor(...){}]   //只有类名是必须的，其他的都是可选的  

一个类可以有**一个主构造函数和一个或多个次构造函数；**在主构造函数中，如果**没有任何注解和可见性修饰符**，可以省略**constructor关键字**。
>在Java的构造器中，一种很常见的传参的目的是，在构造器中使用this.xxx=xxx;来实现初始化，**而kotlin的主构造函数并不用这么麻烦，你传入的参数自动完成初始化，不用专门this.xxx=xxx；了**

#### 4.1.1 主构造函数  
主构造函数不能包含任何的代码，初始化的代码可以放到以**init关键字**作为前缀的**初始化块**中；而主构造函数的参数因为添加为了该类的属性，所以自然也可以在初始化块儿中使用。    

使用主构造函数有两种方式：  

	class Persion(name:String){
	}  
	class Persion(var name:String){}  

**上面的形式不会对name属性进行初始化，会让你在init块儿中显示进行初始化；下面的形式则在参数传入的时候自动实现赋值操作。**  

>上面说过，主构造函数的参数在传入的时候自动就初始化了，那么我们还要这个init初始化块干什么？他好像不用执行初始化代码啊？且不说初始化除了为属性赋值外还可以做一些其他的动作，而这些动作只能在初始化块儿中进行；初始化块儿还可以进行一些默认初始化。比如我在主构造函数中声明一个可以为null的属性，当创建一个实例的时候，如果传入的改参数为空，就赋值为一个默认的值（这么做很蠢，完全可以不在主构造函数的参数中声明，或者在次构造函数委托的时候显示为主构造函数的参数赋值），这时候可以在初始化块儿中进行为null的判断。  

#### 4.1.2 次构造函数  
类可以声明一个或者多个次构造函数，次构造函数有几点刚性要求:  

* 如果有主构造函数，每个次构造函数必须或直接或间接的委托给主构造函数，即在次构造函数后面添加this(...)  

		class Person(val name: String) {
    		constructor(name: String, parent: Person) : 	this(name) {
        		parent.children.add(this)
    		}
		}  

>**请注意，初始化块中的代码实际上会成为主构造函数的一部分。委托给主构造函数会作为次构造函数的第一条语句，因此所有初始化块中的代码都会在次构造函数体之前执行。即使该类没有主构造函数，这种委托仍会隐式发生，并且仍会执行初始化块**

* 次构造函数的参数不会添加为类的属性，且不能使用var或者val修饰  


### 4.2 继承  
在 Kotlin 中所有类都有一个共同的超类 Any，这对于没有超类型声明的类是默认超类。  
要声明一个显式的超类型，我们把类型放到类头的冒号之后  

	open class Base(p: Int)

	class Derived(p: Int) : Base(p)  

>类上的 **open** 标注与 Java 中 final 相反，它**允许其他类从这个类继承**。**默认情况下，在 Kotlin 中所有的类都是 final**， 对应于 Effective Java书中的第 17 条：**要么为继承而设计，并提供文档说明，要么就禁止继承**。  


如果派生类有一个主构造函数，其基类型可以（**并且必须**） 用基类的主构造函数参数就地初始化。也就是说，如果子类含有主构造函数，那么必须委托给父类的主构造函数就地初始化，所以子类的主构造函数的参数必须和父类的参数一样。

如果类没有主构造函数，那么每个次构造函数必须使用 super 关键字初始化其基类型，或委托给另一个构造函数做到这一点。 注意，在这种情况下，不同的次构造函数可以调用基类型的不同的构造函数(**这时候就可以调用基类的任何一个构造函数了，不仅仅是主构造函数**)：  

	class MyView : View {
    	constructor(ctx: Context) : super(ctx)

    	constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
	}  

#### 4.2.1 覆盖方法  
与 Java 不同，Kotlin 需要显式标注可覆盖的成员（我们称之为开放）和覆盖后的成员：  

	pen class Base {
    	open fun v() {}
    	fun nv() {}
	}
	class Derived() : Base() {
    	override fun v() {}
	}   
Derived.v() 函数上必须加上 **override标注**。如果没写，编译器将会报错。 如果函数没有标注 open 如 Base.nv()，则*子类中不允许定义相同签名的函数*， 不论加不加 override。在一个 final 类中（没有用 open 标注的类），开放成员是禁止的。  
标记为 override 的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，使用 final 关键字：  

	open class AnotherDerived() : Base() {
    	final override fun v() {}
	}  


#### 4.2.2  覆盖属性  

属性覆盖与方法覆盖类似；在超类中声明然后在派生类中重新声明的属性必须以**override**开头，并且它们必须具有兼容的类型。每个声明的属性可以由具有初始化器的属性或者具有 getter 方法的属性覆盖。  
你也可以用一个 var 属性覆盖一个 val 属性，但反之则不行。这是允许的，因为一个 val 属性本质上声明了一个 getter 方法，而将其覆盖为 var 只是在子类中额外声明一个 setter 方法。    
**请注意，你可以在主构造函数中使用 override 关键字作为属性声明的一部分。**

>**注意：这里的覆盖也和Java一样，可以通过super访问到父类的属性** 
#### 4.2.3 调用超类实现  
派生类中的代码可以使用 super 关键字调用其超类的函数与属性访问器的实现：  

	open class Foo {
    	open fun f() { println("Foo.f()") }
    	open val x: Int get() = 1
	}

	class Bar : Foo() {
    	override fun f() { 
        	super.f()
        	println("Bar.f()") 
    	}
    
    	override val x: Int get() = super.x + 1
	}  

在一个**内部类中访问外部类的超类**，可以通过由外部类名限定的 super 关键字来实现：super@Outer：  

	class Bar : Foo() {
    	override fun f() { /* …… */ }
    	override val x: Int get() = 0
    
    	inner class Baz {
        	fun g() {
            	super@Bar.f() // 调用 Foo 实现的 f()
            	println(super@Bar.x) // 使用 Foo 实现的 x 的 getter
        	}
    	}
	}  

#### 4.2.4 覆盖规则  
在 Kotlin 中，实现继承由下述规则规定：如果一个类从它的直接超类继承相同成员的多个实现， 它必须覆盖这个成员并提供其自己的实现（也许用继承来的其中之一）。 为了表示采用从哪个超类型继承的实现，我们使用由尖括号中超类型名限定的 super，如 super<Base>：  

	open class A {
    	open fun f() { print("A") }
    	fun a() { print("a") }
	}

	interface B {
    	fun f() { print("B") } // 接口成员默认就是“open”的
    	fun b() { print("b") }
	}

	class C() : A(), B {
    	// 编译器要求覆盖 f()：
    	override fun f() {
        	super<A>.f() // 调用 A.f()
        	super<B>.f() // 调用 B.f()
  		}
	}  

同时继承 A 和 B 没问题，并且 a() 和 b() 也没问题因为 C 只继承了每个函数的一个实现。 但是 f() 由 C 继承了两个实现，所以我们必须在 C 中覆盖 f() 并且提供我们自己的实现来消除歧义。    

#### 4.2.5 静态方法的实现  
kotlin中实现java中的静态方法的方式有两种：  

* 通过使用包级函数实现  
* 通过伴生对象，在伴生对象中**object**中声明方法，然后通过**类名.方法名**调用

### 4.3 属性和字段  
上面的一种声明属性的方式是作为主构造函数的参数，另一种方式就是在类体中声明：  

	class Address {
    	var name: String = ……
    	var street: String = ……
    	var city: String = ……
    	var state: String? = ……
    	var zip: String = ……
	}  

>在**类体中声明属性的时候必须给赋初值(叫做初始器)**，如果使用**lateinit**修饰表示延迟初始化属性，可以不赋初值。    
>**lateinit只能修饰非空的属性，且不能是基本类型** 

其初始器（initializer）、getter 和 setter 都是可选的。属性类型如果可以从初始器 （或者从其 getter 返回值，如下文所示）中推断出来，也可以省略。   

#### 4.3.1 Getter和Setter  
声明一个属性的完整语法是：  

	var <propertyName>[: <PropertyType>] [= <property_initializer>]
    	[<getter>]
    	[<setter>]    

其初始器（initializer）、getter 和 setter 都是可选的。属性类型如果可以从初始器 （或者从其 getter 返回值，如下文所示）中推断出来，也可以省略。  

*思考：上面说过，在类体中的属性必须赋初值啊，这里怎么又说初始器是可选的啊？*  

>1. 如果使用lateinit关键字修饰的话就可以不赋初值 2. 如果该属性的所有访问器(**get和set方法**)都不是默认的访问器，则**不可以赋初值**  

我们可以编写自定义的访问器，非常像普通函数，刚好在属性声明内部。这里有一个自定义 getter 的例子:  

	val isEmpty: Boolean
    	get() = this.size == 0  

 一个自定义的 setter 的例子:  

	var stringRepresentation: String
    	get() = this.toString()
    	set(value) {
        	setDataFromString(value) // 解析字符串并赋值给其他属性
    }  

按照惯例，setter 参数的名称是 value，但是如果你喜欢你可以选择一个不同的名称。  

##### 幕后字段  
在 Kotlin 类中不能直接声明字段。然而，当一个属性需要一个幕后字段时，Kotlin 会自动提供。这个幕后字段可以**使用field标识符**在访问器(**get和set方法**)中引用：

	var counter = 0 // 注意：这个初始器直接为幕后字段赋值
    	set(value) {
        	if (value >= 0) field = value
    }  
field 标识符只能用在属性的访问器内。  

如果属性至少一个访问器使用默认实现，或者自定义访问器通过 field 引用幕后字段，将会为该属性生成一个幕后字段。  

例如，下面的情况下， 就没有幕后字段：  

	val isEmpty: Boolean
    	get() = this.size == 0  

>记得上面的问题，说为什么先说类体中声明的属性必须赋初值，又说其初始器是可选的？原来**初始器是为幕后字段赋值**，所以赋初值的前提是拥有幕后字段    

*思考：这里的get和set方法如何实现JavaBean中的要求？*    

之前一直不明白幕后字段以及getter和setter到底是怎么一回事，看了这篇文章后才恍然大悟：[Android技术杂货铺](https://mp.weixin.qq.com/s/J4yJYq9L4FK8AomS_7JjZA "")；原来是**在setter和getter不能通过this.xxx来访问属性**，因为在本身的**对象.xxx**的形式就是通过调用getter或者setter实现的，这时候如果在setter和getter中这么使用的话就是一种无限递归了，会造成stackoverflow异常，所以正确的访问该属性是通过**field这个幕后字段来访问**

##### 延迟初始化    
一般地，属性声明为非空类型必须在构造函数中初始化。 然而，这经常不方便。例如：属性可以通过依赖注入来初始化， 或者在单元测试的 setup 方法中初始化。 这种情况下，你不能在构造函数内提供一个非空初始器。 但你仍然想在类体中引用该属性时避免空检查。

为处理这种情况，你可以用 **lateinit** 修饰符标记该属性

**该修饰符只能用于在类体中的属性（不是在主构造函数中声明的 var 属性，并且仅当该属性没有自定义 getter 或 setter 时），而自 Kotlin 1.2 起，也用于顶层属性与局部变量。该属性或变量必须为非空类型，并且不能是原生类型。**

### 4.4 数据类  
我们经常创建一些只保存数据的类(比如JavaBean)，在这些类中，一些标准函数往往是从数据机械推导出来的。在Kotlin中，这叫做**数据类并标记为data：**  

	data class User(val name: String, val age: Int)  

编译器自动从**主构造函数中声明的所有属性**(在类体中声明的属性并不会有这种性质)导出以下成员：

* equals()/hashCode() 对  
* toString() 格式是 "User(name=John, age=42)"  
* componentN() 函数 按声明顺序对应于所有属性  
* copy() 函数（见下文）  

为了确保生成的代码的一致性和有意义的行为，数据类必须满足以下要求：  

* 主构造函数需要至少有一个参数；  
* 主构造函数的所有参数需要标记为 val 或 var；  
* 数据类不能是抽象、开放、密封或者内部的  
* （在1.1之前）数据类只能实现接口  

此外，成员生成遵循关于成员继承的这些规则：  

* 如果在数据类体中有显式实现 equals()、 hashCode() 或者 toString()，或者这些函数在父类中有 final 实现，那么不会生成这些函数，而会使用现有函数；  
* 如果超类型具有 open 的 componentN() 函数并且返回兼容的类型， 那么会为数据类生成相应的函数，并覆盖超类的实现。如果超类型的这些函数由于签名不兼容或者是 final 而导致无法覆盖，那么会报错；  
* 不允许为 componentN() 以及 copy() 函数提供显式实现。  

#### 4.4.1 复制函数  
在很多情况下，我们需要复制一个对象改变它的一些属性，但其余部分保持不变。 copy() 函数就是为此而生成。对于上文的 User 类，其实现会类似下面这样：  

	fun copy(name: String = this.name, age: Int = this.age) = User(name, age)   

这让我们可以写：  

	val jack = User(name = "Jack", age = 1)
	val olderJack = jack.copy(age = 2)  
注意：我们通常不需要重写该方法  

### 5. 嵌套类和枚举类  
#### 5.1 嵌套类  
嵌套类很简单，把类定义在一个类里就可以了：  

	class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
	}   

使用上和Java中的嵌套类很像：  

	val demo = Outer.Nested().foo() // == 2  

#### 5.2 内部类  
内部类和嵌套类一样，首先也要定义在类里面，只是多了一个**inner标识**，表示可以访问外部类的成员，内部类会带有一个对外部类的引用：  
	
	class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
	}  

使用和Java中的内部类也很像：  

	val demo = Outer().Inner().foo() // == 1  

### 6. 枚举类和密封类  
#### 6.1 枚举类  
枚举类的最基本的用法是实现类型安全的枚举：  

	enum class Direction {
    NORTH, SOUTH, WEST, EAST
	}  
> 这里的枚举类的声明和Java不一样，在class前添加enum标识  

**每个枚举常量都是一个对象(可以用枚举类实现单例模式)**  

#### 6.2 初始化  
因为每一个枚举都是枚举类的实例，所以他们可以是这样初始化过的：  

	enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
	}  

#### 6.3 匿名类  
枚举常量也可以声明自己的匿名类：  

	enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
	}  

#### 6.4 使用枚举常量  
就像在 Java 中一样，Kotlin 中的枚举类也有合成方法允许列出定义的枚举常量以及通过名称获取枚举常量。这些方法的签名如下（假设枚举类的名称是 EnumClass）：  

	EnumClass.valueOf(value: String): EnumClass
	EnumClass.values(): Array<EnumClass>  

如果指定的名称与类中定义的任何枚举常量均不匹配，valueOf() 方法将抛出 IllegalArgumentException 异常。

自 Kotlin 1.1 起，可以使用 enumValues<T>() 和 enumValueOf<T>() 函数以泛型的方式访问枚举类中的常量 ：  

	enum class RGB { RED, GREEN, BLUE }

	inline fun <reified T : Enum<T>> printAllValues() {
    	print(enumValues<T>().joinToString { it.name })
	}

	printAllValues<RGB>() // 输出 RED, GREEN, BLUE  

每个枚举常量都具有在枚举类声明中获取其名称和位置的属性：  

	val name: String
	val ordinal: Int  

### 7. 对象和对象表达式  
#### 7.1 对象表达式  
实际上就是Java中的匿名内部类的实现，直接看例子：  

	open class objectTest{
    open fun myprintln():Unit{
        println("this is parent")
    }
	}

	class objectSub:objectTest(){
    fun subprintln(sub:objectTest){
        sub.myprintln()
    }
	}

	fun main(args:Array<String>){
    objectSub().subprintln(object : objectTest(){
        override fun myprintln() {
            println("this is object")
        }
    })
	}  
	/*output:  
	this is object
	
在kotlin中实现类似于匿名内部类的方式是：**object:ClassConstructor{override 实现}**  
这里叫做对象表达式，可以理解为**object:ClassConstructor{override 实现}是一个表达式，返回的是一个匿名的内部类对象。**  
任何时候，如果我们只需要*一个对象而已*，并不需要特殊的超类型，那么我们可以简单的写：  

	fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
	}  
>个人觉得这么干好像不怎么实用。  

请注意，匿名对象可以用作只在本地和私有作用域中声明的类型。如果你使用匿名对象作为公有函数的返回类型或者用作公有属性的类型，**那么该函数或属性的实际类型会是匿名对象声明的超类型，如果你没有声明任何超类型，就会是 Any。在匿名对象中添加的成员将无法访问。**  

	class C {
    // 私有函数，所以其返回类型是匿名对象类型
    private fun foo() = object {
        val x: String = "x"
    }

    // 公有函数，所以其返回类型是 Any
    fun publicFoo() = object {
        val x: String = "x"
    }

    fun bar() {
        val x1 = foo().x        // 没问题
        val x2 = publicFoo().x  // 错误：未能解析的引用“x”
    }
	}  

*问题：为什么这么设计？public方法会有什么风险吗？*  

#### 7.2 对象声明  
单例模式在一些场景中很有用， 而 Kotlin（继 Scala 之后）使单例声明变得很容易：  

	object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ……
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ……
	}  
这称为对象声明。并且它总是在 object 关键字后跟一个名称。 就像变量声明一样，对象声明不是一个表达式，不能用在赋值语句的右边。

对象声明的初始化过程是**线程安全**的。

如需引用该对象，我们直接使用其名称即可：  

	DataProviderManager.registerDataProvider(……)  

*问题:对象声明如何用于单例模式？*  
>**解释：通过"name.method"形式调用对象的方法，之所以能够实现单例模式，是因为object对象声明这种形式只会保证只被初始化一次，以后如果该实例没有消失，那么都是通过该实例调用其object定义的方法的。**

#### 7.3 伴生对象  
类内部的对象声明可以用 **companion** 关键字标记：  

	class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
    }  

**该伴生对象的成员可通过只使用类名作为限定符来调用(可通过此方式实现单例模式)**：  

	class objectSub private constructor(var name :String):objectTest(){
    	fun subprintln(sub:objectTest){
        	sub.myprintln()
    }

    	companion object {
        	fun getInstance(name:String):objectSub{
            	return objectSub(name)
        	}
    	}
	}

	fun main(args:Array<String>){
    	println(objectSub.getInstance("su").name)
	}  

可以省略伴生对象的名称，在这种情况下将使用名称 **Companion**：  

	class MyClass {
    companion object {
    }
	}

	val x = MyClass.Companion  

请注意，**即使伴生对象的成员看起来像其他语言的静态成员，在运行时他们仍然是真实对象的实例成员**，*对象声明实现单例模式更加纯粹，不需要依附于一个类。*  

#### 7.5 三者之间的差别  
对象表达式和对象声明之间有一个重要的语义差别：  

* 对象表达式是在使用他们的地方**立即**执行（及初始化）的；
* 对象声明是在第一次被访问到时**延迟**初始化的；  
* 伴生对象的初始化是在相应的类被加载（解析）时，与 Java 静态初始化器的语义相匹配。  

## 五、函数  
在kotlin中函数用**fun**表示，函数的不同的用法和定义和Java中的方法定义差不多，这里就只介绍一下kotlin对Java的方法的定义的扩展：  

* 默认参数：减少方法重载数量
* 表达式函数体：声明函数更加简单

### 5.1 默认参数  
函数的参数可以有默认值，当在使用的时候，如果这些有默认值的参数不传值的话，就按照默认值来，这可以**减少重载数量**：  

	fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) {
	……
	}  

**默认值通过类型后面的 = 及给出的值来定义。**  
覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须**从签名中省略默认参数值**：  

	open class A {
    	open fun foo(i: Int = 10) { …… }
	}

	class B : A() {
    	override fun foo(i: Int) { …… }  // 不能有默认值
	}  

如果一个默认参数在一个无默认值的参数之前，那么该默认值**只能通过使用命名参数调用该函数来使用**：  

	fun foo(bar: Int = 0, baz: Int) { /* …… */ }

	foo(baz = 1) // 使用默认值 bar = 0  

>命名参数就是在含有大量的默认参数的时候，如果单独的为其中一个默认的参数传值，很大可能会出现该参数前有很多同类型的默认参数，导致如果不声明传的是哪一个默认参数的话，会无法识别；所以索性就直接规定，在一个参数列表中，当一个参数前有默认参数时，不管传值的参数是什么类型，都得给我声明参数名。  

### 5.2 可变参数  
如果参数列表中的一个参数用了**vararg**修饰，那么该参数的数量是可变的。  
我们知道，在Java中的可变参数是可以传一个数组作为参数的，但是kotlin中就不行；得用**星号操作符**将可变数量参数以命名的形式传入：  

	fun myprintln(vararg strings:String){
    	println(strings[0])
	}

	fun main(args:Array<String>){
    	val test= arrayOf("h","e","l","l","o");
    	myprintln(strings = *test)
	}  
	//可以看到，在函数中我们可以把参数当做一个数组来用，但是我们传参数的时候却不能传入一个数组  

**另外，在kotlin中是可以有局部函数的**   

### 5.3 Lambda表达式和匿名函数  
kotlin中的高阶函数指的是，可以接受一个函数作为参数或者返回值是一个函数的函数。  
#### 5.3.1 函数类型  
首先函数也有类型，在kotlin中在一个高阶函数的参数列表中去声明一个函数作为参数的时候，也是要有类型的，函数的类型由参数类型列表用*->*链接返回值组成：  

	fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean)  
	比如这里less是一个接受两个T类型的参数，并且返回值是一个T的函数  

#### 5.3.2 Lambda表达式  
Lambda 表达式的完整语法形式，即函数类型的字面值如下：  

	val sum = { x: Int, y: Int -> x + y }  

**lambda 表达式总是括在花括号中**， 完整语法形式的参数声明放在花括号内，并有可选的类型标注， 函数体跟在一个 -> 符号之后。如果推断出的该 lambda 的返回类型不是 Unit，那么该 lambda 主体中的最后一个（或可能是单个）表达式会视为返回值。  

> **Lambda实际上就是存储了一小段代码，随时可以调用者端代码，将一个Lambda赋值给一个变量不过是为这段代码添加一个标识，通过标识来调用这段代码，我们也可以直接调用Lambda：**
> {x : Int,y : Int -> print("x + y =${(x + y)}")}( 1, 2)    这段代码完全可以执行：x + y = 3


如果我们把所有可选标注都留下，看起来如下：  

	val sum: (Int, Int) -> Int = { x, y -> x + y }  

>注意：**Lambda表达式中没有return语句，默认按最后一行的结果作为返回值，如果使用了return语句，则效果是从调用Lambda的外层函数处return**    

同样，如果lambda表达式参数的类型可以被推导出来，就不用显式地指定lambda参数的类型。

	people.maxBy { p : Persion -> p.age }
	people.maxBy { p -> p.age}


**在Lambda表达式中也可以使用表达式之前就被声明的变量**

#### 成员引用  
当我们想要写成Lambda表达式的代码已经是类的成员的时候该怎么办？再在Lambda表达式中写一次？这和我们减少重复代码的初衷有些冲突，这时候**成员引用**就起了作用，**成员引用的作用就是：将函数转化成一个值，然后传递它**

	

#### 5.3.3 匿名函数  
上面提供的 lambda 表达式语法缺少的一个东西是指定函数的返回类型的能力。在大多数情况下，这是不必要的。因为返回类型可以自动推断出来。然而，如果确实需要显式指定，可以使用另一种语法： **匿名函数 **。  

	fun(x: Int, y: Int): Int = x + y  

匿名函数看起来非常像一个常规函数声明，除了其名称省略了。其函数体可以是表达式（如上所示）或代码块：  

	fun(x: Int, y: Int): Int {
    	return x + y
	}  

>**Lambda表达式与匿名函数之间的另一个区别是非局部返回的行为。一个不带标签的 return 语句总是在用 fun 关键字声明的函数中返回。这意味着 lambda 表达式中的 return 将从包含它的函数返回，而匿名函数中的 return 将从匿名函数自身返回。**      

#### 5.3.4 with函数   
当调用一个对象的多个方法的时候，我们需要不停的写这个对象的名称，调用几次方法就要写几次这个名称；**with函数**的作用就是使你只需要写一次就可以：  

	fun test() : String{
		val result = StringBuilder()
		for (letter in 'A' .. 'Z') {
			result.append(letter)
		}
		result.append("\n Now I know the alphabet!")
		return result.toString()
	}

	//with 函数
	fun test() : String {
		val result = StringBuilder()
		return with(stringBuilder) {
			for (letter in 'A' .. 'Z') {
				this.append(letter)
			}
			append("\n Now I know the alphabet!")
			this.toString()
		}
	}

> **with函数**的参数实际上是两个，第一个是一个对象，第二个实际上是**Lambda表达式**，如果在Lambda表达式中想要调用第一个参数的方法，用**this关键字或者省略this都可以**；**with函数的返回值就是lambda表达式的返回值**

#### 5.3.5 apply函数  
apply函数的作用和with函数几乎是一样的，区别就是：对象是主动的调用apply函数，而不是像with函数一样作为参数传进去；还有就是，apply函数最后返回的一定是调用apply的对象本身，而不是lambda表达式的结果：  

	fun test() :String {
		return StringBuilder().apply{
			for (letter in 'A' .. 'Z') {
				this.append(letter)
			}
			append("\n Now I know the alphabet!")
		}.toString()
	}

### 5.4 表达式函数体  
> **表达式和语句：表达式和语句的区别就在于，表达式是有值的，而语句是没有值的！！**在kotlin的控制流中，只有循环是语句，if和when条件都是表达式；而Kotlin中定义一个函数的时候，如果函数体只是一个表达式的话，可以直接去掉花括号和return语句，以等号相连  

	fun getMax(a:Int,b:Int) = if (a > b) a else b
	fun sayHello() = println ("hello world")    //函数是有返回值的，所以函数调用一定是个表达式  

### 5.5 局部函数  
当一个函数中有一段代码需要使用多次的时候，你当然可以将这段代码复制多次，但是Kotlin中的局部函数则可以更优雅的实现，**函数的一大优点就是代码的复用**，所以局部函数的作用就是这个，**局部函数定义在函数体里面，外面的对象是无法访问的。**

## 六、其他  
### 6.1 解构  
之前的学习中我们了解过componentN()这个函数，实际上这就是每一个类中默认的解构函数。  
首先我们来看一看解构的使用：  

	data class Persion(val name:String,val age:Int)  
	...  
	val(mName,mAge)=Persion("su",20)  
	println("name is $mName,age is $mAge")  
	
	output:
		name is su,age is 20  

**没错，一个解构声明可以创建多个变量。**  
一个解构声明会被编译成：  
	
	val mName = person.component1()
	val mAge = person.component2()  

其中的 *component1() 和 component2()* 函数是在 Kotlin 中广泛使用的 *约定原则* 的另一个例子。 （参见像 + 和 *、for-循环等操作符）。 任何表达式都可以出现在解构声明的右侧，只要可以对它调用所需数量的 component 函数即可。 当然，可以有 component3() 和 component4() 等等。

请注意，**componentN()** 函数需要用 **operator 关键字标记，以允许在解构声明中使用它们**  
  
解构对中的各个变量会被赋值为某一个类的构造器中声明的变量的顺序，可是我们如果想跳过第一个变量，构造一个解构对，只得到第二第三个变量呢？--**下划线用于未使用的变量**  

如果在解构声明中你不需要某个变量，那么可以用下划线取代其名称：  

	val (_,mAge)=persion  
对于以这种方式跳过的组件，不会调用相应的 componentN() 操作符函数。  

### 6.2 类型的检查与转换"is"和"as"  
#### 6.2.1 智能转换  
很多情况下，在kotlin中不需要使用显示转换操作符，因为编译器**跟踪不可变值的is检查和显式转换**，并在需要时**自动插入(安全的)转换**：  

	open class Parent(){
	}

	class Child():Parent(){
    	fun childPrintln(){
        	println("this is child println")
    	}
	}

	fun main(args:Array<String>){
    	var child:Parent
    	child=Child()
    	println(child.toString())
    	if(child is Child){
        	println(child.toString())
        	child.childPrintln()
    	}
	}  
	/*output
	Child@266474c2
	this is child println  
第一条打印的结果与Java并无区别，类似于Java中的多态的概念，运行时类型是Child，所以打印显示是一个Child的实例；但是第二条的打印就与Java有细微的不同了，这里只是在if语句中判断了它是Child类型的实例，但是我们在声明的时候声明的是Parent类型，这里没有像Java那样，需要显示转换为Child类型才能调用Child类的独有方法----**这就是Kotlin中的智能转换**  
这种智能转换甚至可以出现在**&&和||**的右侧：  

	// `||` 右侧的 x 自动转换为字符串
    if (x !is String || x.length == 0) return

    // `&&` 右侧的 x 自动转换为字符串
    if (x is String && x.length > 0) {
        print(x.length) // x 自动转换为字符串
    }  

注意，上面说过编译器**跟踪不可变值的is检查和显式转换**，并在需要时**自动插入(安全的)转换**，当编译器不能保证变量在检查和使用之间不可改变时，智能转换不能用。 更具体地，智能转换能否适用根据以下规则：  

*   val 局部变量——总是可以，局部委托属性除外  
*   val 属性——如果属性是 private 或 internal，或者该检查在声明属性的同一模块中执行。智能转换不适用于 open 的属性或者具有自定义 getter 的属性；  
*   var 局部变量——如果变量在检查和使用之间没有修改、没有在会修改它的 lambda 中捕获、并且不是局部委托属性；  
*   var 属性——决不可能（因为该变量可以随时被其他代码修改）。  

#### 6.2.2 "不安全的"和"安全的"转换操作符  
通常，如果转换是不可能的，转换操作符会抛出一个异常。因此，我们称之为不安全的。 Kotlin 中的不安全转换由中缀操作符 as（参见operator precedence）完成：  

	val x: String = y as String  
请注意，null 不能转换为 String 因该类型不是可空的， 即如果 y 为空，上面的代码会抛出一个异常。 为了匹配 Java 转换语义，我们必须在转换右边有可空类型，就像：  

	val x: String? = y as String?

为了避免抛出异常，可以使用安全转换操作符 as?，它可以在失败时返回 null：  

	val x: String? = y as? String  


请注意，尽管事实上 as? 的右边是一个非空类型的 String，但是其转换的结果是可空的。  

#### 6.2.3 类型擦除  
Kotlin 在编译时确保涉及泛型操作的类型安全性， 而在运行时，泛型类型的实例并无未带有关于它们实际类型参数的信息。例如， List<Foo> 会被擦除为 List<*>。通常，在运行时无法检测一个实例是否属于带有某个类型参数的泛型类型。

为此，编译器会禁止由于类型擦除而无法执行的 is 检测，例如 ints is List<Int> 或者 list is T（类型参数）。当然，你可以对一个实例检测星投影的类型：   

	if (something is List<*>) {
    something.forEach { println(it) } // 这些项的类型都是 `Any?`
	}  

类似地，当已经让一个实例的类型参数（在编译期）静态检测， 就可以对涉及非泛型部分做 is 检测或者类型转换。请注意， 在这种情况下，会省略尖括号：  

	fun handleStrings(list: List<String>) {
    if (list is ArrayList) {
        // `list` 会智能转换为 `ArrayList<String>`
    }
	}  

### 6.3 操作符重载  
在Java中，除了系统为我们提供的String的"+"操作符被重载了之外，Java是不让开发者自己去自定义操作符重载的，而Kotlin则放开了手脚，允许开发者自定义操作符重载：  
#### 6.3.1 一元操作  
**一元前缀操作符**  

* +a:译为**a.unaryPlus()**  
* -a:译为**a.unaryMinus()**
* !a:译为**a.not()**  

这个表格是说，当编译器处理例如表达式+a时，它执行以下步骤：  

*   确定 a 的类型，令其为 T
*   为接收者 T 查找一个带有 operator 修饰符的无参函数 unaryPlus（），即成员函数或扩展函数；
*   如果函数不存在或不明确，则导致编译错误
*   如果函数存在且其返回类型为 R，那就表达式 +a 具有类型 R；  

看例子：

	class People(var name:String){
    	operator fun unaryMinus(){
        	name="-$name"
    	}
    
	}

	fun main(args:Array<String>){
    	var p=People("su")
    	-p
    	println(p.name)
    	println(p.name)
	}
	/*output
	-su

**递增与递减**  

* ++a:译为**a.inc()**  
* --a:译为**a.dec()**

**inc() 和 dec() 函数必须返回一个值，它用于赋值给使用 ++ 或 -- 操作的变量。它们不应该改变在其上调用 inc() 或 dec() 的对象。**  

编译器执行以下步骤来解析后缀形式的操作符，例如 a++：  
* 确定 a 的类型，令其为 T；
* 查找一个适用于类型为 T 的接收者的、带有 operator 修饰符的无参数函数 inc()；
* 检查函数的返回类型是 T 的子类型

计算表达式的步骤是：  

* 把 a 的初始值存储到临时存储 a0 中；
* 把 a.inc() 结果赋值给 a；
* 把 a0 作为表达式的结果返回。

对于前缀形式 ++a 和 --a 以相同方式解析，其步骤是：

* 把 a.inc() 结果赋值给 a；
* 把 a 的新值作为表达式结果返回。  

>这里的"前加"和"后加"的区别就如同Int等的前加和后加的区别一样，是在过程中有区别，如果都执行完了，再去看结果，二者都是一样的，看例子：  

	class People(var name:String){
    	operator fun inc():People{
        	var peo=People("guang")
        	return peo
    	}
	}  
	fun main(args:Array<String>){
    	var p=People("su")
    	println(p++.name)
    	println(p.name)
	}  
	/*output
	su  

>可以看到，后加执行后p的name属性并没有改变，如果是前加的话，输出的结果是改变后的结果。  

Kotlin中能够重载的操作符有很多，我就不一一列举了，请查看官方文档。  

### 6.4 空安全  
Kotlin 的类型系统旨在消除来自代码空引用的危险，在 Kotlin 中，类型系统区分一个引用可以容纳 null （可空引用）还是不能容纳（非空引用）。  

	var a: String = "abc"
	a = null // 编译错误  

如果要允许为空，我们可以声明一个变量为可空字符串，写作 String?：  

	var b: String? = "abc"
	b = null // ok  

可是入过我没有指定一个变量是空安全但是还是需要进行一些操作怎么办？  
#### 在条件中检查null  
编译器会跟踪所执行检查的信息，并允许你在 if 内部调用 length。 同时，也支持更复杂（更智能）的条件：  

	if (b != null && b.length > 0) {
    	print("String of length ${b.length}")
	} else {
    	print("Empty string")
	}  
#### 安全的调用  
你的第二个选择是安全调用操作符，写作 **?.**：  

	b?.length  

如果 b 非空，就返回 b.length，否则返回 null；如果用该操作符进行链式的调用，如果有一个环节为null，该表达式整个返回null  

#### Elvis操作符  
当我们有一个可空的引用 r 时，我们可以说“如果 r 非空，我使用它；否则使用某个非空的值 x”：  

	val l: Int = if (b != null) b.length else -1  

除了完整的 if-表达式，这还可以通过 Elvis 操作符表达，写作 **?:**：  

	fun test (s : String?) {
		val ss = s ?: "test"
	}	  

如果 ?: 左侧表达式非空，elvis 操作符就返回其左侧表达式，否则返回右侧表达式。 请注意，当且仅当左侧为空时，才会对右侧表达式求值。  

> 该运算符和非空调用的区别是，一个是**调用的操作**，一个并不涉及到调用

#### ！！操作符  
第三种选择是为 NPE 爱好者准备的：非空断言运算符（!!）将任何值转换为非空类型，**若该值为空则抛出异常**。我们可以写 b!! ，这会返回一个非空的 b 值 （例如：在我们例子中的 String）或者如果 b 为空，就会抛出一个 NPE 异常：  

	val l = b!!.length  
	fun test ( ss : String?) {
		val s =ss!!        //如果ss为空，直接抛出空指针异常
	}

因此，如果你想要一个 NPE，你可以得到它，但是你必须显式要求它，否则它不会不期而至。

####  as？操作符
该运算符和类型转换运算符的区别猜也能猜出来，如果真的是左侧表达式的子类，那么就执行类型转换，否则返回null  

	var child : Child = Child()
	child as? Parent()

#### let 函数  
**let函数和安全调用运算符一起使用**，let函数将调用者作为参数传递给一个lambda表达式。  
let函数和安全调用符一起使用的作用就是，如果不为空的话，就可以通过lambda表达式对该调用者做出各种处理，虽然我们可以先执行一个if判断，然后进行处理，但是**let函数和安全调用符结合使用**的方式显然更加简单：  

	val test : String?
	test ?. let { test -> test.length()
		test.toString()
	}

	//等价写法
	if ( test ) {
		test.length()
		test.toString()
	}    

#### 平台类型  
Kotlin中的类型要么是可空的，要么是非空的，但是如果是来自Java代码的变量呢？因为在Java中可不会管这件事儿，所以Kotlin中有一种**平台类型**(如String！)，表示这种变量的可空性未知，我们可以对这种变量进行任何操作，就像Java中那样，同样，也如同Java中那样可能会遇到空指针异常。**Kotlin中无法声明一个平台类型变量，只能由Java代码引入。**

## 七、集合与数组  
Kotlin中的结合哈Java中最大的一个区别就是，将**访问集合数据和修改集合数据分开**了，在Kotlin中集合分为**只读(Collection)的和可变的(MutableCollection)**,其中，Collection中定义了集合的访问方法，而MutableCollection继承Collection后又定义了一些修改数据的方法。   

	只读的集合创建函数
	listOf，mapOf，mapOf
	
	可变的集合创建函数
	mutableListOf，arrayListOf，mutableSetOf，hashSetOf，linkedSetOf，sortedSetOf，mutableMapOf，hashMapOf，linkedMapOf，sortedMapOf  

Kotlin代码之间的交互是没有问题的，问题是如果调用Java代码呢？或者Java代码调用Kotlin代码呢？集合的只读性和可变性怎么保证？  

事实上，这就没有办法保证了，如果Kotlin代码引用Java代码的话，其中的集合到底是只读的还是可变的，我们就只能根据情景自己去选择实现了(而且其泛型参数都为平台类型)；另一方面，在Java中引用Kotlin代码的话，如果是一个只读的集合，Java代码依然能够去修改它。

