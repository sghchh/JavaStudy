# Java反射  
## 一、通过反射查看类信息  
### 1、获得Class对象  
在Java程序中获得Class对象有三种方法：  

* Class.forName(String):这里传入的一定是类的全限定名  
* 某个类.class:这种方法相对于第一个更加简单快捷，性能上也要好一些  
* 某个类的对象xxx.getClass()  

### 2、通过Class对象查看类的信息  
#### 2.1 查看构造器  
查看构造器根据名字可以分为查看public构造器和查看所有修饰符的构造器；当然，也可以分为查看指定构造器和查看所有构造器  

* Constructor<T> getConstructor(Class<T>... parameterTypes):返回Class对应的类的带指定形参列表的(这里的形参列表只关注形参类型)、public构造器  
* Constructor<?>[] getConstructors():返回Class对应的类的所有public构造器   
* Constructor<T> getDeclaredConstructor(Class<T>... parameterTypes):返回Class对应的类的带指定形参列表的(这里的形参列表只关注形参类型)的构造器，与访问修饰符无关  
* Constructor<?>[] getDeclaredConstructors():返回Class对应的类的所有构造器  

#### 2.2 查看方法  
查看类的方法和查看构造器差不多，也是由一堆Class对象的方法来获取。  

* Method getMethod(String name,Class<?>... parameterTypes):返回此对象对应类的、带指定形参列表的public方法  
* Method[] getMethods():返回此对象对应类所有的public方法   
* Method getDeclaredMethod(String name,Class<?>... parameterTypes):返回此对象对应类的、带指定形参列表的方法(与访问修饰符无关)  
* Method[] getDeclaredMethod():返回此对象对应类的所有方法   

#### 2.3 查看属性  
看了上面两个后，不用猜也能知道这四个方法分别是：  

* Field getField(String name)  
* Field[] getFields()
* Field getDeclaredField(String name)
* Field[] getDeclaredFields()  

类似的还有查看**注释**  

#### 2.4 其他  

* Class<?>[] getDeclaredClasses():返回该Class对象对应类里面包含的所有内部类  
* Class<?> getDeclaringClass():返回外部类  
* Class<?>[] getInterfaces():返回实现的所有的接口 
* Class<?> getSuperclass():返回父类  
* String getName():返回类的全限定名
* String getSimpleName():返回类的简称
* Package getPackage():返回此类的包  

### 3、操作对象  
获取了Class对象之后，我们还可以使用这个对象来完成穿件该类的实例、调用方法、访问属性等常规操作  
#### 3.1 创建对象  
使用反射创建对象有两种方式：  
**
* 直接使用Class对象的**newInstance()方法**，不过这种方式是通过调用默认的构造器来创建实例的，所以，**必须提供默认构造器**才行  
* 先获取Constructor对象，再通过Constructor对象的newInstance()方法创建对象，这种方法的有点显而易见，可以调用指定的构造器初始化  

#### 3.2 调用方法  
调用方法是先获取**Method对象**，然后通过Method对象的**invoke(Object obj,Object... args)方法**来实现的，第一个参数为该类的实例，其他的参数是该方法的实参  

#### 3.3 访问属性的值  
访问或者修改属性的值首先要获取Field对象，然后调用Field的**getXxx(Object obj)**方法来获取该属性的值，**setXxx(Object obj,Xxx val)**方法来更新值(参数中的obj为该类的实例，Xxx可选为8个基本类型，如果是**引用类型，不用加Xxx**)  

## 二、反射的应用  
### 1. JDK动态代理  
反射最典型的应用就是JDK中的动态代理了  

java的java.lang.reflect包提供了一个**Proxy类和InvocationHandler接口**，**前者用来创建动态代理的类或者对象，后者用来实现动态代理类**  

Proxy提供了如下方法来创建一个动态代理类或代理类的实例：  

* static Class<?> getProxyClass(ClassLoader loader,Class<?>...interfaces):第一个参数是一个类加载器，而第二个参数是表示该代理类将实现interfaces中指定是接口  
* static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler handler):该方法直接返回一个实现了interfaces接口的代理类，发现该方法和上面的方法相比只是多了一个参数  

实例：  

	//第一种方法  
	InvocationHandler handler=new MyInvocationHandler();
	Class proxyClass=Proxy.getProxyClass(Foo.class.getClassLoader(),new Class[]{Foo.class});
	Constructor ctor=proxyClass.getConstructor(new Class[]{InvocationHandler.class});
	Foo f=(Foo)ctor.newInstance(new Object[]{handler});  

	//第二种方法  
	InvocationHandler handler=new MyInvocationHandler();
	Foo f=Proxy.newProxyInstance(Foo.class.getClassLoader(),new Class[]{Foo.class},handler);  

可以发现第二种方法就是将第一种方法的所有步骤合并了而已(具体的区别此处不做解释)，所以我们还是推荐直接使用第二种方式。  

所以，怎么也绕不开**InvocationHandler接口**，我们就来看看该接口：  

	public interface InvocationHandler {
	    public Object invoke(Object proxy, Method method, Object[] args)
	        throws Throwable;
	}  

里面只定义了一个方法，invoke  

下面来解释这个接口的方法，我们上面创建动态代理对象或者类的时候都有一个**interfaces参数表示我们最后得到的动态代理类实现了该参数指定的所有的接口**，这里的意思是我们通过动态代理类调用所有接口中定义的方法的时候，调用的都是**InvocationHandler中的invoke方法**，而所有的信息都保存在了invoke的参数中，**proxy就代表动态代理对象，method则代表了interfaces的所有的方法中被调用的那个，args则是调用接口的方法时传入的参数**。  

	public class MyInvocationHandler implements InvocationHandler {
	    public Object invoke(Object proxy, Method method, Object[] args)
	        System.out.println("hello world");
			return new Object();
	} 

	public interface Foo{
		public void hello();
		public void say();
	}

	InvocationHandler handler=new MyInvocationHandler();
	Foo f=Proxy.newProxyInstance(Foo.class.getClassLoader(),new Class[]{Foo.class},handler);
	f.say();
	f.hello();  

上面会输出两个hello world，当然你可以在invoke中针对不同的方法输出不同的语句。  

##### 动态代理在AOP中的应用  
Java Spring中的AOP都是基于动态代理实现的。**将客户请求的业务需求交给真正的对象来实现，而在invoke方法中加入一些其他的业务之外的操作**  
