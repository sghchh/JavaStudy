#Retrofit注释  
##1.方法标签  
常用的方法有POST，GET方法  
有关http的基础知识请见：  
> http://blog.csdn.net/itachi85/article/details/50982995  

* GET 对应GET方法  
* POST 对应POST方法  
注意：使用Retrofit时方法标签后面的括号里的字符窜不能为空的（这里的不为空不是指null），最次也是一个("/");  

		@GET("app/web/index.php")
    	Observable<Forumlist> getForemlistEntity(@Query("r") String r);
		以上的才是正确的，而下面的就不行:  
		@GET("")  
		....  
##2.修饰方法  
###2.1 Headers  
用于添加多个请求头，当请求头重名的时候，不会相互覆盖  

		@Headers("Cache-Control:max-age=640000)  
		@GET("/")  
		....  
###2.2 FormUrlEncoded  
准确的说这个是修饰Field和FieldMap的，使用该注解表示请求正文将使用表单网址编码。字段应该声明为参数，并用Field或者FieldMap注释。  

		@FormUrlEncoded  
		@GET("/")  
		...  
###2.3 Multipart  
表示请求体是多部分的，每一个部分作为一个参数，而且用Part注解声明  
##3.修饰方法的形参  
###3.1 用于拼接URL  
####3.1.1 Query和QueryMap  
用于添加查询参数，拼接在baseUrl以及方法后面的括号内容的后面：  
>http://image.baidu.com/search/detail?ct=503316480&z=0&tn=baiduimagedetail&ipn=d&cl=2&cm=1&sc=0&lm=-1&ie=gbk&pn=0&rn=1&di=20648774080&ln=30&word=%CD%BC%C6%AC&os=2526534960,1897446856&cs=3432487329,2901563519&objurl=http%3A%2F%2Fpic62.nipic.com%2Ffile%2F20150319%2F12632424_132215178296_2.jpg&bdtype=0&simid=4048659111,430516437&pi=0&adpicid=0&timingneed=0&spn=0&is=0,0&fr=ala&ala=1&alatpl=others&pos=1  

比如这个URL，问号(?)后面的（ct,z,tn等）的拼接就是通过Query这个注释来完成的。（如果传入的是一个数组，则所有数组的非空item的键都是统一的，他们都拼接都URL后，如name=zhang&name=li&name=liu)  
		
		@GET("/")
		Observable<Body> getBody(@Query("name") String name);  
而QueryMap则是使用Map的形式传入参数  
####3.1.2 Field和FieldMap  
首先这两个的关系和上面的Query与QueryMap之间的关系是一样的。**注：Field注释的值表现在请求体中，而Query注释的值表现在URL中，因为请求体GET方法中是不能存在的，所以这个注释GET方法不能出现，而POST方法中可以出现，使用这个注释的时候必须要有@FormUrlEncoded这个注解**。这个注释的使用形式和Query一样  
####3.1.3 Body  
这是可以看成多个Field注解，也是只能在POSt方法中使用，如果不想注解太多，可以把想要添加在方法形参中的参数以Bean的形式定义一个对象出来，直接把这个对象作为一个参数，这时候就要用Body注释了  
####3.1.4 Header和HeaderMap  
与Headers不同，这个注解作用于形参，表示添加一个头  
###3.2 替换  
####3.2.1 Path  
Path注解用于替换一个URL中的片段  
如：http://102.10.10.132/api/Comments/1如果最后的那一个1是动态的，也可能是别的数，则可以这么用：  

		@GET("Comments/{id}")  
		Observable<Body> getBody(@Path("id") int id);  
表示@Path注释后的参数用于替换{id}，**被替换的部分必须用花括号多住，名字可以随便起**。