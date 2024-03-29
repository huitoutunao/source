# HTTP 基本认识

## HTTP 是什么

超文本传输协议，即 HTTP 是一个在计算机世界里专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范。

## HTTP 演变

20 世纪 90 年代初期，HTTP/0.9 版本诞生了，主要用于学术交流，需求很简单——用来在网络之间传递 HTML 超文本的内容，所以只允许用“GET”请求从服务器上获取 HTML 文档，并且在响应请求之后立即关闭连接，功能非常有限。

1996 年正式发布 HTTP/1.0 版本，更新内容如下：

1. 增加了 HEAD、POST 等新方法。
2. 增加了响应状态码，标记可能的错误原因。
3. 引入了协议版本号概念。
4. 引入了 HTTP Header（头部）的概念，让 HTTP 处理请求和响应更加灵活。
5. 传输的数据不再仅限于文本。

1999 年更新了 HTTP/1.1 版本，更新内容如下：

1. 增加了 PUT、DELETE 等新的方法。
2. 增加了缓存管理和控制。
3. 明确了连接管理，允许持久连接。
4. 允许响应数据分块（chunked），利于传输大文件。
5. 强制要求 Host 头，让互联网主机托管成为可能。

2015 年发布了 HTTP/2 版本，更新内容主要如下：

1. 二进制协议，不再是纯文本。
2. 可发起多个请求，废弃了 1.1 里的管道。
3. 使用专用算法压缩头部，减少数据传输量。
4. 允许服务器主动向客户端推送数据。
5. 增强了安全性，“事实上”要求加密通信。

## HTTP 报文结构

它由三部分组成：

1. 起始行：描述请求或相应的基本信息。
2. 头部字段集合：使用 `key-value` 的形式更详细地说明报文。
3. 消息正文：实际传输的数据，它不一定是纯文本，可以是图片、视频等二进制数据。

其中 1 和 2 通常合称为请求头或响应头（header），3 称为实体（body），完整的 HTTP 报文就像下面这样：

1. 起始行。
2. 头部。
3. 空行。
4. 实体。

### 起始行

请求报文的起始行称为请求行，它由三部分构成：

- 请求方法：如 `GET/POST` 等表示对资源的操作。
- 请求目标：通常是一个 URI，标记了要操作的资源。
- 版本号：HTTP 协议版本。

响应报文的起始行称为状态行，它由三部分构成：

- 版本号：HTTP 协议版本。
- 状态码：一个三位数，使用代码的形式表示处理结果。
- 原因：详细的解释说明。

### 常用的头字段

- Host 字段，它属于请求字段，只能出现在请求头里，它同时也是唯一一个 `HTTP/1.1` 规范里要求必须出现的字段，也就是说，如果请求头里没有 Host，那这就是一个错误的报文。
- `User-Agent` 是请求字段，只出现在请求头里。它使用一个字符串来描述发起 HTTP 请求的客户端，服务器可以依据它来返回最合适此浏览器显示的页面。
- Date 字段是一个通用字段，但通常出现在响应头里，表示 HTTP 报文创建的时间，客户端可以使用这个时间再搭配其他字段决定缓存策略。
- Server 字段是响应字段，只能出现在响应头里。它告诉客户端当前正在提供 Web 服务的软件名称和版本号。
- `Content-Length`，它表示报文里 body 的长度，也就是请求头或响应头空行后面数据的长度。

### 请求方法

1. GET：获取资源，可以理解为读取或者下载数据。
2. HEAD：获取资源的元信息。
3. POST：向资源提交数据（“新建”），相当于写入或上传数据。
4. PUT：类似 POST，“更新/修改”。
5. DELETE：删除资源。
6. CONNECT：建立特殊的连接隧道。
7. OPTIONS：列出可对资源实行的方法。
8. TRACE：追踪请求 - 响应的传输路径。

### 响应状态码

- 1××：提示信息，表示目前是协议处理的中间状态，还需要后续的操作。
- 2××：成功，报文已经收到并被正确处理。
- 3××：重定向，资源位置发生变动，需要客户端重新发送请求。
- 4××：客户端错误，请求报文有误，服务器无法处理。
- 5××：服务器错误，服务器在处理请求时内部发生了错误。

301：永久重定向，意思是此次请求的资源已经不存在了，需要改用新的 URI 再次访问。

302：临时重定向，意思是请求的资源还在，但需要暂时用另一个 URI 来访问。

## 结语

本文到这里就结束了。这是学习[《透视HTTP协议》](https://time.geekbang.org/column/intro/100029001?tab=catalog)罗剑锋老师课程的笔记。希望可以帮到你。
