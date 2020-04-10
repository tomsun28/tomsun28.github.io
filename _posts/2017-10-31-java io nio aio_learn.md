---
layout: post
title: java io nio aio_learn
date: 2017-10-31
tag: java
---

# 关于Java io nio aio 的学习

<br>
## Java io

<br>

|      | 字节流          | 字符流    |
| ---- | :----------- | ------ |
| 输入流  | InputStream  | Reader |
| 输出流  | OutputStream | Writer |

<br>
![](\images\posts\io_nio_aio\image1.PNG)
<br>
### 1.  Java 文件IO

#### 用字节流，字符流读写文件

```java
public static void Read_WriteToFile()throw IOException{
  File file = new File("path.txt");
  //字节流读取
  byte[] byteArray = new byte[(int)file.length()];
  InputStream ins = new FileInputStream(file);
  int size = ins.read(byteArray);//读取字节流到byteArray中返回大小
  ins.close;
  //字节流写入
  OutputStream ous = new FileOutputStream(file);
  ous.write(byteArray);//将byteArray中的内容写入file
  ous.close;
  //字符流读取
  Reader reader = new FileReader(file);
  char[] charArray = new char[(int)file.length()];
  size = reader.read(charArray);//读取file字符流到charArray
  reader.close;
  //字符流写入
  Write os = new FileWrite(file);
  os.write(charArray);//将charArray内容写入file
  os.close;
}

```

<br>
#### 用缓冲字节流读写文件

```java
public static void Buffer_File()throws IOException{
  //用缓冲字节流读文件
  File file =new File("path.txt");
  byte[] byteArray = new byte[(int)file.length()];
  InputStream ins = new BufferedInputStream(new FileInputStream(file),2*1024);
  int size = ins.read(byteArray);
  ins.close;
  //用缓冲字节流写文件
  OutputStream ous = new BufferedOutputStream(new FileOutputStream(file),2*1024);
  ous.write(byteArray);
  ous.close;
}

```

### 2. BIO网络编程

#### 传统的BIO编程
<br>
传统的同步阻塞模型开发中,ServerSocket负责绑定IP地址,启动监听端口;Socket负责发起连接操作。连接成功后,双方通过输入和输出流进行同步阻塞式通讯。  
传统的BIO模型图：  
![](\images\posts\io_nio_aio\image2.PNG)

<br>
同步阻塞式I/O创建的Server源码：
```
/**
 * 
 */
package io;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @ClassName ServerNormal
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月1日 下午1:45:05
 * @site usthe.com
 */
public final class ServerNormal {
	private static int DEFAULT_PORT=12345;
	private static ServerSocket serverSocket;
	public static void start() throws IOException{
		start(DEFAULT_PORT);
	}
	public synchronized static void start(int port) throws IOException{
		if(serverSocket!=null)
			return;
		try {
			serverSocket = new ServerSocket(port);
			while(true)
			{
				Socket socket = serverSocket.accept();
				new Thread(new ServerHandler(socket)).start();
			}
		} finally {
			// TODO: handle finally clause
			if(serverSocket!=null)
			{
				System.out.println("服务器关闭");
				serverSocket.close();
				serverSocket = null; 
			}
				
		}
		
		
	}
	

}

```
客户端消息处理线程ServerHandler源码：
```
/**
 * 
 */
package io;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;

/**
 * @ClassName ServerHandler
 * @Description TODO 用于处理一个客户端的Socket连接
 * @author tomsun28
 * @Date 2017年11月1日 下午9:57:47
 * @site usthe.com
 */
public class ServerHandler implements Runnable {

	/* (non-Javadoc)
	 * @see java.lang.Runnable#run()
	 */
	private Socket socket;
	 /**
	 * @Title ServerHandler
	 * @Description TODO
	 * @param 
	 * @throws 
	 */
	public ServerHandler(Socket socket) {
		// TODO Auto-generated constructor stub
		this.socket = socket;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		BufferedReader in = null;
		DataOutputStream out = null;
		try {
			in  = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			 out = new DataOutputStream(socket.getOutputStream());
           
			String expression;
			while(true){
				if((expression = in.readLine())==null)break;
				System.out.println("服务器收到消息！"+expression);
				
			}
			out.writeUTF("谢谢连接我：" + socket.getLocalSocketAddress() + "\nGoodbye!");
			out.close();
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}
	}

}

```
同步阻塞式I/O创建的Client源码：
```
/**
 * 
 */
package io;

import java.io.BufferedReader;
import java.io.DataInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.nio.channels.ScatteringByteChannel;

import javax.sound.midi.VoiceStatus;

/**
 * @ClassName Client
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月1日 下午10:20:30
 * @site usthe.com
 */
public class Client {
	private static int DEFAULT_SERVER_PORT = 12345;
	private static String DEFAULT_SERVER_IP = "127.0.0.1";
	public static void send(String expression)
	{
		send(DEFAULT_SERVER_PORT,expression);
	}
	public static void send(int port,String expression)
	{
		Socket socket = null;
		DataInputStream in = null;
		PrintWriter out = null;
		String str;
		try {
			socket = new Socket(DEFAULT_SERVER_IP, port);
			
			out = new PrintWriter(socket.getOutputStream(),true);
			
			out.println(expression);
			System.out.println("hahhahaha");		
				in = new DataInputStream(socket.getInputStream());

                System.out.println("from server"+in.readUTF());

		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}finally{
			if(in != null){
				try {
					in.close();
				} catch (Exception e2) {
					// TODO: handle exception
				}
				in = null;
				
			}
			if(out !=null){
				out.close();
				out = null;
			}
			if(socket != null)
			{
				try{
					socket.close();
				}catch(IOException e){
					e.printStackTrace();
				}
				socket = null;
			}
		}
	}
}

```
I/O测试代码：
```
/**
 * 
 */
package io;


/**
 * @ClassName TestBIO
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月1日 下午10:34:17
 * @site usthe.com
 */
public class TestBIO {
	
	public static void main(String args[]){
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				try {
					ServerNormal.start();
				} catch (Exception e) {
					// TODO: handle exception
					e.printStackTrace();
				}
			}
		}).start();
		String exp = "hello server!";
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				// TODO Auto-generated method stub
				Client.send(exp);
			}
		}).start();
	}
}

```
<br>
#### 伪异步I/O编程

为了改进这种一个connect一个thread情况,用线程池来管理请求线程,实现一个或多个线程处理N个客户端(但底层还是BIO),这种通常称为伪异步IO。
伪异步IO模型图：
![](\images\posts\io_nio_aio\image3.PNG)
<br>
改动server代码,将为连接请求所新建的线程交给线程池管理。
```
/**
 * 
 */
package io;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @ClassName ServerNormal
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月1日 下午1:45:05
 * @site usthe.com
 */
public final class ServerNormal {
	private static int DEFAULT_PORT=12345;
	private static ServerSocket serverSocket;
	private static ExecutorService executorService = Executors.newFixedThreadPool(60);
	public static void start() throws IOException{
		start(DEFAULT_PORT);
	}
	public synchronized static void start(int port) throws IOException{
		if(serverSocket!=null)
			return;
		try {
			serverSocket = new ServerSocket(port);
			while(true)
			{
				Socket socket = serverSocket.accept();
//				new Thread(new ServerHandler(socket)).start();
				executorService.execute(new ServerHandler(socket));
			}
		} finally {
			// TODO: handle finally clause
			if(serverSocket!=null)
			{
				System.out.println("服务器关闭");
				serverSocket.close();
				serverSocket = null; 
			}		
		}
	}
}

```

<br>
<br>
## Java nio

<br>
  IO是以流的方式处理数据,一次一个字节的处理数据,而NIO是以块的方式处理数据,一次操作消费一个数据块。NIO可以为非阻塞IO。  
  Buffer,Channel,Seletor 是NIO的三个核心对象。  

<br>
### 缓冲区Buffer

<br>
  Buffer是一个对象,包含一些要写入或者读出的数据。在NIO中，数据是放入buffer对象的，而在IO中，数据是直接写入或者读到Stream对象的。应用程序不能直接对 Channel 进行读写操作，而必须通过 Buffer 来进行，即 Channel 是通过 Buffer 来读写数据的。  
  在NIO中，所有的数据都是用Buffer处理的，它是NIO读写数据的中转池。Buffer实质上是一个数组，通常是一个字节数据，但也可以是其他类型的数组。但一个缓冲区不仅仅是一个数组，重要的是它提供了对数据的结构化访问，而且还可以跟踪系统的读写进程。  
  使用 Buffer 读写数据一般遵循以下四个步骤：

    1. 写入数据到 Buffer；
    2. 调用 flip() 方法；
    3. 从 Buffer 中读取数据；
    4. 调用 clear() 方法或者 compact() 方法。
    
    
    limit - buffer读写上限，
    posttion - 读写时当前游标的位置
    capacity - 缓冲区的最大容量
    mark     - 快照点，供reset恢复到快照点
    
    clear 清空缓冲区  posttion=0,limit=capacity,mark=-1
    flip  反转缓冲区  limit=position,position=0,mark=-1
    rewind 重0开始读缓冲区 position=0,mark=-1
    reset 根据mark恢复缓冲区  position=mark
    
    compact 压实缓冲区，复用缓冲区空间，将已读的内容丢弃
    slice 共享缓冲区空间limit-position，其获取的是同一缓冲区
    

  当向 Buffer 写入数据时，Buffer 会记录下写了多少数据。一旦要读取数据，需要通过 flip() 方法将 Buffer 从写模式切换到读模式。在读模式下，可以读取之前写入到 Buffer 的所有数据。  
  一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。有两种方式能清空缓冲区：调用 clear() 或 compact() 方法。clear() 方法会清空整个缓冲区。compact() 方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。  
  Buffer主要有如下几种：  

![](\images\posts\io_nio_aio\image4.PNG)

<br>
### 通道Channel

<br>
  Channel是一个对象，可以通过它读取和写入数据。所有数据都通过Buffer处理,不会直接把字节写入到Channel中,也不会直接从Channel中读数据。可以把它看做IO中的流。但是它和流相比还有一些不同：  

    1. Channel是双向的，既可以读又可以写，而流是单向的
    2. Channel可以进行异步的读写
    3. 对Channel的读写必须通过buffer对象

  在Java NIO中Channel主要有如下几种类型：  

    1. FileChannel：从文件读取数据的
    2. DatagramChannel：读写UDP网络协议数据
    3. SocketChannel：读写TCP网络协议数据
    4. ServerSocketChannel：可以监听TCP连接

<br>
### 多路复用器Selector

<br>
  Selector是Java NIO编程基础,其是一个对象,它可以注册到很多个Channel上,监听各个Channel上发生的事件,并且能够根据事件情况决定Channel读写。这样通过一个线程管理多个Channel,就可以处理大量网络连接了。  

1. 创建一个Selector  

```
Selector selector = Selector.open();
```
2. 注册Channel到Selector上  

```
channel.configureBlocking(false); //注册的Channel必须设置成异步模式
SelectionKey key = channel.register(selector,SelectionKey.OP_READ); //注册感兴趣的事件到selector获得对应的SelectorKey
//需要注意register()方法的第二个参数，它是一个“interest set”,意思是注册的Selector对Channel中的哪些时间感兴趣，事件类型有四种：connect,accept,read,write 
//通道触发了一个事件意思是该事件已经 Ready(就绪)。所以某个Channel成功连接到另一个服务器称为 Connect Ready。一个ServerSocketChannel准备好接收新连接称为 Accept Ready，一个有数据可读的通道可以说是 Read Ready，等待写数据的通道可以说是Write Ready。
//上面四个事件多对应到SelectionKey中的常量为：SelectionKey.OP_CONNECT,SelectionKey.OP_ACCEPT,SelectionKey.OP_READ,SelectionKey.OP_WRITE
//多个感兴趣事件操作符连接：SelectionKey.OP_ACCEPT|SelectionKey.OP_READ

```
<br>
### NIO编程

<br>
#### Server

<br>
```
/**
 * 
 */
package nio;



/**
 * @ClassName Server
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月3日 下午7:30:20
 * @site usthe.com
 */
public class Server {
	 private static int DEFAULT_PORT=12345;
	 private static ServerHandle serverHandle;
	 public static void start(){
		 start(DEFAULT_PORT);
	 }
	 
	 public static synchronized void start(int port){
		 if(serverHandle!=null){
			 serverHandle.stop();
		 }
		 serverHandle = new ServerHandle(port);
		 new Thread(serverHandle,"server").start();
		 
	 }
	 
	 public static void main(String[] args){
		 start();
	 }
	

}


```
<br>
#### ServerHandle

<br>
```
/**
 * 
 */
package nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;



/**
 * @ClassName ServerHandle
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月3日 下午7:43:24
 * @site usthe.com
 */
public class ServerHandle implements Runnable {
	
	private static Logger logger = LoggerFactory.getLogger(ServerHandle.class);
	
	private Selector selector;
	private ServerSocketChannel serverChannel;
	private volatile boolean started;
	
	public ServerHandle(int port){
		
		try {
			//创建选择器
			selector = Selector.open();
			//打开监听的通道
			serverChannel = ServerSocketChannel.open();
			//将此通道设置为非阻塞模式
			serverChannel.configureBlocking(false);
			//绑定端口port，backlog设置为1024
			serverChannel.socket().bind(new InetSocketAddress(port), 1024);
            //将channel注册到selector上,注册响应事件
			serverChannel.register(selector,SelectionKey.OP_ACCEPT);
			//标记服务已经开启
			started = true;
			System.out.println("服务器已经启动端口号为："+port);
			
			
		} catch (Exception e) {
			// TODO: handle exception
			logger.error("出错系统进行退出：{}", e);
			System.exit(1);
		}
	}
	
	public void stop(){
		started=false;
	}
	
	public void run(){
		while(started){
			try {
				//无论是否有事件发生,每隔以一秒唤醒一次
				selector.select(1000);
				//阻塞形式,至少有一个注册的事件发生时才会继续
				//selector.select();
				//获取每个管道对应的selectionKey
				Set<SelectionKey> keys = selector.selectedKeys();
				Iterator<SelectionKey> iterator = keys.iterator();
				SelectionKey key = null;
				while(iterator.hasNext()){
					key = iterator.next();
					iterator.remove();
					try {
						inputHandle(key);
					} catch (Exception e) {
						// TODO: handle exception
						if(key!=null){
							key.cancel();
							if(key.channel()!=null){
								key.channel().close();
							}		
						}
					}
				}
			} catch (Exception e) {
				// TODO: handle exception
				logger.error(e.toString());
			}
			
		}
		if(selector!=null){
			try {
				selector.close();
			} catch (Exception e) {
				// TODO: handle exception
				logger.error("关闭selector失败：{}",e);
			}
		}
	}
	
	private void inputHandle(SelectionKey key) throws IOException{
		if(key.isValid())
		{
			//处理新接入的请求消息
			if(key.isAcceptable()){
				ServerSocketChannel ssc = (ServerSocketChannel)key.channel();
				//通过serverSocketChannel的accept()创建SocketChannel实例
				SocketChannel sc = ssc.accept();
				//将SocketChannel设置为非阻塞
				sc.configureBlocking(false);
				//把socketChannel注册到selector上,注册事件为OP_READ
				sc.register(selector,SelectionKey.OP_READ);
			}
			//处理读消息
			if(key.isReadable()){
				SocketChannel sc = (SocketChannel)key.channel();
				//创建ByteBuffer开辟1M的缓冲区
				ByteBuffer buffer =ByteBuffer.allocate(1024);
				//将请求消息读出写入到buffer中,返回读到的字节数
				int readBytes = sc.read(buffer);
				
				if(readBytes>=0){
					//将缓冲区buffer转换成读模式
					buffer.flip();
					//根据缓冲区可读的字节数创建字节数组
					byte[] bytes = new byte[buffer.remaining()];
					//将缓冲区可读的字节数组复制到新建的字节数组bytes中
					buffer.get(bytes);
					String expression = new String(bytes,"UTF-8");
					logger.info("服务器收到消息：{}",expression);
					//发送应答消息
					String response = "嗨我是服务器！";
					//清空buffer
					buffer.clear();
					//将buffer转换为写模式
					buffer.flip();
					//写数据到buffer
					buffer.put(response.getBytes());
					//把buffer转换为读模式
					buffer.flip();
					//将buffer的数据读取到channel中发送
					sc.write(buffer);
				}
				else{
					key.cancel();
					sc.close();
				}
			}
		}			
		}		
	}




```
<br>
#### Client

<br>
```
/**
 * 
 */
package nio;

import java.io.IOException;

/**
 * @ClassName Client
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月4日 下午2:10:23
 * @site usthe.com
 */
public class Client {
	private static String DEFAULT_HOST = "127.0.0.1";
	private static int DEFAULE_PORT = 12345;
	private static ClientHandle clientHandle;
	public static void start(String ip,int port){
		if(clientHandle!=null){
			clientHandle.stop();
		}
		clientHandle = new ClientHandle(ip,port);
		new Thread(clientHandle,"client").start();
	}
	public static boolean sendMsg(String msg) throws IOException
	{
		if(msg.equals("q")) return false;
		clientHandle.sendMsg(msg);
		return true;
	}
}

```
<br>
#### ClientHandle

<br>
```
/**
 * 
 */
package nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.ClosedChannelException;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @ClassName ClientHandle
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月4日 下午1:30:09
 * @site usthe.com
 */
public class ClientHandle implements Runnable {

	private static final Logger logger = LoggerFactory.getLogger(ClientHandle.class);
	
	private String host;
	private int port;
	private Selector selector;
	private SocketChannel socketChannel;
	private volatile boolean started;
	
	public ClientHandle(String ip,int port){
		this.host=ip;
		this.port=port;
		try {
			//创建选择器
			selector = Selector.open();
			//打开通道
			socketChannel = SocketChannel.open();
			//ture为阻塞模式,false为非阻塞模式
			socketChannel.configureBlocking(false);
			started=true;
		} catch (Exception e) {
			// TODO: handle exception
			logger.error("通道开启失败：{}",e);
			System.exit(1);
		}
		
		
	}
	
	public void stop(){
		started = false;
	}
	
	@Override
	public void run() {
		// TODO Auto-generated method stub
		
		try {
			if(socketChannel.connect(new InetSocketAddress(host, port)));
			else
				socketChannel.register(selector,SelectionKey.OP_READ);
		} catch (IOException e) {
			// TODO Auto-generated catch block
//			e.printStackTrace();
			logger.error(e.toString());
			System.exit(1);
		}
		
		while(started){
			
			try {
				selector.select(1000);
				Set<SelectionKey> keys = selector.selectedKeys();
				Iterator<SelectionKey> iterator = keys.iterator();
				SelectionKey key = null;
				while(iterator.hasNext()){
					key = iterator.next();
					iterator.remove();
					try {
						inputHandle(key);
					} catch (Exception e) {
						// TODO: handle exception
						if(key!=null)
						{
							key.cancel();
							if(key.channel()!=null){
								key.channel().close();
							}
						}
					}
				}
				
			} catch (Exception e) {
				// TODO: handle exception
				logger.error(e.toString());
				System.exit(1);
			}
			
		}
		
		if(selector!=null){
			try {
				selector.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}	
		}
	}
	private void inputHandle(SelectionKey key) throws IOException
	{
		if(key.isValid()){
			SocketChannel sc = (SocketChannel)key.channel();
			
			if(key.isReadable()){
				ByteBuffer buffer = ByteBuffer.allocate(1024);
				int readBytes = sc.read(buffer);
				if(readBytes>0){
					buffer.flip();
					byte[] bytes = new byte[buffer.remaining()];
					buffer.get(bytes);
					String result = new String(bytes,"UTF-8");
					logger.info("客户端收到消息：{}",result);		
				}
				else if(readBytes<0){
					key.cancel();
					sc.close();
				}	
			}
		}
	}
	public void sendMsg(String msg) throws IOException{
		socketChannel.register(selector,SelectionKey.OP_READ);
		byte[] bytes = msg.getBytes();
		ByteBuffer buffer = ByteBuffer.allocate(bytes.length);
		buffer.put(bytes);
		buffer.flip();
		socketChannel.write(buffer);
	}
}

```
<br>
#### Test

<br>
```
/**
 * 
 */
package nio;

import java.io.IOException;
import java.util.Scanner;

/**
 * @ClassName Test
 * @Description TODO
 * @author tomsun28
 * @Date 2017年11月4日 下午2:07:46
 * @site usthe.com
 */
public class Test {
	
	private static String DEFAULT_HOST = "127.0.0.1";
	private static int DEFAULE_PORT = 12345;
	@SuppressWarnings("resource")
	public static void main(String[] args) throws InterruptedException, IOException
	{
		Server.start();
		Thread.sleep(1000);
		Client.start(DEFAULT_HOST, DEFAULE_PORT);
		while(Client.sendMsg(new Scanner(System.in).nextLine()));
		
	}

}

```
<br>
<br>
<br>
## Java AIO

<br>
待续。。。。。。。

<br>
<br>
<br>
转载自[Java网络IO编程总结](http://blog.csdn.net/anxpp/article/details/51512200) 参考自 [菜鸟教程](http://www.runoob.com/java/java-files-io.html)   [Java NIO详解](http://blog.csdn.net/suifeng3051/article/details/48160753)
