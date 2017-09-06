#SugarORM-安卓数据库使用简介  
一. 在gradle中添加依赖  
    
    compile 'com.github.satyan:sugar:1.4'  
######
二.在清单文件中添加配置  
    
        <meta-data android:name="DATABASE" android:value="sugar.db"/>
        <meta-data android:name="VERSION" android:value="1"/>
        <meta-data android:name="QUERY_LOG" android:value="true"/>
        <meta-data android:name="DOMAIN_PACKAGE_NAME" android:value="sugar"/>  
这些个属性都是Application下的，下面就介绍一下这些个属性的意思：  
1.DATABASE：value值是系统创建的数据库文件的名字  
2.VERSION：数据库的版本  
3.QUERY_LOG：value值是布尔型，表示是否支持log  
4.DOMAIN_PACKAGE_NAME：数据库实现类的包名  
####
三.创建数据库的实现类，继承自SugarRecord

    public class Book extends SugarRecord {
    public String name;
    public Book(){}
    public Book(String name){
        this.name=name;
    }
    //get和set方法
    public String getName(){
        return this.name;
    }
    public void setName(String name){
        this.name=name;
    }
    }
这里有坑：1.这个实现类的构造器中必须有一个无参构造器 2.SugarRecord自己内置了一个Long型参数id（自增），如果非要自定义一个id必须用采用@SerializedName（“id”）注解  
######
四.使用  
在初始化数据库对象之前一定要**添加以下代码**  
  
    SugarContext.init（）；  
接下来就是简单的数据的增删改了。  
  
    插入：save（）方法
    Book book=new Book("穆斯林的葬礼");
    book.save();
    
    删除：delete（）方法
    Book book=Book.findById(Book.class,id);
    book.delete();
   
    查询：
    //1. list条件查询
    List<Book> books = Book.find(Book.class, "name=？","西游记");
    //2. bean条件查询
    Book.find(Book.class, "name = ?", "金瓶梅");
    //3. 条件查询，以此是分组，排序，limit查询
    find(Class<T> type, String whereClause, String[] whereArgs, String groupBy, String orderBy, String limit)
    //4. 自定义查询，依然支持查询语句查询
    // Could execute other raw queries too here..
    Note.executeQuery("VACUUM");
    // for finders using raw query.
    List<Note> notes = Note.findWithQuery(Note.class, "Select * from Note where name = ?", "satya");

