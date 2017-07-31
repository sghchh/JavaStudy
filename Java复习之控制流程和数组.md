#Java复习—控制流程和数组  
----  
##条件语句  
条件语句很简单没什么好说的，这里重点强调一下一个细节：switch循环中的细节  
switch循环中作为判断的变量类型可以是byte,short,char,int,以及Java7新增的String类型；*注意：以下几个类型不行*：long,float,double,StringBuffer,StringBuilder  
##循环语句  
循环语句有以下几个组成部分：  

1. init-statement：初始化语句
2. test_expression:循环条件  
3. body_statement:循环体  
4. iteration_statement:迭代语句  
可以知道while和do while循环的初始化语句是在循环体前的；而for循环则比较封闭，所有语句都是在循环的结构里，要么是在圆括号里（当然，for循环中的初始化语句也可以放在for循环之前，这样就扩大了变量的作用域），要么是在花括号里；

#####while以及do while循环  
while循环和do while循环有两个主要的区别：  

1. while循环的循环条件的判断先于循环体的执行，也就是说先判断是否满足循环条件，从而决定是否执行循环；后者则是先执行一次循环体，再判断是否满足循环条件继续执行。  
2. do while最后条件的括号外要加上分号  
代码示例：  

		int count=0;
		do
		{
			System.out.println(count);
			count++;
		}while(count<0);//最后要有个分号
		
		int b=0;
		while(b<0)
		{
			System.out.println(b);
			b++;
		}  
结果是第一个循环会打印出0；第二个不会。  
####for循环  
较简单，最常用  
####循环控制语句  

* break：结束一个循环  
* continue：结束一次循环体的执行，继续下一次  
* return：结束一个方法，循环自然就结束了  
##数组  
数组是一种引用类型的变量  
数组的初始化，初始化有两方面的意思，一是为数组中的元素分配内存空间，二是为数组中的每一个元素赋初值。 
 
* 动态初始化：指定数组的内存空间，初始值由计算机自动赋值  
* 静态初始化：指定初始值，内存空间由计算机决定
