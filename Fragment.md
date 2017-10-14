#Android Fragment详解  
fragment的生命周期：  
![](http://img.blog.csdn.net/20140719225005356?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

* onAttach(Activity):当Fragment与Activity发生关联的时候调用。  
* onCreateView(LayoutInflater,ViewGroup,Bundle):创建改Fragment的视图  
* onActivityCreated(Bundle):当Activity的onCreate方法返回时调用  
* onDestoryView():当改Fragment的视图被移除时调用  
* onDetach():当改Fragment与Activity关联被取消时调用  
**注：除了onCreateView，其他的方法如果被重写了，必须调用父类对于该方法的实现**
##1. 静态创建fragment  
静态的使用Fragment的方法是将Fragment当做一个普通的view来对待（fragment并没有继承View类，不能通过findViewById方法得到实例）  

1. 定义一个类来继承Fragment类  
2. 在该类中至少实现onCreateView方法，该方法确定了Fragment的布局视图  
3. 定义一个layout资源文件用来搭建这个Fragment的布局（这个布局文件在onCreateView方法中被引用来作为Fragment的视图）  
4. 在Activity的布局文件中添加一个fragment元素，必须要指定一个xml元素为name，这个name的值就是自定义的继承了Fragment的类的包名。  

##2. 动态创建Fragment  
如果我们想实现点击一个button后，在一块预留的区域内创建一个Fragment，静态加载肯定是不行的。这时候就要进行动态的加载一个fragment了。  

1. 定义一个类来继承Fragment类  
2. 在该类中至少实现onCreateView方法，该方法确定了Fragment的布局视图  
3. 定义一个layout资源文件用来搭建这个Fragment的布局（这个布局文件在onCreateView方法中被引用来作为Fragment的视图）
4. 在Activity的布局文件中添加一个layout元素    
5. 在Activity中定义一个该类的实例  
6. 通过FragmentTransatcion（事务）的add(LayoutID,Fragment)来添加一个Fragment，或者，replace（LayoutID，Fragment）方法来将上一步创建的Fragment实例替换第四步添加的layout元素  
7. 调用FragmentTransatcion的commit方法提交  
**注：FragmentTransatcion对象的得到经历了一下过程：FragmentManager fm=getFragmentManager();FragmentTransatcion ft=fn.beginTransatcion();**  
##Fragment与Activity之间的通信  
###1. Activity向Fragment传递数据  
在Activity中创建Bundle数据包，并调用Fragment的setArguments（Bundle）方法即可将Bundle数据包传递给Fragment。   
*可能有小伙伴想，我们可以在Fragment类中把它布局中的控件都给定义出来啊，然后提供一些public方法来为这些控件设置数据，最后直接在Activity中得到Fragment对象然后调用这些public方法来为Fragment中的控件设置数据啊（就像RecyclerView一样）。没错，这样的确可以实现相应的功能，但是我们使用Fragment的目的就是让Activity变得更加简单，你这么一折腾，Activity中的代码就变得复杂了，违背了我们使用Fragment的初衷*
###2. Fragment向Activity传递数据  

* 在Fragment中定义一个内部回调的接口  
* Activity实现这个接口，并且重写接口中的方法  
自习想一想这个传递数据的方式，由于是Activity中明确实现了接口中的方法，所以，数据是在Activity中处理了的，因此可以理解为什么是Fragment向Activity传递数据了吧。  
##Fragment之间的通信  
如果在当前的Fragment中点击一个按钮启动一个DialogFragment，在后者中点击按钮返回，并且获得数据。这时候涉及到Fragment之间的通信。  

	 AddMatterDialog dialog=new AddMatterDialog();
     dialog.setTargetFragment(this, C.ADDMATTERDIALOG);
     dialog.show(getFragmentManager(),null);  
这是当前Fragment中的点击启动dialogfragment的点击事件。重点是setTargetFragment这句话，把当前的Fragment设为dialogfragment的targetFragment，就实现了一种“绑定”关系。  

	//实现fragment之间的通信
    //实现在MatterFragment中点击menuitem项的加号时弹出对话框AddMatterFragment
    //后者传递给他新添加的matter的信息
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if(requestCode== C.ADDMATTERDIALOG)
        {
            Bundle bundle=data.getExtras();
            matterManager.addMatter(bundle.getString("title"),bundle.getString("content"));
            list.add(bundle.getString("title"));
            adapter.notifyDataSetChanged();
        }
    }  
第二步就是重写Fragment的onActivityResult方法，**该方法和Activity的同名方法不是一个方法，这里该方法返回的是一个Fragment。**   

	/*
         为intent添加数据
            */
                    Bundle data=new Bundle();
                    data.putString("title",title.getText().toString());
                    data.putString("content",content.getText().toString());

                    //把数据包添加到intent中
                    intent.putExtras(data);
                    getTargetFragment().onActivityResult(C.ADDMATTERDIALOG, Activity.RESULT_OK,intent);

然后就可以在dialogfragment中调用targetfragment的onActivityResult方法了。
