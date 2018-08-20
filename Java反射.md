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

* 直接使用Class对象的**newInstance()方法**，不过这种方式是通过调用默认的构造器来创建实例的，所以，**必须提供默认构造器**才行  
* 先获取Constructor对象，再通过Constructor对象的newInstance()方法创建对象，这种方法的有点显而易见，可以调用指定的构造器初始化  

#### 3.2 调用方法  
调用方法是先获取**Method对象**，然后通过Method对象的**invoke(Object obj,Object... args)方法**来实现的，第一个参数为该类的实例，其他的参数是该方法的实参  

#### 3.3 访问属性的值  
访问或者修改属性的值首先要获取Field对象，然后调用Field的**getXxx(Object obj)**方法来获取该属性的值，**setXxx(Object obj,Xxx val)**方法来更新值(参数中的obj为该类的实例，Xxx可选为8个基本类型，如果是**引用类型，不用加Xxx**)