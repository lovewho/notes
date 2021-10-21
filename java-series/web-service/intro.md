# Web Service

## 1. What is service

传统上，我们把计算机组件或应用程序提供的某种功能称为服务。根据来源的不同，"服务" 又可以分成两种：

- 本地服务（不需要访问 Internet，由本地资源提供）
- 网络服务 (通过 internet 访问其他站点提供的服务)

也就是说 “网络服务”（Web Service）的本质，就是通过网络去获取其他网站的资源。如 “天气预报” 功能，我们本身没有关于天气预报的相关信息，但是可以通过调用别人提供的网络服务来获取这些信息。

网络服务通常是与应用程序绑定的，应用程序对外提供的接口我们都可以看作 Web 服务，也就是说我们可以把 Web 服务看作是一种通过网络在不同平台之间交换消息的方式，为了保证不同平台的应用服务可以互相操作，所有的 Web 服务都应该以标准的 Web 协议提供服务。

## 2. What's is Web Service

Web Service 也叫 [XML Web Service](https://www.w3schools.com/xml/xml_services.asp)，它是一种对外提供服务的方式，或者说它是一种对外提供服务的标准，由 W3C 定义。它可以接收 Internet 中的请求，并使用 SOAP 协议交换数据。是一种跨语言和跨操作系统平台的远程调用技术。它具有以下特性：

- 可通过 Internet 或专用网络使用
- 使用标准的 XML 消息传递系统
- 不依赖于任何一种操作系统或编程语言
- 通过通用 XML 语法进行自我描述
- 可通过简单的查找机制发现

### 2.1. Core Component

基本的 Web Service 平台是 XML + HTTP，通常包括以下核心组件：

- UDDI: 统一描述、发现和集成(Universal Description, Discovery and Integration)的缩写。它是一个基于 XML 	的跨平台的描述规范，可以使世界范围内的企业在互联网上发布自己所提供的服务。相当于注册中心，用于管理、发布、查询 Web 服务。
- WSDL: Web 服务描述语言(Web Service Description Language)。一个XML格式文档，用以描述服务端口访问方式和使用协议的细节。通常用来辅助生成服务器和客户端代码及配置信息。
- SOAP: 简单对象访问协议(Simple Object Access Protocol)。是一种基于 XML 的协议，用于在 Web 上交换结构化数据。需同时绑定一个网络传输协议。这个协议通常是 HTTP 或 HTTPS，但也可能是 SMTP 或 XMPP。

### 2.2.  Architecture

Web 服务的体系结构是基于Web服务提供者、Web服务请求者、Web服务中介者三个角色和发布、发现、绑定三个动作构建的。

简单地说，Web 服务提供者就是 Web 服务的拥有者，它耐心等待为其他服务和用户提供自己已有的功能；Web 服务请求者就是Web 服务功能的使用者，它利用 SOAP 协议向 Web 服务提供者发送请求以获得服务；Web 服务中介者的作用是把一个 Web 服务请求者与合适的 Web 服务提供者联系在一起，它充当管理者的角色，一般是UDDI。

这三个角色是根据逻辑关系划分的，在实际应用中，角色之间很可能有交叉：一个 Web 服务既可以是 Web 服务提供者，也可以是Web 服务请求者，或者二者兼而有之。

一个完整的 Web Service 工作流程如下。

<img src="../../assets/java/web-service-work-flow.png" style="zoom:50%;" />

- 发布：Web 服务提供者将需要向外提供的服务注册（发布）到 UDDI 上。
- 发现：Web 服务请求者向Web服务中介者（UDDI）请求特定的服务，中介者根据请求信息为请求者寻找相应的服务。
- 发现：Web服务中介者向 Web 服务请求者返回满足条件的Web服务描述信息，该描述信息用WSDL写成，各种支持Web服务的机器都能阅读。
- 绑定：利用从 Web 服务中介者返回的描述信息生成相应的 SOAP 消息，发送给Web服务提供者，以实现Web服务的调用。
- 绑定：Web服务提供者按 SOAP 消息执行相应的Web服务，并将服务结果返回给 Web 服务请求者。

## 3. Java Implemention

- [JAX-WS](https://javaee.github.io/tutorial/jaxws.html)：用于构建 Web 服务 的 Java API（Java API for XML Web Services），是随着 JAVA 6 一起发布的。通过 Java 注解机制来简化 Web 服务客户端和服务端的开发和部署。
- [CXF](https://cxf.apache.org/)：CXF 是 Apache 提供的一个开源服务框架，它提供了 JAX-WS 的全面支持，除了 JAX-WS 标准定义的特性之外，Apache CXF 还提供了 WSDL 和 Java 类之间的转换功能、用于操作原始 XML 消息的 API、对 JAX-RS 的支持、与 Spring 框架的集成等，使用 CXF 您能以更简单的方式构建全面、强大的 Web 服务。

  










