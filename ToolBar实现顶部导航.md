#ToolBar实现顶部导航  
##1. 认识Toolbar  
###1.1 本质  
查看api文档的话，会发现ToolBar继承自ViewGroup类，因此ToolBar也是一个容器类，同时它的内部还真的包括了五个部分：  

![](http://mmbiz.qpic.cn/mmbiz_png/QFjUqsncFKnkj67TTUp68aRMaELvMeyjklcXXYsa0OXb4llnjZIdv9kCwRfC190q1konvOOqFBtAKXdEOCFqoQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)  
文档说得很清楚，一个Toolbar 从左到右包括了 一个navigation button、一个logo、一个title和subtitle、一个或多个自定义的View和一个 action menu 这5部分。也就是这个ViewGroup 容器里面包含了这五部分内容，对应着一个界面看一下：  

![](http://mmbiz.qpic.cn/mmbiz_png/QFjUqsncFKnkj67TTUp68aRMaELvMeyjPDSDmoibU6HBL2LVTRaFUYlia8Q7TGQCBh613b7k9xtwf2ichzuCibRwrA/640?wx_fmt=png&wxfrom=5&wx_lazy=1)  
这五个部分的内容需要我们设置才能显示，比如那个nacigation button只有在toolbar的xml文档中设置navigationIcon属性给他指定一个drawable等资源才可以看见。  
##2. 使用准备  
由于我们这里是使用ToolBar来构建一个导航栏，所以我们需要将Activity的主题style中的parent属性设置为一个NoActionBar的就行，并且用Java代码写下如下两句：  

	toolbar=(Toolbar)findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);  
##3. 使用  
###3.1 XML文档各属性  
想要使用ToolBar，自然要在布局文件中加入ToolBar这个控件，常规的属性使用：  

* android:id  
* android:layout_width  
* android:layout_height  
* android:background  
* android:theme  
这些属性的使用很常规，这里不再多说，**接下来涉及到ToolBar的属性使用就比较有要求了：必须使用自定义命名空间（xmlns:toolbar="http://schemas.android.com/apk/res-auto",当然这里也可以不命名为toolbar，其他任何名字都行）,本人亲测，以下这些属性真的只有开头是自定义的命名空间的时候才有效：**  

* toolbar:navigationIcon 设置navigation button  
* toolbar:logo 设置logo 图标  
* toolbar:title 设置标题  
* toolbar:titleTextColor 设置标题文字颜色  
* toolbar:subtitle 设置副标题  
* toolbar:subtitleTextColor 设置副标题文字颜色  
* toolbar:titleTextAppearance 设置title text 相关属性，如：字体,颜色，大小等等  
* toolbar:subtitleTextAppearance 设置subtitle text 相关属性，如：字体,颜色，大小等等  
* toolbar:logoDescription logo 描述  
* toolbar:popupTheme  设置点击溢出栏后弹出的效果  

*注：安卓内部自带一个返回的箭头的样式按钮，就是常见的返回的导航。想要使用这个自带的东西，需要添加如下Java代码：*  

	getSupportActionBar().setHomeButtonEnabled(true);
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);  
效果就是下图的黑色箭头：  

![](http://o9w936rbz.bkt.clouddn.com/blog/img/201701/3/snipaste20170118_084810.png)  
*但是：这个是和navigation button是不相容的，如果设置了navigation button，则这两段代码并不会产生效果.*  
###3.2 添加Menu项  
首先先要在res下创建一个menu路径，然后再创建一个menu菜单资源作为添加到ToolBar上的各个项，每一个item项有如下几个属性：  

* android:id 这个属性在注册item的监听事件的时候会根据id来绑定item  
* android:icon 指定显示的图标样式  
* android:title 对这个图标或者说item的描述，主要是在溢出的时候可以选择显示描述，而不是显示图标  
* app:showAsAction 表示图标在各种情况下是否显示  
**注：这里又涉及到了自定义命名空间的问题了，showAsAction必须是自定义命名空间开头才可以,该属性常用的三种值是：**  

* always:总是显示在ToolBar上  
* ifRoom:如果有地方就显示，否则藏在溢出栏  
* never:永远隐藏在溢出栏  
* 
配置好了menu的xml之后就要重写Activity的一个方法来使ToolBar加载这个菜单：  

		@Override
    	public boolean onCreateOptionsMenu(Menu menu) {
        	getMenuInflater().inflate	(R.menu.toolbar,menu);
        	return true;
    	}   

这样就搞定了，请看一个实例：  
![](http://o9w936rbz.bkt.clouddn.com/blog/img/201701/3/snipaste20170118_084810.png)  
右侧那3个竖直排列的点就是溢出栏。  
如果想要更改溢出栏的颜色，则需要在style配置文件中增加如下代码：  

	<item name="android:colorControlNormal">@color/titleColor</item>  
**这里的name后面的字符串不能改。**同样，这个溢出栏的显示图标也是可以自定义的，不过也是需要特殊的操作，看如下代码：  

	<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
        <item name="android:colorControlNormal">@color/titleColor</item>  
		<!--这里是应用自定义溢出栏的风格-->
        <item name="android:actionOverflowButtonStyle">@style/OverflowButtonStyle</item>
    </style>

	<!--这里是自定义风格的配置-->
    <style name="OverflowButtonStyle" parent="android:Widget.ActionButton.Overflow">
        <item name="android:src">@drawable/ic_add_black_24dp</item>
    </style>  
这是style的XML文件，这里面除了自定义风格的xml文件的name属性是自己随便以外，其他的都不能是别的。  
![](http://o9w936rbz.bkt.clouddn.com/blog/img/201701/3/snipaste20170118_085413.png)  
##4. 监听事件  
###4.1 navigation button的监听  
调用setNavigationOnClickListener(new View.OnClickListener);方法  

	toolbar.setNavigationOnClickListener(new View.OnClickListener() 
			{ 
				@Override 
				public void onClick(View v) {
					 Toast.makeText(getApplicationContext(), "点击了返回箭头", Toast.LENGTH_LONG).show(); 
				} 
			});
###4.2 menu的监听  
重写Activity的onOptionsItemSeleted(MenuItem item);方法  

	@Override
	public boolead onOptionsItemSeleted(MenuItem item){  
		switch(item.getItemId()){  
			case R.id.XXX:  
				break;  
		}  
		return true;  
	}  
看到这里就明白了，为什么在定义menu菜单的时候要为每一个item设置id属性了吧。**注意：之前提到的那个左上角的自带的返回的箭头的按钮，这个按钮也可以设置监听事件，它的默认id是android.R.id.home，通常这个按钮的点击事件的处理逻辑就是调用finish()方法**  
##5. ToolBar和SearchView的结合使用  
如果想在action menu上添加一个搜索框（SearchView），则只需要将一个menu的xml中的item增加一个属性app:actionViewClass="android.support.v7.widget.SearchView"即可：  
	
	<item android:id="@+id/search"
        android:title="search"
        app:showAsAction="always"
        app:actionViewClass="android.support.v7.widget.SearchView"
        />  
这段xml的意思是用SearchView来代替一个item，所以这时候不同设定icon属性，本人亲测，即使设置了icon属性，也不会有任何效果。**由于SearchView是在menu中设定的，之前说过menu的初始化加载需要重写方法public boolean onCreateOptionsMenu(Menu menu)，所以SearchView的初始化也是在这个方法中进行**  

	@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.toolbar,menu);
        MenuItem menuItem=menu.findItem(R.id.search);
		//获取搜索框
        SearchView searchView=(SearchView) MenuItemCompat.getActionView(menuItem);
		//设置显示搜索按钮
        searchView.setSubmitButtonEnabled(true);
		//设置监听事件
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
			//当点击搜索按钮的时候的监听
            @Override
            public boolean onQueryTextSubmit(String query) {
                Toast.makeText(MainActivity.this,"点击了搜索",Toast.LENGTH_SHORT).show();
                return true;
            }

			//搜索框中的文本变化的时候触发
            @Override
            public boolean onQueryTextChange(String newText) {
                return false;
            }
        });
        return true;
    }



