# Google API Design
Google API Documents Chinese Documents

## 目录
1. [简介](#1-Introduction)
1. [面向资源的设计](#2-Resource-Oriented-Design)



<h1 id="1-Introduction"><code>简介</code></h1>

前言
这是一份适用于网络API的通用指南。本指南自2014 年起在Google 内部使用，并且是我们设计Cloud API 和其它Google API时所遵循的依据。我们将这份指南分享出来供外部的开发者参考，使我们之间的共同开发变得轻松。

外部开发者可能会在设计配合Google Cloud Endpoints 使用的gRPC API 时觉得本指南尤其有用, 且我们强烈推荐此类开发者遵从这些设计原则。不过我们并不强求任何非谷歌的开发者遵循本原则并且你完全可以在不参照本指南的前提下使用Cloud Endpoints 和/或gRPC 。

本指南对REST API 和RPC API 均为适用，并对gRPC API 有特别的关注。gPRC API 使用Protocol Buffers去定义API 表层和API Service Configuration去配置其API 服务，包括HTTP 映射，日志和监控。Google API 和gRPC Cloud Endpoints 使用HTTP 映射功能进行JSON/HTTP 到Protocol Buffers/RPC的转码。

本指南是一份不断变化的文档，不断被采用、接纳的新风格和设计模式会不断地被添加进来。在这种指导精神下，本指南不会终结且在追寻API 设计的艺术及匠心上将一直都会有进步空间。

文档用语
不同级别的要求类词语:
  ● 绝对要求："MUST", "REQUIRED", "SHALL"
  ● 绝对不要："MUST NOT", "SHALL NOT"
  ● 一般应该："SHOULD", "RECOMMENDED"
  ● 一般不要："SHOULD NOT"
  ● 可能，可选 "MAY", "OPTIONAL"

在本文中使用解释参照其在RFC 2119中的描述。
在本文档中，这些关键词由粗体高亮标示。

<h1 id="2-Resource-Oriented-Design"><code>面向资源的设计</code></h1>

本指南的目标是帮助开发者设计出简介、一致且好用的网络API 。与此同时，此指南也有助于统一基于socket 的RPC API和基于HTTP 的REST API 的设计。

长久以来，人们通过API接口和方法，如CORBA 和Windows COM 来设计RPC API。随着时间流逝，越来越多的接口和方法被引入。最终的结果将是数目惊人且各不相同的接口和方法。为了正确的使用它们，开发者不得不得进行仔细的学习，这不仅耗时而且易错。

REST风格体系最早在2000年被提出，并被设计为配合HTTP/1.1工作。REST的核心原则是定义可被少许方法进行操作的命名资源。这些资源和方法被称为API 的名词 (nouns) 和动词 (verb)。在HTTP协议下，资源名很自然地被映射到URL上而方法则映射到HTTP方法POST GET PUT PATCH 和 DELETE上。

在因特网上，HTTP REST API 最近获得了巨大的成功。在2010年，将近74%的公开网络API 是HTTP REST API。

尽管HTTP REST API 在因特网上非常流行，但其传送的流量却少于传统的RPC API。例如：在美国大约一半的高峰期网络流量是视频内容，而由于性能原因，没有人会考虑使用REST API 去传送这些内容。在数据中心内部，许多公司使用基于socket的RPC API 去承载大部分网络流量，而这些流量可能比公开REST API 上的大上几个数量级。

现实中，RPC API 和 HTTP REST API 都有许多不同的使用理由。理想情况下，一个API平台应该为所有的API提供最好的支持。本指南帮助你设计和构造符合此原则的API。其使用面向资源的设计原则去设计范用API，并且规定了许多通用的设计模式去增加可用性、降低复杂度。

注意： 本指南解释了如何在不依赖编程语言，操作系统和网络协议的情况下将REST 原则应用于API 设计。它并不是一份仅仅关于构造REST API 的指南。

什么是REST API ？
REST API 是一系列个体可描述 (individually-addressable) 的资源（API的名词）的模型。资源可以通过他们的资源名称来提及，并可以通过一个小集合内的方法（即API的动词）来操作。

REST Google API 的标准方法（也被称为REST方法）包括List, Get, Create Update 和Delete。当功能不能轻松地映射到标准方法时，如数据库事务，API设计者也可以使用自定义方法（也被称为自定义动词或自定义操作）。

注意： 自定义动词并不意味着创建自定义HTTP动词来实现自定义方法。对于基于HTTP的API，自定义动词会被映射到合适的HTTP动词上。

设计流程
The Style Guide suggests taking the following steps when designing resource- oriented APIs (more details are covered in specific sections below):

本指南建议按照下列步骤来设计面向资源的API（更多细节会在以后具体的章节所描述）。

  ● 确定API提供的资源类型
  ● 查明不同资源间的关系
  ● 根据资源的类型和关系，决定资源名称的规范
  ● 决定资源的范式 (schema)
  ● 为资源加上方法的最小集合

资源 (Resources)

<span style="font-size: 15px; color: rgb(51, 51, 51);">本指南的目标是帮助开发者设计出简介、一致且好用的网络API 。与此同时，此指南也有助于统一基于socket 的RPC API和基于HTTP 的REST API 的设计。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">长久以来，人们通过API接口和方法，如CORBA 和Windows COM 来设计RPC API。随着时间流逝，越来越多的接口和方法被引入。最终的结果将是数目惊人且各不相同的接口和方法。为了正确的使用它们，开发者不得不得进行仔细的学习，这不仅耗时而且易错。</span>

[<span style="font-size: 15px; color: rgb(0, 154, 97);">REST</span>](http://en.wikipedia.org/wiki/Representational_state_transfer)<span style="font-size: 15px; color: rgb(51, 51, 51);">风格体系最早在2000年被提出，并被设计为配合HTTP/1.1工作。REST的核心原则是定义可被少许方法进行操作的命名资源。这些资源和方法被称为API 的名词 (nouns) 和动词 (verb)。在HTTP协议下，资源名很自然地被映射到URL上而方法则映射到HTTP方法</span><span style="font-size: 13px; color: rgb(199, 37, 78);">POST</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> </span><span style="font-size: 13px; color: rgb(199, 37, 78);">GET</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> </span><span style="font-size: 13px; color: rgb(199, 37, 78);">PUT</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> </span><span style="font-size: 13px; color: rgb(199, 37, 78);">PATCH</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">DELETE</span><span style="font-size: 15px; color: rgb(51, 51, 51);">上。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在因特网上，HTTP REST API 最近获得了巨大的成功。在2010年，将近74%的公开网络API 是HTTP REST API。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">尽管HTTP REST API 在因特网上非常流行，但其传送的流量却少于传统的RPC API。例如：在美国大约一半的高峰期网络流量是视频内容，而由于性能原因，没有人会考虑使用REST API 去传送这些内容。在数据中心内部，许多公司使用基于socket的RPC API 去承载大部分网络流量，而这些流量可能比公开REST API 上的大上几个数量级。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">现实中，RPC API 和 HTTP REST API 都有许多不同的使用理由。理想情况下，一个API平台应该为所有的API提供最好的支持。本指南帮助你设计和构造符合此原则的API。其使用面向资源的设计原则去设计范用API，并且规定了许多通用的设计模式去增加可用性、降低复杂度。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意：</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 本指南解释了如何在不依赖编程语言，操作系统和网络协议的情况下将REST 原则应用于API 设计。它</span><span style="font-size: 15px; color: rgb(51, 51, 51);">并不是</span><span style="font-size: 15px; color: rgb(51, 51, 51);">一份仅仅关于构造REST API 的指南。</span>

<span style="font-size: 21px; color: rgb(51, 51, 51);">什么是REST API ？</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">REST API 是一系列个体可描述 (individually-addressable) 的资源（API的名词）的模型。资源可以通过他们的</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">资源名称</span>](https://cloud.google.com/apis/design/resource_names)<span style="font-size: 15px; color: rgb(51, 51, 51);">来提及，并可以通过一个小集合内的方法（即API的动词）来操作。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">REST Google API 的标准方法（也被称为REST方法）包括List, Get, Create Update 和Delete。当功能不能轻松地映射到标准方法时，如数据库事务，API设计者也可以使用自定义方法（也被称为自定义动词或自定义操作）。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意：</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 自定义动词并不意味着创建自定义HTTP动词来实现自定义方法。对于基于HTTP的API，自定义动词会被映射到合适的HTTP动词上。</span>

<span style="font-size: 21px; color: rgb(51, 51, 51);">设计流程</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">The Style Guide suggests taking the following steps when designing resource- oriented APIs (more details are covered in specific sections below):</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">本指南建议按照下列步骤来设计面向资源的API（更多细节会在以后具体的章节所描述）。</span>

*   确定API提供的资源类型
*   查明不同资源间的关系
*   根据资源的类型和关系，决定资源名称的规范
*   决定资源的范式 (schema)
*   为资源加上方法的最小集合

<span style="font-size: 21px; color: rgb(51, 51, 51);">资源 (Resources)</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">面向资源的API通常按照资源阶层进行建模，其中每一个节点可以是单个简单资源或者是一个资源集合。为了方便，他们通常被分别称为一个资源或者一个集合。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">一个集合含有一系列相同类型的资源。比如，一个用户拥有一个联系人集合。一个资源拥有一些状态以及0个或多个子资源 (sub-resource)。每个子资源可以是简单资源或者是资源集合。举例来说，Gmail API 有一个用户资源集合，其中每个用户拥有消息集合，帖子集合，标签集合，一个用户资料资源和若干个用户设置资源。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">尽管在存储系统和REST API 之间有一些概念上的一致性，但提供面向资源的API 的服务却不一定要是一个数据库，并且其在解释资源资源和方法时用于很大的灵活性。例如，创建一个日历时间（资源）可能会为与会者创建额外的时间，发送邮件邀请给与会者，预定会议室并更新视频会议日程。</span>

<span style="font-size: 21px; color: rgb(51, 51, 51);">方法（Methods)</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">面向资源的API 的关键特点是它强调资源（数据模型）甚于作用于资源的方法（功能性）。一个典型的面向资源的API 会暴露大量的仅具有少数方法的资源。方法可以是标准方法，也可以是自定义方法。对于本指南，标准方法是：List, Get, Greate, Update 和Delete。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">当API 的功能可以自然地映射到一种标准方法时，该方法应该在API 设计时被使用。对于不能轻易地映射到某个标准方法上的功能，可以使用自定义方法。自定义方法提供了和设计传统RPC API相近的自由度，从而可以用来实现编程模式，如数据库事务或者数据分析。</span>

<span style="font-size: 21px; color: rgb(51, 51, 51);">示例</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">接下来的章节通过一些实际的例子展示了如果和对于大规模的服务使用基于资源的API 设计。</span>

<span style="font-size: 17px; color: rgb(51, 51, 51);">Gmail API</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">Gmail API 服务实现了Gmail API 并向使用者暴露了大部分Gmail 的功能。其定义了下列资源模型：</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">Gmail API 服务: gmail.googleapis.com</span>

*   用户集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/</span>_ 每个用户又拥有下列资源：_

*   _消息资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/</span>_/messages/

*   _用户帖子资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/</span>_/threads/
*   _标签资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/</span>_/labels/
*   _修改历史资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/</span>_/history/
*   _代表用户资料的资源: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/</span>_/profile
*   代表用户设置的资源: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/_/settings_</span>

_<span style="font-size: 17px; color: rgb(51, 51, 51);">Google Cloud Pub/Sub API</span>_

_<span style="font-size: 15px; color: rgb(51, 51, 51);">pubsub.googleapis.com 服务实现了Google Cloud Pub/Sub API, 其定义了下列资源模型：</span>_

*   _API服务: pubsub.googleapis.com_
*   _主题资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">projects/</span>_/topics/
*   _订阅资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">projects/</span>_/subscriptions/*

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意：</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 其它Pub/Sub API 实现可能采用不同的资源名称范式</span>
