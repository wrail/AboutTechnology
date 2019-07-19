## Tomcat介绍

> Tomcat是一个开源的 Java Web 应用服务器，实现了 Java EE(Java Platform Enterprise Edition)的部 分技术规范，比如 Java Servlet、Java Server Page、JSTL、Java WebSocket。
>
> Java EE 是 Sun 公 司为企业级应用推出的标准平台，定义了一系列用于企业级开发的技术规范，除了上述的之外，还有 EJB、Java Mail、JPA、JTA、JMS 等，而这些都依赖具体容器的实现。

一些市面上的Web服务器，如JBoss，Jetty等等

![img](https://upload-images.jianshu.io/upload_images/10649427-08b59b78625b3476.jpeg?imageMogr2/auto-orient/)

## Tomcat的组成

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190719140036155.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

* `Server`：代表Tomcat容器，包含多组服务，负责对各个Service的管理和启动，同时监听8005端口发过来的shutdown，用来关闭整个容器。
* `Service`：用于提供服务，Service主要包含Connector和Container两个核心组件，和多个功能组件，多个Service间是独立的，但是都共享着Tomcat的JVM资源，
* `Connector`：用于Tomcat和外部的连接，监听某固定端口接收到的外部请求，然后传递给Container，并且将Container的处理结果返回给外部。
* `Container`：Servlet容器，内部有多层容器组成，处理业务逻辑，用来管理Servlet的生命周期。
* `Jasper`：tomcat的JSP解析器，将JSP转为JAVA文件，编译为class文件。
* `Naming`：命名服务，主要是将名称和class对象联系起来，可以使用此名称去访问对象。
* `Session`：负责Session的生命周期，以及Session的持久化，集群，自定义等等，经常用来存一些临时信息。
* `Loging`：日志记录，包含一些运行信息，错误信息等。
* `JMX`：JavaSE的技术规范，是一个为应用程序、设备、系统等植入管理功能的框架，通过 JMX 可以远程监控 Tomcat 的运行状态。

### Connector的组成

`Connector`用于接受请求并将请求封装成Request和Response，然后交给`Container`进行处理，`Container`处理完之后再交给`Connector`返回给客户端。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190719172812765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

可以看到Connector实际上是通过ProtocolHandler来处理外部来的这行请求的，不同的ProtocoalHandler 代表不同的连接类型，如下图Http11和AJP

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190719174418627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

**ProtocolHandle内部是如何处理的呢？**

内部是通过EndPoint，Processor，Adapter

1. EndPoint通过Socket处理外部连接（TCP/IP）
2. 然后Processor接收到EndPoint的Socket请求（HTTP），将其封装为HttpRequest和HttpResponse

3. 然后将这些请求交给Adapter，然后Adapter处理后将请求交给Container

**Endpoint的内部组成**

`Endpoint`的抽象实现类AbstractEndpoint里面定义了`Acceptor`和`AsyncTimeout`两个内部类和一个`Handler接口`。

`Acceptor`用于监听请求

`Handler`用于处理接收到的Socket，在内部调用`Processor`进行处理。

`AsyncTimeout`用于检查异步Request的超时

### Container的组成

Container正如上面说的是一个Servlet容器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190719140146435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

* `Engine`：最外层容器，用来管理多个站点，并且一个Servcie只能有一个Engine
* `Host`：代表是站点（虚拟主机），可以配置Host来添加站点
* `Context`：代表的是一个应用程序（应用上下文），一般指的是一个war包，包含多个Wrapper。
* `Wrapper`：用来封装Servlet，负责 Servlet 实例的创 建、执行和销毁。



借用网上一张图片

![img](https://mmbiz.qpic.cn/mmbiz_png/UtWdDgynLdZlz44rysyrVpFqrW1Yc0eVzG6ScAiaicp62YI1Co6lOgyNVslAibJc7IHNibVib1S4K06D7bmq10icho8A/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ROOT存放的是主应用，其他放的是子应用。

## Tomcat  Server处理请求

下图是Tomcat Server处理接受到的HTTP请求流程

从外界——>Connector——>Container----->Connector------>响应

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190719141533127.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)
	



## Container处理请求

Container里有那么多层，那Container是怎么处理请求的呢？

Container处理请求是根据涉及模式中的责任链模式，使用Pipeline-Valve管道来处理的

![img](https://img-blog.csdn.net/20180518194127860?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pzajEzMjYzNjkwOTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

如上图，每个人只做和自己相关的，如果不是就向下传递，最后都没有处理的话就会有异常。

具体过程如下：

Pipeline的处理流程图如下（图D）：

![img](https://img-blog.csdn.net/20180518194138321?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pzajEzMjYzNjkwOTE4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（2）在Engine的管道中依次会执行EngineValve1、EngineValve2等等，最后会执行StandardEngineValve，在StandardEngineValve中会调用Host管道，然后再依次执行Host的HostValve1、HostValve2等，最后在执行StandardHostValve，然后再依次调用Context的管道和Wrapper的管道，最后执行到StandardWrapperValve。

（3）当执行到StandardWrapperValve的时候，会在StandardWrapperValve中创建FilterChain，并调用其doFilter方法来处理请求，这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet，其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理！

（4）当所有的Pipeline-Valve都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给Connector了，Connector在通过Socket的方式将结果返回给客户端。



[参考博客](https://blog.csdn.net/jsj13263690918/article/details/80368757)

[参考博客](https://www.jianshu.com/p/3059328cd661)

## Tomcat一些配置

### 修改Tomcat内存

**为什么要修改Tomcat内存？**

日常开发中，开发项目比较大的时候依赖的jar包比较多，并且在应用服务器启动的时候，会将项目引用的所有的类依次全部加载到内存当中，java的逻辑内存模式分为堆内存（存储类的实例，数组、引用类型也就是用new生成的对象）、栈内存（存储局部变量比如方法参数）、静态内存区（持久区，该区内存不会被gc回收，存常量、静态变量、类的源数据：方法属性什么的）

**在开发当中经常遇到的内存溢出的异常**

- **OutOfMemoryErroe:Java heap space异常------>堆内存满了**

  JVM中堆内存的大小默认使用的最小内存是我们物理内存的1/64，最大的使用我们物理内存的1/4，我们通过调整JVM中的初始内存和最大内存来改变我们使用内存的限制

- **OutOfMemoryError:PermGen space异常**

  表示静态内存区满了，通常是因为加载的类太多导致的，jdk8以下的需要修改两个参数限制静态区最小和最大内存范围，，jdk8改变了内存模型，将类定义存放到了源数据空间，而源数据空间与堆内存共享的是同一块内存区域，所以在jdk8版本以后就不会再出现PermGen space异常了。

- **StackOverflowError异常**

  栈内存溢出：通常是由于死循环或无线递归导致的

  

**配置内存参数**

可以在通过下面两种方式来修改Tomcat的内存

1. 修改bin目录下的catalina.bat;
2. 修改bin目录下的startup.bat

```JVM
set JAVA_OPTS="-server -Xms256m -Xmx512m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m"
```

-Xms：java heap初始大小 ，默认为物理内存的1/64，最大不要超过物理内存的80%

-Xmx：java heap的最大值，建议设置为物理内存的一半，不要超过实际的物理内存

MetaspaceSize:初始元空间的值，默认21m，

MetaspaceSize：最大元空间的值，默认无上限

虚拟机的堆大小决定了虚拟机花费在数据垃圾上的时间和频率，调整虚拟机的堆大小目的是最小化垃圾回收的时间，一般用物理内存的80%作为堆内存的大小

### 配置HTTPS

需要在server.xml中配置数字证书和密码

## SSO——单点登录

![1563542490402](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1563542490402.png)

1. 浏览器访问客户端
2. 客户端重定向到CAS服务端
3. 用户认证
4. 认证后随机生成一个Ticket，发送到客户端
5. 客户端带着Ticket访问服务端
6. 验证成功后，把此用户信息返回给客户端

