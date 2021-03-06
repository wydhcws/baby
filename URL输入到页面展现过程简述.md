#####在浏览器中输入URL到整个页面显示在用户面前这个过程大致可以被分为以下几个阶段：
1. 域名解析
2. 服务器处理
3. 网站处理
4. 浏览器处理与绘制

#####URL的定义

   URL（Uniform Resource Locator），即统一资源定位符，是对可以从互联网上得到的资源的位置和访问方法的一种简洁的表示，是互联网上标准资源的地址（网址）。互联网上的每个文件都有一个唯一的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。

基本URL包含模式（或称协议）、服务器名称（或IP地址）、路径和文件名，如“协议://授权/路径查询”。完整的、带有授权部分的普通统一资源标志符语法看上去如下：协议://用户名:密码@子域名.域名.顶级域名:端口号/目录/文件名.文件后缀?参数=值#标志

URL常用协议有:
- http——超文本传输协议资源       
- https——用安全套接字层传送的超文本传输协议（加密）
- ftp——文件传输协议
- mailto——电子邮件地址
- file——当地电脑或网上分享的文件

#具体过程介绍

#一、域名解析
互联网上每一台计算机的唯一标识是它的IP地址，但是IP地址并不方便记忆。用户更喜欢用方便记忆的网址去寻找互联网上的其它计算机。所以互联网设计者需要在用户的方便性与可用性方面做一个权衡，这个权衡就是一个网址到IP地址的转换，这个过程就是域名解析。它实际上充当了一个翻译的角色，实现了网址到IP地址的转换。域名的解析工作由DNS服务器完成。

**IP 地址**：IP 协议为互联网上的每一个网络和每一台主机分配的一个逻辑地址。IP 地址如同门牌号码，通过 IP 地址才能确定一台主机位置。服务器本质也是一台主机，想要访问某个服务器，必须先知道它的 IP 地址

**域名（ DN ）**：IP 地址由四个数字组成，中间用点号连接，在使用过程中难记忆且易输入错误，所以用我们熟悉的字母和数字组合来代替纯数字的 IP 地址，比如我们只会记住 www.baidu.com（百度域名） 而不是 220.181.112.244（百度的其中一个 IP 地址）。

**DNS**： 每个域名都对应一个或多个提供相同服务服务器的 IP 地址，只有知道服务器 IP 地址才能建立连接，所以需要通过 DNS 把域名解析成一个 IP 地址。
####具体流程
1. 浏览器搜索自己的DNS缓存 – 浏览器会缓存DNS记录一段时间
2. 搜索操作系统中的Hosts文件 - 从 Hosts 文件查找是否有该域名和对应 IP。
3. 搜索路由器缓存 – 一般路由器也会缓存域名信息。
4. 搜索ISP DNS 缓存 （ISP:互联网服务提供商）– 比如到电信的 DNS 上查找缓存。
如果都没有找到，则向根域名服务器查找域名对应 IP，根域名服务器把请求转发到下一级，直到找到 IP

**小知识**：
1.  8.8.8.8 ：8.8.8.8是一个IP地址，是Google提供的免费DNS服务器的IP地址
    114.114.114.114 ：是国内第一个、全球第三个开放的DNS服务地址,又称114DNS
2. DNS劫持：
DNS劫持又称域名劫持，是指在劫持的网络范围内拦截域名解析的请求，分析请求的域名，把审查范围以外的请求放行，否则返回假的IP地址或者什么都不做使请求失去响应，其效果就是对特定的网络不能反应或访问的是假网址。

#二、服务器处理
服务器是一台安装系统的机器，常见的系统如Linux、windows server 2012等系统里安装的处理请求的应用叫 **Web server**（web服务器）
常见的 web服务器有 Apache、Nginx、IIS、Lighttpd
web服务器接收用户的Request 交给网站代码，或者接受请求反向代理到其他 web服务器

 >![ ** 饥人谷服务器处理过程示意图**](http://upload-images.jianshu.io/upload_images/1347915-8ab232409366510a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#三、网站处理
>![**MVC架构网站处理示意图**.png](http://upload-images.jianshu.io/upload_images/1347915-e72d0d502efd551a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**MVC**全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

- Model（模型）：是应用程序中用于处理应用程序数据逻辑的部分，通常模型对象负责在数据库中存取数据。
- View（视图）：是应用程序中处理数据显示的部分。，通常视图是依据模型数据创建的。（**前端工程师主要负责**）
- Controller（控制器）：是应用程序中处理用户交互的部分，通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

#四、浏览器处理与绘制

解析过程：
- HTML字符串被浏览器接受后被一句句读取解析
- 解析到link 标签后重新发送请求获取css；解析到 script标签后发送请求获取 js，并执行代码
- 解析到img 标签后发送请求获取图片资源

绘制过程：
- 浏览器根据 HTML 和 CSS 计算得到渲染树，绘制到屏幕上，js 会被执行

解析顺序：
- 浏览器是一个边解析边渲染的过程。首先浏览器解析HTML文件构建DOM树，然后解析CSS文件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上。
- JS的解析是由浏览器中的JS解析引擎完成的。JS是单线程运行，也就是说，在同一个时间内只能做一件事，所有的任务都需要排队，前一个任务结束，后一个任务才能开始。
- 浏览器在解析过程中，如果遇到请求外部资源时请求过程是异步的，并不会影响HTML文档进行加载，但是当文档加载过程中遇到JS文件，HTML文档会挂起渲染过程，不仅要等到文档中JS文件加载完毕还要等待解析执行完毕，才会继续HTML的渲染过程。原因是因为JS有可能修改DOM结构，这就意味着JS执行完成前，后续所有资源的下载是没有必要的，这就是JS阻塞后续资源下载的根本原因。CSS文件的加载不影响JS文件的加载，但是却影响JS文件的执行。JS代码执行前浏览器必须保证CSS文件已经下载并加载完毕。