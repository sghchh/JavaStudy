#URL类  
URL：统一资源定位符，由协议名和资源名组成，协议名例如：http，https，ftp等；出去协议名，后面的那一长串是资源名。  
URL类的方法直接通过代码来一一讲解。  

	import java.net.*;
	public class URLTest {

		public static void main(String[] args)
		{
		try {
			URL url=new URL("http://www.baidu.com/home/news/data/newspage?nid=16599126135219466111&n_type=0&p_from=1&dtype=-1");
		    System.out.println("文件名："+url.getFile());
		    System.out.println("host是："+url.getHost());
		    System.out.println("路径是："+url.getPath());
		    System.out.println("Port:"+url.getPort());
		    System.out.println("query:"+url.getQuery());
		    System.out.println("DefaultPort:"+url.getDefaultPort());
		    System.out.println("相对路径:"+url.getRef());
		} catch (MalformedURLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
	}  
    ------------output----------  
	文件名：/home/news/data/newspage?nid=16599126135219466111&n_type=0&p_from=1&dtype=-1
	host是：www.baidu.com
	路径是：/home/news/data/newspage
	Port:-1
	query:nid=16599126135219466111&n_type=0&p_from=1&dtype=-1
	name:80
	相对路径:null  
首先：URL类支持多种构造器，其中有的构造器可以为url指定端口号，如果没有显示指明端口号，那么根据协议的不同会使用默认的端口，比如http协议的默认端口号是80。首先看url.getPort()方法，这个方法在这个实例中的返回值是-1，这是因为如果没有显式指定端口号的话，这个方法的返回值就是-1；而调用url.getDefaultPort()方法才是返回协议的默认端口号。  
#InetAddress类  
InetAddress类主要用来标志一台电脑的，它主要涉及到host以及port信息，和URL要区分开；这个类没有暴露构造器，但是提供了一些静态方法来创建一个InetAddress对象：  

	public class Test {
		public static void main(String[] args) throws UnknownHostException
		{
			InetAddress address=InetAddress.getLocalHost();
			System.out.println(address.getHostAddress());
			System.out.println(address.getHostName());
		
			InetAddress address2=InetAddress.getByName("DESKTOP-67VB9NI");
			System.out.println(address2.getHostAddress());
		
	}

	}