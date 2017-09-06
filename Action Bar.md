#ActionBar  
ActionBar是一种新增的导航栏，新建一个Activity默认是有ActionBar的。去除ActionBar的方法有两种：  

###移除
* 通过xml配置文件静态去除：找到清单文件中引用的theme资源（style资源）将parent改为一个XXX.NOActionBar即可  
* 通过Java代码动态移除ActionBar：  

		ActionBar actionBar=getActionBar();
		actionBar.hide();
		//显示则是  
		actionBar.show();  
**注意：当创建一个ActionBar对象的时候，涉及到导包的过程这时候就有不同的版本了，一个是android.app.actionbar，这个包下创建的ActionBar对应的获得方法是getActionBar（总之亲自试验，获得这个对象后调用hide并没有什么卵用）；另一个是android.support.v7.app.actionbar，这个包下的获取ActionBar的方法是getSupportActionBar（）（经过试验，这个对象调用hide是管用的）**  
###修改ActionBar的图标和标题  
对于主题包含ActionBar的情况下，可以通过在清单文件下的activity的lebal属性来设置文字，logo属性来设置图标（可能会无效）。  
当然如果实在有ActionBar的主题下，也是可以根据需要往ActionBar上添加一些想要的图标的：  
**当Activity启动的时候，系统会调用Activity的onCreateOptionsMenu()方法来取出所有的Action按钮**根据这个方法也可以看出，ActionBar的按钮是由Menu菜单资源定义的，因此，我们需要定义一个菜单资源，在这个菜单资源中就定义了想要增加的图标：  
		
		<?xml version="1.0" encoding="utf-8"?>
		<menu xmlns:android="http://schemas.android.com/apk/res/android"
    		xmlns:app="http://schemas.android.com/apk/res-auto">

    		<item android:id="@+id/action_search"
        		android:icon="@drawable/ic_search_black_24dp"
        		app:actionViewClass="android.support.v7.widget.SearchView"
        		android:title="Search"
        		android:inputType="textCapWords"
        		android:imeOptions="actionSearch"
        		android:orderInCategory="80"
        		app:showAsAction="ifRoom|collapseActionView"/>

    		<item android:id="@+id/action_settings"
        		android:icon="@drawable/ic_apps_black_24dp"
        		android:title="Settings"
        		android:orderInCategory="100"
        		app:showAsAction="always"/>

    		<item android:id="@+id/action_info"
        		android:title="Details"
        		android:icon="@drawable/ic_report_black_24dp"
        		android:orderInCategory="90"
        		app:showAsAction="ifRoom"/>
	    
因此想要将这个菜单文件中的item在ActionBar中显示出来，就需要重写方法

		public boolean onCreateOptionsMenu(Menu menu) {
        	getMenuInflater().inflate(R.menu.main_menu, menu);
        		return super.onCreateOptionsMenu(menu);
    }


####menu的属性  
* id：控件id
* title：标题（必需）
* icon：图标
* showAsAction：控制显示到actionbar还是overflow
* actionViewClass：构建视图所使用的View
* actionProviderClass：基本同上
* menuCategory：同种菜单项的种类。该属性可取4个值：container、system、secondary和alternative。通过menuCategroy属性可以控制菜单项的位置。例如将属性设为system，表示该菜单项是系统菜单，应放在其他种类菜单项的后面。
* orderInCategory：同种类菜单的排列顺序。该属性需要设置一个整数值,越小越靠左  
**理解：ActionBar的图标只是一个外在表现，它只是一个图片而已，虽然也可以通过Activity的onOptionsItemSelected(MenuItem item)方法实现点击事件，但它依旧不是View控件。而actionViewClass这个属性则可以放上去一个View类来作为视图。一个view的点击事件就可以比普通的图标作为视图完成更多的事了。而ActionProvider就更加强大，它的响应事件完全由自己处理（在onPerformDefaultAction（）方法中实现逻辑，而不需要在onCreateOptionsMenu（）实现）**  
#####showAsAction的几个属性，解释一下  
* ifRoom：根据当前的空间大小调整
* never：永远不会显示。只会在溢出列表中显示，而且只显示标题，所以在定义item的时候，最好把标题都带上
* always：无论是否溢出，都会显示
* withText：withText值示意Action bar要显示文本标题。Action bar会尽可能的显示这个标题，但是，如果图标有效并且受到Action bar空间的限制，文本标题有可能显示不全。
* collapseActionView：声明了这个操作视窗应该被折叠到一个按钮中，当用户选择这个按钮时，这个操作视窗展开。否则，这个操作视窗在默认的情况下是可见的，并且即便在用于不适用的时候，也要占据操作栏的有效空间。一般要配合ifRoom一起使用才会有效果。  


