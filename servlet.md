# servlet

## 什么是servlet

### 一、servlet的功能

1. 读取客户提交的数据
2. 读取由浏览器发送的隐式请求数据
3. 生成结果
4. 向客户发送显式数据
5. 发送隐式的HTTP响应数据

### 二、servlet的工作原理





![img](http://wenwen.soso.com/p/20111014/20111014082225-1164637525.jpg)

### 三、servlet的生命周期

1. 初始阶段：调用**init**方法。servlet对象被创建时，由容器调用此方法对该对象进行初始化
2. 响应客户请求阶段：调用**service**方法。在客户请求到达时，容器调用此方法完成对请求阶段的处理和响应。要注意的是，在**service**方法被调用之前，必须确保 **init**方法被正确调用
3. 终止阶段：调用**destroy**方法。当容器检测到一个servlet对象应该从服务器中移除时，会调用此方法完成servlet对象被销毁前的收尾工作



## servlet与客户端通信

### 一、servlet生成纯文本

servlet通过**response**对象对客户端做出响应，但是servlet中没有传入输出流对象**out**。若要向客户端输出消息，首先要通过**response**对象获得输出流对象

`printWriter out = response.getWriter()`



### 二、servlet 生成HTML

### 三、接收客户提交参数

servlet 利用**HTTPServletRequest** 对象的**getparameter**方法接收单个值参数，利用**getParameterValues**接收成组参数

### 四、session对象

**session**内置对象的工作机制是使用**Cookie**，在每次客户端发送请求时将jsessionid值加入HTTP头部发送给服务器

`HttpSession session = request.getSession();`

### 五、servlet 上下文

**application** 内置对象是**servletContext**接口的实例，即web应用环境。web应用的基本信息都存储在这个**servletContext**对象中。

`ServletContext context = getServletContext();`

### 六、servlet 的请求转发

在servlet中想要实现jsp中的include动作和forward动作，需要使用**RequestDispatcher** 对象

**RequestDispatcher**对象称为请求转发对象。在servlet中，利用**RequestDispatcher**对象，可以将请求转发给另外一个servlet或jsp页面，**甚至HTML页面**，来处理对请求的响应

`RequestDispatcher dispatcher = request.getRequestDispatcher("Question.jsp");`

生成的dispatcher对象即是目的为Question.jsp的请求转发对象。其中，Question.jsp不包含任何路径，表示与当前servlet映射的URL在同一路径下

RequestDispatcher接口中定义了两种方法用于请求转发

1. `void forward(ServletRequest request, ServletResponse response)`

   forward 方法将请求转发给服务器上另外一个Servlet、jsp或者html文件，这个方法必须在响应被提交给客户端之前调用，否则抛出异常。forward方法调用后在servlet的响应缓存中没有提交的内容将自动消除。即原Servlet的输出不会被返回给客户端

2. `void include(ServletRequest request, ServletResponse response)`

   include 方法用于在响应中包含其他资源（servlet、jsp、html）的内容。即请求转发后，原先的servlet还可以继续输出响应信息，被包含的资源做出的响应将并入原先servlet的响应对象中