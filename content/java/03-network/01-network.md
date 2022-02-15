---
title: 网络编程
date: 2021-04-09
categories: [Network]
toc: true
---

## 1、概述

#### 计算机网络

计算机网络是指将[地理](https://baike.baidu.com/item/地理)位置不同的具有独立功能的多台[计算机](https://baike.baidu.com/item/计算机/140338)及其外部设备，通过通信线路连接起来，在[网络操作系统](https://baike.baidu.com/item/网络操作系统/3997)，[网络管理软件](https://baike.baidu.com/item/网络管理软件/6579078)及[网络通信协议](https://baike.baidu.com/item/网络通信协议/4438611)的管理和协调下，实现[资源共享](https://baike.baidu.com/item/资源共享/233480)和信息传递的计算机系统。

#### 网络编程的目的

传播交流信息，数据交换，通信

#### 想要达到这个效果需要什么：

1. 如何准确的定位网络上的一台主机
2. 如何传输数据？



javaweb：网页编程	B/S

网络编程：TCP/IP	C/S



## 2、网络通信的要素

TCP/IP参考模型：

![image-20210420192100552](../../../../../../../../Library/Application Support/typora-user-images/image-20210420192100552.png)

小结：

1. 网络编程中有两个主要问题
	- 如何准确的定位到网络上的一台或者多台主机
	- 找到主机之后如何进行通信
2. 网络编程中的要素
	- IP 和 端口号
	- 网络通信协议
3. 万物皆对象



## 3、IP

IP 地址：InetAddress

- 唯一定位一台网络上计算机
- 127.0.0.1：本机 localhost
- IP 地址的分类
	- IPV4 / IPV6
		- IPV4 = 127.0.0.1，4个字节组成，0～255，42亿。
		- IPV6 = 2001:0bbd:aaaa:0015:3333:4444:2222:9999，128位，8个无符号整数
	- 公网（互联网）- 私网（局域网）
		- ABCD类地址
		- 192.168.200.X，专门给组织内部使用
	- 域名：记忆IP问题



## 4、端口

端口表示计算机上的一个程序的进程；

- 不同的进程有不同的端口号

- 被规定 0 ～ 65535

- TCP，UDP：65535*2，单个协议下，端口号不能冲突

- 端口分类

	- 公有端口 0～1023

		- HTTP：80
		- HTTPS：443
		- FTP：21
		- Telent：23

	- 程序注册端口：1024～49151，分配用户或者程序

		- Tomcat：8080
		- MySQL：3306
		- Oracle：1521

	- 动态、私有：49152～65535

		`netstat -ano` 查看所有的端口

		`netstat -ano|findstr "5900"` 查看特定端口

		`tasklist|findstr "9696"` 根据pid查询端口

		

## 5、通信协议

协议：约定，就跟我们现在说的普通话。

网络通信协议：速率，传输码率，代码结构，传输控制。。。

问题：非常的复杂

#### TCP/IP协议

重要：

- TCP：用户传输协议
- UDP：用户数据协议
- IP：网络互连协议

#### TCP、UDP 对比

TCP：打电话

- 连接，稳定
- 三次握手，四次挥手
	- 你瞅啥？瞅你咋地？干一场！（最少需要三次，才能保证连接！）
	- 我要断开，我知道你要断开了，你真的断开了吗？我真的要走了！
- 客户端、服务端
- 传输完成，释放连接，效率低

UDP：发短信

- 不连接，不稳定
- 客户端、服务端：没有明确的界限
- 不管有没有准备好，都可以发给你
- 导弹
- DDOS：洪水攻击（饱和攻击）



## 6、TCP

客户端：

1. 连接服务器，通过Socket
2. 发送消息

```java
		public static void main(String[] args) {
        Socket socket = null;
        OutputStream os = null;
        try {
            // 1.服务器的地址
            InetAddress serverIP = InetAddress.getByName("127.0.0.1");
            int port = 9999;
            // 2.创建一个socket连接
             socket = new Socket(serverIP, port);
            // 3.发送消息
             os = socket.getOutputStream();
            os.write("hello".getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                os.close();
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```



服务器：

1. 建立服务的端口，ServerSocket
2. 等待用户的连接，通过accept
3. 接收用户的消息

```java
    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        Socket socket = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try {
            // 1.要有一个地址
            serverSocket = new ServerSocket(9999);
            // 2.等待客户端连接
            socket = serverSocket.accept();
            // 3.读取客户端消息
            is = socket.getInputStream();
            // 管道流
            baos = new ByteArrayOutputStream();
            byte[] bytes = new byte[1024];
            int len;
            while ((len = is.read(bytes)) != -1) {
                baos.write(bytes, 0, len);
            }
            System.out.println(baos.toString());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                baos.close();
                is.close();
                socket.close();
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
```



## 7、UDP

发短信：不用连接，需要知道对方的地址！

发送消息：

```java
public class UdpClient {
    public static void main(String[] args) throws Exception {
        // 1.建立一个socket
        DatagramSocket socket = new DatagramSocket();
        // 2.建个包
        String msg = "hello";
        DatagramPacket packet = new DatagramPacket(
                msg.getBytes(),
                0,
                msg.getBytes().length,
                InetAddress.getByName("127.0.0.1"),
                9999);
        // 3.发送包
        socket.send(packet);
        // 4.关闭流
        socket.close();
    }
}
```

接收消息：

```java
public class UdpServer {
    public static void main(String[] args) throws Exception {
        // 开放端口
        DatagramSocket socket = new DatagramSocket(9999);
        // 接收数据包
        byte[] bytes = new byte[1024];
        DatagramPacket packet = new DatagramPacket(bytes, 0, bytes.length);
        socket.receive(packet);
        System.out.println(packet.getAddress().getHostAddress());
        System.out.println(new String(packet.getData()));
        socket.close();
    }
}
```



















