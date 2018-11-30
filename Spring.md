# Spring框架学习  
## 一、JavaBean的配置--依赖注入  
### 1. 自动化装配Bean  
Spring从两个角度来实现自动化装配：  

* 组件扫描：Spring会自动发现应用上下文中所创建的Bean  
* 自动装配：Spring自动满足Bean之间的依赖  

自动化配置并不是一种实现依赖配置的方案，而是能够将**显式配置**降低到最少的方案。  

#### 1.1 创建一个可被发现的Bean    
通过Java的方式使得一个Bean可被发现的方式是使用**@Component注解**，这个简单的注解表明该类会作为组件类，并告知Spring要为这个类创建bean；而且这个注解后面还可以紧跟一个字符串，**作为这个组件的ID**：  

	@Component("test")
	public class Test{}

具有同样效果的还有**@Name**注解，其用法和作用和**@Component**如出一辙  

#### 1.2 设置组件扫描的基础包  
当想要注入一个Bean依赖的时候，我们可以通过组件扫描来自动为我们传入这个Bean对象，而且默认情况下，**这个Bean对象的构造器只会被调用一次，之后的调用都是传入的第一次调用构造器得到的对象**，想要实现组件扫描，只需要使用**@ComponentScan**注解就可以，这个注解后面同样可以跟一个字符串参数，用来表示扫描的范围：  

	@ComponentScan(basePackages={"package1","package2"})
	public class Test{}

	@ComponentScan(baskPackageClasses={Test1.class,Test2.class})
	public class Test{}  

两种方式：**第一种是根据给出的包名，扫描对应的包；第二种是根据给出的类，这些类所在的包就作为扫描的包**  

#### 1.3 自动装配  
上面的只是**声明bean和发现bean**，而一个类需要另一个bean注入的时候，为了让Spring自动帮我们装配，我们需要**@Autowired**注解：  
  
	class Test{
		private Child a;
		
		//构造器注入
		@Autowired
		public Test(Child a){
			this.a=a;
		}

		//属性注入
		@Autowired
		public void setA(Child a){
			this.a=a;
		}
	}  

上面的两种方式的依赖注入都是当创建类的实例或者调用setter方法的时候，Spring会为我们传入bean的实例  

但是，当没有扫描到符合类型的bean的时候，两种方式都会抛出异常，为了不让它抛出异常，可以在注解后面进行配置  

	@Autowired(required=false)   

### 2. 通过Java代码装配bean  
上面的方式有一个很大的缺陷是，对于那些第三方的或者jar文件中的类，由于我们没法为他们添加**@Component**注解，所以我们就没法让Spring为他们创建为bean；另一方面我们也没法使用**@Autowired**注解；这时候**Java代码方式的显式装配**就发挥出了它的作用了。  

通过这种方式进行装配，我们需要引入一个**配置类(使用@Configuration)**，配置类不参与任何业务的实现，只是将**依赖和需要注入的目标进行配对而已**，因此，配置类一定也要**@ComponentScan**注解的。
  
#### 2.1 声明简单的Bean  
要在JavaConfig中声明bean，我们需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加@Bean注解。    

	@Bean  
	public CD sgtPeppers(){
		return new SgtPeppers();
	}  

**@Bean注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean。**  

默认情况下，bean的ID与带有@Bean注解的方法名是一样的。在本例中，bean的名字将会是sgtPeppers。如果你想为其设置成一个不同的名字的话，那么可以重命名该方法，也可以通过name属性指定一个不同的名字：  

	@Bean(name="test")  
	public CD sgtPeppers(){
		return new SgtPeppers();
	}    

#### 2.2 通过配置文件实现注入  
上面我们**成功的将一个类型声明为bean了，在JavaConfig中装配bean的最简单方式就是引用创建bean的方法**  

	public CDPlayer cdPlayer(){
		return new CDPlayer(sgtPeppers());
	}  

看起来，CompactDisc是通过调用sgtPeppers()得到的，但情况并非完全如此。因为sgtPeppers()方法上添加了@Bean注解，Spring将会拦截所有对它的调用，并确保直接返回该方法所创建的bean，而不是每次都对其进行实际的调用。    

### 3. 通过XML装配bean  
#### 3.1 创建XML配置规范  
在Java方式的装配中，声明一个配置类需要使用**@Configuration**注解，而XML中就需要创建一个XML文件，以**<beans元素**为根。

#### 3.2 声明一个简单的bean  
要在XML中声明一个bean，我们需要在上面创建的XML文件中使用**<bean元素**，这类似于Java中的**@Bean注解**。  

	<bean class="soundsystem.SgtPeppers"/>  

**这里的class属性是必须的，而且必须是类的全限定名**，你也可以向java中那样声明一个id，**bean元素中的id属性就是这个作用**。  

	<bean id="compactDisc" class="soundsystem.SgtPeppers"/>  

#### 3.3 通过构造器注入初始化bean  
构造器注入，有两种基本的配置方案可供选择：  

* <constructor-arg>元素
* 使用Spring 3.0所引入的c-命名空间  

拿上面的SgtPeppers bean做例子：  

	<bean id="cdPlayer" class="soundsystem.CDPlayer">
		<constructor-arg ref="compactDisc"/>
	</bean>  

**constructor-arg就是该类的构造器注入，后面的ref说明构造器参数是一个引用，这个引用必须是其他bean的ID；而后面会说到，value则表示给定的值要以字面量(String字面量和基本类型)的形式注入到构造器中**  

而**c-命名空间**更加简便，其语法为：  

	c:paramsName-ref/value="beanID"  
	c:构造器参数名-ref(代表引用)=bean的ID/字面量的值    
	其中构造器参数名也可以使用"_参数的位置"来使用,比如"c:_0"代表构造器第一个参数；而且当参数为字面量的时候，也没有value，直接参数名后面跟id就好  
	c:_title="test"  



上面的例子就可以写成下面这样：

	<bean id="cdPlayer" class="soundsystem.CDPlayer">
		<c:cd-ref="compactDisc"/>
	</bean>      

还有一个就是，可以注入集合，同时集合的元素可以是引用ref，也可以是字面量value：  

	<constructor-arg>
		<list>
			<value> "test"</value>
			<value> "hello"</value>  
		</list>
	</constructor-arg>  

不仅仅是List，还可以是set  

#### 3.4 属性注入  
属性注入和构造器注入是一样的，只不过把**<constructor换成property；把c换成p**  

## 二、高级装配  
### 1. 消除自动化装配的歧义  
注入依赖的最常用情况是一个接口的多个实现类来作为注入的对象，才能典型地体现出松耦合的优点。而如果只是使用@Autowired注解，那么对于一个接口来说，它的实现类有很多个，那么Spring在选择注入的对象的时候就会有歧义  

### 1.1 标注首选Bean  
最简单的一个做法是为某一个子类设置**首选标志@Primary**：  

	//同样也可以用在@Bean的方法
	@Component
	@Primary
	public class Icecream implements Dessert{}  

很容易理解，这个注解的作用是在选择的时候**如果没有其他的区别方式，面临多个选择的时候，有限选择该注解表识的Bean**。  

该注解的使用没有限定，也就是说，如果选项都有该属性，那么还是会报错(个人觉得这个操作很骚，既然是Primary首选项了，干嘛还有多个首选项？)  

### 1.2 限定自动装配  
Primary注解的功能很有限，所以我们推荐使用的是功能强大的限定自动装配方案。使用方式是在需要注入依赖的地方使用**@Qualifier注解来指定要使用哪个Bean**：  

	@Autowired
	@Qualifier("iceCream")
	public void setDessert(Dessert dessert){
		this.dessert=dessert;
	}

这里的Qualifier注解后面的字符串用来标识想要注入的依赖对象，**其值可以是beanID，也可以是在定义Component或者Bean的时候使用Qualifier注解指定的字符串**：  

	@Component
	@Qualifier("cold")
	public class IceCream implements Dessert{}  

同样，对于不同的Component或者Bean，他们的Qualifier注解中的字符串可以相同，为了这样一来就又出现了类似于Primary那样的问题，而Java中又不能使用多个相同的注解(也就不能用多个Qualifier来限定注入的依赖对象)；Spring为我们提供了**自定义限定符注解**来规避这一问题：  

	@Target({ElementType.CONSTRUCTOR,ElementType.FIELD,ElementType.METHOD,ElementType.TYPE})
	@Retention(RetentionPolicy.RUNTIME)  
	@Qualifier
	public @interface Cold{}

这样一来，就可以这么用了：  

	@Component
	@Cold
	public class IceCream implements Dessert{}  

## 三、面向切面编程  
首先引入AOP前我们先来设想一个情景，当我们在实现多个业务的模块的时候，每个模块除了自身核心的功能要实现外，可能都涉及到用户信息的安全问题，这就导致了，每个业务模块儿除了自己的核心功能外，还要去处理安全的问题，这就导致了该模块儿代码的混杂；其次，我们很容易就能想到，我们可以单独将安全的逻辑拿出来，作为依赖注入到不同的模块中去，但是这样就导致了代码的耦合。  

面向切面编程就是为了解决上面的两个问题,在不会导致耦合的情况下，将除了业务核心代码抽离出来，使得业务模块能够专注于业务的实现。

首先解释一下什么是切面：经过上面的解释，一个应用就像是一堆功能模块的组合，而类似于**“安全”**这种需求就像是一个层一样，**将功能模块包裹住**。而**通知(Advice)则代表了切面功能的实现，以及何时触发**；**切点(Pointcut)则代表了切面在何处被触发**；**织入(Weaving)是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中**。  

### 1. 通过注解来实现切面  
我们首先来看一个最简单的例子来了解一下大致的流程：  

	@Aspect
	@Component
	public class Audience{
		@Before("execution(public * concert.Performance.perform(..))")
		public void silenceCellPhones(){}
	}  

**@Aspect**表示该类是一个**切面**，而**@Before表示这个方法是一个Advice，Before表示在切点方法调用前，该方法调用(对应何时、切面功能实现)**；而@Before后面的括号中的参数，我们可以猜到，应该是某一个**Pointcut(切入点)**才对(Spring中过滤出切入点的语句先不讲)。  

可以看到，上面的方法找一个切入点的语句有些麻烦，当我们不光想要使用Before，还要使用After、AfterReturning等的时候，如果都在后面的括号中抄写一遍显得有点傻。所以Spring可以使我们**定义一个切点，当需要多次使用该切点的时候，可以很简单的使用**  

	@Aspect
	@Component
	public class Audience{
		@Pointcut("execution(public * concert.Performance.perform(..))")
		public void test(){}
		
		@Before("test()")
		public void silenceCellPhones(){}
	} 

### 2. 处理通知中的参数  
之前的示例中切面中的**通知方法**都是无参的，毕竟切点方法都是无参的，但是如果切点的方法是有参数的，那么如何处理呢？  

首先看一个示例：  

	@Aspect
	@Component
	public class Audience{
		@Pointcut("execution(public * concert.Performance.perform(int))&&args(number)")
		public void test(int number){}
		
		@Before("test(number)")
		public void silenceCellPhones(int number){}
	} 

这里和上面的不同在于**在声明切点的时候，加上了args(number)**;这么做的效果是，**切点方法中的参数以args指定的名称传递给了通知的参数列表，所以定义通知方法的时候，参数的名称要和args后面的一样才行**。  

### 3. 切面表达式  
上面的示例中都用到了**execution()**这种形式来找到一个切点，这就是切面表达式的形式；切面表达式的组成有三个元素：**Wildcards(通配符)、Operators(运算符)、designators(指示器)**  

#### 3.1 Wildcards  
通配符只用到了三个：  

* "星号"：匹配任意数量的**字符串**  
* "+"：匹配指定类**及其子类**  
* "..":一般用于匹配任意数的子包或参数    

看如下几个例子：  

	execution(public void com.example.as.Test.*Adapter.call())  //表示匹配com.example.as.Test包下任何类名以Adapter结尾的类的call()方法  
	execution(public void com.example.as.Test.Adapter.call(int，..))  //表示匹配com.example.as.Test包下Adapter类的call方法，只不过该call方法的参数必须满足第一个为int型，其他的参数无所谓  

#### 3.2 Operators  
运算符只有**与、或、非**三种，用来**限定匹配**  
	
#### 3.3 Designators  
designators译为指示器，用来表示是**用何种形式来筛选出来切入点的**，最常用的是**execution用来匹配特定的方法**。  

execution表达式很像一个方法描述符：  

	//从上到下依次是：修饰符、返回值类型、包名、方法名(方法参数类型)、抛出的异常的全限定类名。其中带？的表示可以省略。
	execution{
		modifier-pattern?
		ret-type-pattern
		declaring-type-pattern?
		name-pattern(param-pattern)
		throws-pattern?
	}  

具体的例子就看上面的例子

## 四、Spring访问数据库  
目前最火的就是关系型数据库了，Spring中访问关系型数据库有很多方式：**传统的JDBC方式、模板方式、Spring Data方式**。  

而缓存可以在数据已经存在的情况下不去进行数据库操作，从而节省开支和时间。  

#### 总述1  

* DAO(data access object)：数据访问对象  
* Repository  

上面两个动西本质上是一个东西，Spring为了使得访问数据的逻辑不分散到多个组件中去，而推荐将访问数据的逻辑都写到一个组件中去。  

* template：模板，数据访问过程中固定不变的部分  
* callback：回调，数据访问过程中可变的部分    

他们在使用的分工上tamplate更像是一个DAO，它内部定义了数据的访问方法，但是具体的实现却是callback来实现的(为JDBCOptions，其内部定义了访问数据的方法)

配置数据源：  

* 连接池方式：除了下面的JDBC的url、datasource、driverclassname等属性外，还有一些属性用来配置**池**的参数，如initialsize、maxactive等  
* JDBC方式：就是常见的url、datasource、driverclassname、password、username等，和连接池的区别是  
	* 没有池的作用，也就不用配置和池相关的参数  


#### 总述2  

* ORM(object-relational mapping)对象-关系映射数据库想要做的是：将对象的属性映射到数据库的列上，从而自动生成查询和语句(也就是说把对象映射为数据表)；同时还能实现延迟加载、预先抓取、级联等功能  

常用的两种ORM集成方案：  

* Hibernate
* JPA(Java Persistence API)：Java持久化API

使用Hibernate前需要先获得Hibernate Session对象，该对象中定义了对数据的操作，包括如何保存、删除、更新数据的功能。而获取这个Session的标准方式是通过Hibernate SessionFactory来获取，SessionFactory主要负责Session的打开、关闭和管理。而且在SessionFactory中要配置**datasource数据源：datasource属性、映射的数据文件(说白了就是需要将哪些类映射为数据表)：mappingResources属性或者PackagesToScan属性、使用哪种数据库：hibernateProperties属性**













	



	  
	
  





