#### 网络编程

网络编程使用java.net包

常见的软件架构：

1.C/S：即Client/Server，在用户本地需要下载并安装客户端程序，在远程有一个服务器端程序。例如QQ

优点：画面可以做得非常精美，用户体验好

缺点：既要开发客户端，又要开发服务端。用户需要下载和更新的时候太麻烦

2.B/S：即Browser/Server，只需要一个浏览器，用户通过不同的网址，客户访问不同的服务器。例如京东

优点：不需要开发客户端，只需要页面+服务端。开发、部署、维护都很简单。用户也不需要下载。

缺点：如果应用过大，用户体验会受到影响



#### 网络编程三要素

1.确认对方的地址（IP）

2.确认对方接收数据的软件（端口号，一个端口号只能被一个软件绑定使用）

3.确定网络传输的规则（协议）

所以，网络编程的三要素：IP（**设备**在网络中的地址，是唯一的标识）、端口号（应用程序在设备中唯一的标识）、协议



#### IPv4

全称：Internet Protocol version 4，互联网通信协议第四版

采用32位地址长度，分成4组。所以真正的IP地址并不是我们在电脑上所看到的IP地址。

32位，就是4个字节，为了更好表示，点分十进制法应运而生：将每个字节的二进制表示转成对应十进制数字。

网络编程的内容在我之前的文章里面有相对详细的讲解。我们知道IPv4最多只能表示42.9亿个地址。在2019年11月26日已经全部分配完毕。IPv6应运而生



#### IPv4的地址分类形式

公网地址（万维网使用）和私有地址（局域网使用）。192.168.开头的就是私有地址，范围即为192.168.0.0--192.168.255.255，专门为组织机构内部使用，以此节省IP

怎么个节约法呢？以网吧为例，网吧里的电脑有很多，但不是每一台电脑在连接外部网络时都有一个公网IP，它们往往共享同一个公网IP，再由路由器给每一台电脑分配局域网IP，这样就实现了IP的节约。

一个特殊的IP：127.0.0.1，也可以试试localhost，它是回送地址，也称本地回环地址，也称本地IP，永远只会寻找当前所在本机。

假设192.168.1.100是我电脑的IP，那么这个IP跟127.0.0.1是一样的吗？答案是否定的。每一个路由器给你分配的IP都有可能不同，即当你换了一个地方上网，局域网IP有可能会发生变化。但如果我们往127.0.0.1发送数据，那么它是不需要经过路由器的，网卡会直接把数据发过来。

关于网络编程的两个常用cmd命令：ipconfig和ping，不赘述，网络编程的内容在我之前的文章有详细的讲解。



#### IPv6

采用128位地址长度，分成8组，即每组16个bit，它能给地球上每一粒沙子都分配一个IP地址。为了方便表示，冒分十六进制表示法应运而生。冒分就是用冒号隔开，并且还会把每一节的前导零省略。如果计算出的16进制表示形式中间有多个连续的，那么就可以使用0位压缩表示法，即把这些0及其中间的冒号都统一用两个冒号表示。



#### InetAddress的使用

此类表示互联网协议地址。它有两个子类：`Inet4Address`和`Inet6Address`，在底层它会判断你的系统是使用4还是6，创建的时候实际上创建的是子类。它没有对外提供构造方法，我们需要使用它的静态方法`getByName`去获取对象。

| 方法名称                                  | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| static InetAddress getByName(String host) | 确认主机名称的IP地址，主机名称可以是及其名称，也可以是IP地址 |
| String getHostName()                      | 获取此IP地址的主机名                                         |
| String getHostAddress()                   | 返回文本显示中的IP地址字符串                                 |

```java
public class MyNetDemo {
    public static void main(String[] args) throws UnknownHostException {
        //1.获取InetAddress对象
        //它是IP的对象 实际上可以理解成就是一台电脑的对象
        InetAddress address = InetAddress.getByName("tkv");
        System.out.println(address);

        System.out.println(address.getHostAddress());
        System.out.println(address.getHostName());
    }
}
```



#### 端口号

端口号是应用程序在设备中唯一的标识，它是由两个字节表示的整数，取值范围为0-65535

其中0-1023之间的端口号用于一些知名的网络服务或者应用，我们自己使用1024以上的端口号即可。一个端口号只能被一个应用程序使用。



#### UDP协议

全称用户数据报协议（User Datagram Protocol），是面向无连接通信协议。速度快，有大小限制，一次最多发送64K，但数据不安全，易丢失数据。所以适用于丢失一点数据也无伤大雅的情况，例如网络会议、语音通话、在线视频。

理解面向无连接：UDP不会检查两台电脑是否已经连接成功。

```java
public class MySendMessageDemo {
    public static void main(String[] args) throws IOException {
        //1.创建DatagramSocket对象
        //创建时还会绑定端口，如果是空参构造，则会在所有可用的端口中随机选择一个
        //如果使用有参的，就使用指定的端口
        DatagramSocket ds = new DatagramSocket();

        //2.打包数据
        String str = "666";
        byte[] bytes = str.getBytes();
        InetAddress addr = InetAddress.getByName("127.0.0.1");
        int port = 10086;

        DatagramPacket dp = new DatagramPacket(bytes,bytes.length,addr,port);

        //3.发送数据
        ds.send(dp);

        //4.释放资源
        ds.close();
    }
}
```

```java
public class MyReceiveMessageDemo {
    public static void main(String[] args) throws IOException {
        //1.创建DatagramSocket对象
        //在接受的时候一定要绑定端口，而且绑定的端口一定要与发送的端口一致
        DatagramSocket ds = new DatagramSocket(10086);

        //2.接收数据包
        byte[] bytes = new byte[1024];
        DatagramPacket dp = new DatagramPacket(bytes,bytes.length);
        ds.receive(dp);//阻塞方法

        //3.解析数据包
        byte[] data = dp.getData();
        int len = dp.getLength();
        InetAddress addr = dp.getAddress();
        int port = dp.getPort();

        System.out.println("接收到数据" + new String(data,0,len));
        System.out.println("从" + addr + "这台电脑的" + port + "这个端口发出的");

        //4.释放资源
        ds.close();
    }
}
```



UDP有三种通信方式：单播、组播、广播，顾名思义。上面的代码就是单播，一对一。

组播：组播地址：224.0.0.0-239.255.255.255，其中224.0.0.0-224.0.0.255为预留的组播地址。它和IP不一样的地方在于，IP只能表示一台电脑，而一个组播地址可以表示局域网内的多台电脑。

```java
public class SendMessageDemo {
    public static void main(String[] args) throws IOException {
        //创建MulticastSocket对象
        MulticastSocket ms = new MulticastSocket();

        //创建DatagramPacket对象
        String s = "小野寺小咲不要哭";
        byte[] bytes = s.getBytes();
        InetAddress addr = InetAddress.getByName("224.0.0.1");
        int port = 8080;

        DatagramPacket dp = new DatagramPacket(bytes,bytes.length,addr,port);
        
        //3.发送数据
        ms.send(dp);
        
        //4.释放资源
        ms.close();

    }
}
```

```java
public class ReceiveMessageDemo {
    public static void main(String[] args) throws IOException {
        //1.创建MulticastSocket对象
        MulticastSocket ms = new MulticastSocket(8080);
        
        //2.将当前本机添加到224.0.0.1的这一组当中
        InetAddress addr = InetAddress.getByName("224.0.0.1");
        ms.joinGroup(addr);
        
        //3.创建DatagramPacket数据包对象
        byte[] bytes = new byte[1024];
        DatagramPacket dp = new DatagramPacket(bytes,bytes.length);
        
        //4.接收数据
        ms.receive(dp);
        
        //5.解析数据
        byte[] data = dp.getData();
        int len = dp.getLength();
        String ip = dp.getAddress().getHostAddress();
        String name = dp.getAddress().getHostName();

        System.out.println("IP为：" + ip + "，主机名为：" + name + "的人，发送了数据：" + new String(data,0,len));
        
        //6.释放资源
        ms.close();
        
    }
}
```

可以多创建几个接收端验证。



广播：广播地址：255.255.255.255

其代码和单播几乎一样，只需要把单播的IP改为广播地址，就可以对局域网中所有的电脑发送数据。



#### TCP协议

全称传输控制协议TCP（Transmission Control Protocol），是面向连接的通信协议。速度慢，没有大小限制，数据安全。它要确保连接成功才会发送数据。适用于对数据要求高的情况，例如下载软件、文字聊天、发送邮件。

TCP在通信的两端各建立一个Socket对象，通过Socket产生IO流来进行网络通信。

```java
public class Client {
    public static void main(String[] args) throws IOException {
        //1.创建Socket对象
        //在创建对象的同时会连接服务端，如果连接不上，会报错
        Socket socket = new Socket("127.0.0.1",8080);

        //2.从连接通道中获取输出流
        OutputStream os = socket.getOutputStream();

        //写出数据
        os.write("吾妻樱岛麻衣".getBytes());

        //3.释放资源
        os.close();
        socket.close();
    }
}
```

```java
public class Server {
    public static void main(String[] args) throws IOException {
        //1.创建对象ServerSocket
        ServerSocket ss= new ServerSocket(8080);

        //2.监听客户端连接
        Socket socket = ss.accept();//阻塞

        //3.从连接通道中获取输入流读取数据
        InputStream is = socket.getInputStream();
        int b;
        while((b = is.read()) != -1) {
            System.out.println((char)b);
        }

        //4.释放资源
        socket.close();//断开与客户端的连接
        ss.close();//关闭服务器

    }
}
```

运行后发现，会出现乱码。分析一下原因：在写数据的时候，我们没有指定规则，所以使用IDEA的默认规则UTF-8，那么一个中文汉字就对应三个字节，而在读数据的时候是一个字节一个字节地读，然后把它转成字符。所以相当于每次转的是三分之一个中文。所以我们不能拿字节流去读，可以使用转换流将字节流变成字符流。

```java
public class Server {
    public static void main(String[] args) throws IOException {
        //1.创建对象ServerSocket
        ServerSocket ss= new ServerSocket(8080);

        //2.监听客户端连接
        Socket socket = ss.accept();//阻塞

        //3.从连接通道中获取输入流读取数据
        InputStream is = socket.getInputStream();
        InputStreamReader isr = new InputStreamReader(is);//转换流
        BufferedReader br = new BufferedReader(isr);//缓冲流
        int b;
        while((b = br.read()) != -1) {
            System.out.println((char)b);
        }

        //4.释放资源
        socket.close();//断开与客户端的连接
        ss.close();//关闭服务器
    }
}
```



#### 关于TCP

