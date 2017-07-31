#Java集合  
集合与数组的区别：1.由于数组在声明的时候数量必须确定，而集合的数据数量是可变的  2.集合可以保存有映射关系的数据  3.数组元素可以是任何类型的数据，而集合的元素只能是对象（实际上是对象的引用），而不能是基本数据类型，但是因为有自动装箱的功能，看起来集合像是可以存放基本数据。  
Java的集合类主要由两个接口派生而出：Collection和Map  
Collection接口的三个子接口：List（有序，元素可重），Queue，Set（无序，元素不可重）  
Map：所有的实现类存放的数据都是具有映射关系  
Collection子接口的实现类的添加元素的方法都是add（），Map实现类的添加元素的方法都是put（）
##Collection接口的方法  

* Iterator iterator():返回一个Iterator对象，用于遍历集合里的元素  
* Object[] toArray():将集合转换成一个数组，所有的集合元素变成对应的数组元素  
这里就介绍这两个方法，其他的添加，删除等基本方法就不在赘述。  
####遍历集合的几种方法  

1. 使用Iterator遍历集合  
Iterator对象被称为迭代器，提供了以下几个方法来遍历集合  
     * boolean hasNext():如果被迭代的集合元素没有被遍历完，则返回true  
     * Object next():返回集合里的下一个元素  
     * void remove():删除集合里上一次next（）方法返回的元素  
     * void forEachRemaining(Consumer action):该方法可以用Lambda表达式遍历集合  
**注意：使用迭代器遍历集合的时候，集合中的元素不能被改变，只有迭代器的remove（）方法才能改变集合元素，直接通过集合的增删改的方法会引发异常**  

			public class Test {
				public static void main(String[] args){
					List list=new ArrayList();
					list.add("Java讲义");
					list.add("Android讲义");
					list.add("疯狂讲义");
					Iterator iterator=list.iterator();
					while(iterator.hasNext())
					{
						System.out.println(iterator.next().toString());
					}
				}

			}
###Java8新增的Predicate操作集合  
Java8位Collection集合新增了一个removeIf（Predicate filter）方法，该方法会批量删除符合filter条件的集合元素。该方法需要有一个Predicate（谓词）对象做为参数，Predicate也是函数式接口，因此可以用Lambda表达式做为参数。  
###Java8新增的Stream操作集合  
正是因为上面提到的Predicate批量过滤的方便的功能，所以才引入了Stream操作集合。  
新增的XxxStream中的Xxx代表的是基本数据类型Int，Long等，并且为每一个Stream提供了Builder（例如IntStream.Builder,LongStream.Builder）来创建对应的流。使用Stream的步骤如下：  

* 使用Stream或者XxxStream的builder（）静态方法创建该Stream对应的Builder  
* 重复调用Builder的add（）方法添加元素  
* 调用Builder的build（）方法获取对应的Stream  
* 调用Stream的聚集方法来进行想要的操作  
	
		public class Test {
			public static void main(String[] args){
				//创建Builder
				IntStream.Builder in=IntStream.builder();
				//调用add方法
				in.add(100)
				.add(0)
				.add(9)
				.add(2);
				//通过build（）方法获取Stream
				IntStream ins=in.build();
				//进行聚集操作
				System.out.println(ins.max().getAsInt());
				}

		}
		将输出100；
聚集操作有很多方法，具体请查看相关资料  
Java8允许集合转化成Stream来进行聚集操作，首先通过Collection接口的stream（）方法将集合转化成Stream后，就可以进行聚集操作了。


	

