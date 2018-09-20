# Google API Design
Google API Documents Chinese Documents

## 目录
1. [简介](#1-Introduction)
2. [面向资源的设计](#2-Resource-Oriented-Design)
3. [资源名称](#3-Resource-Name)
4. [标准方法](#4-Standard-Methods)
5. [自定义方法](#5-Custom-Methods)
6. [错误处理](#6-Errors)
7. [命名规范](#7Naming-Conventions)
8. [设计模式](#8Common-Design-Patterns)
9. [使用 Proto3](#9Protocol-Buffers-v3)
10. [版本管理](#10Versioning)
11. [兼容性](#11Compatibility)
12. [目录结构](#12Directory-Structure)
13. [文件结构](#13File-Structure)



<h1 id="1-Introduction"><code>简介</code></h1>

<span style="font-size: 24px; color: rgb(51, 51, 51);">前言</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">这是一份适用于网络API的通用指南。本指南自2014 年起在Google 内部使用，并且是我们设计</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Cloud API</span>](https://cloud.google.com/apis/docs/overview)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 和其它</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Google API</span>](https://cloud.google.com/apis/docs/overview)<span style="font-size: 15px; color: rgb(51, 51, 51);">时所遵循的依据。我们将这份指南分享出来供外部的开发者参考，使我们之间的共同开发变得轻松。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">外部开发者可能会在设计配合</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Google Cloud Endpoints</span>](https://cloud.google.com/endpoints/docs/grpc)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用的gRPC API 时觉得本指南尤其有用, 且我们强烈推荐此类开发者遵从这些设计原则。不过我们并不强求任何非谷歌的开发者遵循本原则并且你完全可以在不参照本指南的前提下使用Cloud Endpoints 和/或gRPC 。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">本指南对REST API 和RPC API 均为适用，并对gRPC API 有特别的关注。gPRC API 使用</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Protocol Buffers</span>](https://cloud.google.com/apis/design/proto3)<span style="font-size: 15px; color: rgb(51, 51, 51);">去定义API 表层和</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">API Service Configuration</span>](https://github.com/googleapis/googleapis)<span style="font-size: 15px; color: rgb(51, 51, 51);">去配置其API 服务，包括HTTP 映射，日志和监控。Google API 和gRPC Cloud Endpoints 使用HTTP 映射功能进行JSON/HTTP 到Protocol Buffers/RPC的</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">转码</span>](https://cloud.google.com/endpoints/docs/transcoding)<span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">本指南是一份不断变化的文档，不断被采用、接纳的新风格和设计模式会不断地被添加进来。在这种指导精神下，本指南不会终结且在追寻API 设计的艺术及匠心上将一直都会有进步空间。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">文档用语</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">不同级别的要求类词语:</span>

*   绝对要求："MUST", "REQUIRED", "SHALL"
*   绝对不要："MUST NOT", "SHALL NOT"
*   一般应该："SHOULD", "RECOMMENDED"
*   一般不要："SHOULD NOT"
*   可能，可选 "MAY", "OPTIONAL"

<span style="font-size: 15px; color: rgb(51, 51, 51);">在本文中使用解释参照其在</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">RFC 2119</span>](https://www.ietf.org/rfc/rfc2119.txt)<span style="font-size: 15px; color: rgb(51, 51, 51);">中的描述。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在本文档中，这些关键词由</span><span style="font-size: 15px; color: rgb(51, 51, 51);">粗体</span><span style="font-size: 15px; color: rgb(51, 51, 51);">高亮标示。</span>

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

<h1 id="3-Resource-Name"><code>资源名称</code></h1>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在面向资源的 API 中，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">资源</span><span style="font-size: 15px; color: rgb(51, 51, 51);">是命名实体，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">资源名称</span><span style="font-size: 15px; color: rgb(51, 51, 51);">是其标识符。每个资源 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（ MUST ）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 有唯一的资源名称。资源名称由资源自己的 ID，任一父资源的 ID 及其 API 服务名称组成。下面我们将看一看资源 ID 和资源名是如何构成的。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">gRPC API 应该为资源名使用无协议（scheme-less）的 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">URI</span>](http://tools.ietf.org/html/rfc3986)<span style="font-size: 15px; color: rgb(51, 51, 51);">。它们通常遵循 REST URL 惯例并且其行为与网络文件路径非常相似。它们能非常容易地映射到 REST API：详细内容查看</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">标准方法</span>](http://tailnode.tk/2017/03/google-api-design-guide/standard-methods/)<span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">集合</span><span style="font-size: 15px; color: rgb(51, 51, 51);">是一种特殊类型的资源，它包含了相同类型子资源的列表。例如，目录是文件资源的集合。集合的资源 ID 叫做集合 ID。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">资源名称由集合 ID 和资源 ID 按层次组织形成，并以斜杠（/）分隔。如果资源包含子资源，子资源名称的格式是父资源名称后面加上子资源 ID，同样地使用斜杠分隔。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">例 1：一个存储服务具有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">buckets</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 集合，每个 bucket 具有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">objects</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 集合：</span>

| <span style="color: rgb(0, 0, 0);">API 服务名</span> | <span style="color: rgb(0, 0, 0);">集合 ID</span> | <span style="color: rgb(0, 0, 0);">资源 ID</span> | <span style="color: rgb(0, 0, 0);">集合 ID</span> | <span style="color: rgb(0, 0, 0);">资源 ID</span> |
| --- | --- | --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">//storage.googleapis.com</span> | <span style="color: rgb(0, 0, 0);">/buckets</span> | <span style="color: rgb(0, 0, 0);">/bucket-id</span> | <span style="color: rgb(0, 0, 0);">/objects</span> | <span style="color: rgb(0, 0, 0);">/object-id</span> |

<span style="font-size: 15px; color: rgb(51, 51, 51);">例 2：一个具有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">users</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 集合的邮件服务，每个用户具有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">settings</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 子资源， </span><span style="font-size: 13px; color: rgb(199, 37, 78);">settings</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 子资源具有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">customFrom</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和另外的子资源：</span>

| <span style="color: rgb(0, 0, 0);">API 服务名</span> | <span style="color: rgb(0, 0, 0);">集合 ID</span> | <span style="color: rgb(0, 0, 0);">资源 ID</span> | <span style="color: rgb(0, 0, 0);">资源 ID</span> | <span style="color: rgb(0, 0, 0);">资源 ID</span> |
| --- | --- | --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">//mail.googleapis.com</span> | <span style="color: rgb(0, 0, 0);">/users</span> | <span style="color: rgb(0, 0, 0);">/name@example.com</span> | <span style="color: rgb(0, 0, 0);">/settings</span> | <span style="color: rgb(0, 0, 0);">/customFrom</span> |

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 设计者可以为资源和集合 ID 选择任何可接受的值，只要它们在资源层次结构中是唯一的即可。你可以在下面找到有关选择适当资源和集合 ID 的更多指南。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">完整资源名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">无协议（scheme-less） </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">URI</span>](http://tools.ietf.org/html/rfc3986)<span style="font-size: 15px; color: rgb(51, 51, 51);">由</span> [<span style="font-size: 15px; color: rgb(0, 154, 97);">兼容 DNS</span>](http://tools.ietf.org/html/rfc1035)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 的 API 服务名和资源路径组成。资源路径也称为</span><span style="font-size: 15px; color: rgb(51, 51, 51);">相对资源名</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。例如：</span>

<span style="font-size: 16px; color: rgb(221, 17, 68);">"//library.googleapis.com/shelves/shelf1/books/book2"</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 服务名用于客户端定位 API 服务端点，如果只为内部服务，它</span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">是假的 DNS 名。如果 API 服务名在上下文中显而易见的话则会经常使用相对资源名。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">相对资源名</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">没有斜杠（/）开头的 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">URI 路径</span>](http://tools.ietf.org/html/rfc3986#appendix-A)<span style="font-size: 15px; color: rgb(51, 51, 51);">标识了 API 服务中的资源。例如：</span>

<span style="font-size: 13px; color: rgb(221, 17, 68);">"shelves/shelf1/books/book2"</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">资源 ID</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">使用非空的 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">URI 段</span>](http://tools.ietf.org/html/rfc3986#appendix-A)<span style="font-size: 15px; color: rgb(51, 51, 51);">标识其父资源中的资源。请看上面的例子。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">资源名称后面跟随的资源 ID </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 具有不只一个 URI 段，例如：</span>

| <span style="color: rgb(0, 0, 0);">集合 ID</span> | <span style="color: rgb(0, 0, 0);">资源 ID</span> |
| --- | --- |
| <span style="color: rgb(0, 0, 0);">files</span> | <span style="color: rgb(0, 0, 0);">/source/py/parser.py</span> |

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果可以的话，API 服务</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">使用 URL 友好的资源 ID。资源 ID </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 明确地记录在文档中，不管它们是由客户端还是服务端分配的。例如，文件名一般由客户端分配，而邮件信息 ID 一般由服务端分配。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">集合 ID</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">使用非空的 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">URI 段</span>](http://tools.ietf.org/html/rfc3986#appendix-A)<span style="font-size: 15px; color: rgb(51, 51, 51);">标识其父资源中的资源集合。请看上面的例子。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">因为集合 ID 经常出现在生成的客户端库中，它们 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 符合以下要求：</span>

*   必须（must） 是合法的 C/C++ 标识符
*   必须（must） 是复数形式的首字母小写的驼峰命名
*   必须（must） 使用清晰简明的英语词汇
*   应该（should） 避免或限定过于笼统的术语。例如：<span style="font-size: 13px; color: rgb(199, 37, 78);">RowValue</span> 优于 <span style="font-size: 13px; color: rgb(199, 37, 78);">Value</span>。除非明确定义，否则 应该（should） 避免使用如下术语：

*   Element
*   Entry
*   Instance
*   Item
*   Object
*   Resource
*   Type
*   Value

<span style="font-size: 28px; color: rgb(51, 51, 51);">资源名 vs URL</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">完整的资源名类似普通的 URL，但它们并不相同。同样的资源能够通过不同版本或不同协议的 API 来暴露出去。完整的资源名并没有指定这些信息，所以必须将它映射到特定的协议和 API 版本上才能直接地使用。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了通过 REST API 使用完整的资源名，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用如下方法将其映射为 REST URL：在服务名前添加 HTTPS 协议、在资源路径前添加 API 主版本号、将资源路径进行 URL 转义。例如：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">/</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/ 这是日历事件的资源名"//calendar.googleapis.com/users/john smith/events/123"// 这是对应的 HTTP URL"[https://calendar.googleapis.com/v](https://calendar.googleapis.com/v)</span><span style="font-size: 13px; color: rgb(51, 51, 51);">3/users/john%</span><span style="font-size: 13px; color: rgb(0, 128, 128);">20</span><span style="font-size: 13px; color: rgb(51, 51, 51);">smith/events/</span><span style="font-size: 13px; color: rgb(0, 128, 128);">123</span><span style="font-size: 13px; color: rgb(221, 17, 68);">"</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">资源名做为字符串</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">除非有向后兼容的问题，Google API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用字符串来表示资源名。资源名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该(should)</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 像普通文件路径那样处理，并且不支持</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">百分号编码</span>](https://zh.wikipedia.org/zh-hans/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81)<span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">对于资源定义，第一个字段 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是资源名称的字符串字段，它 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 叫作 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">name</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：像  </span><span style="font-size: 13px; color: rgb(199, 37, 78);">display_name</span><span style="font-size: 15px; color: rgb(51, 51, 51);">、</span><span style="font-size: 13px; color: rgb(199, 37, 78);">first_name</span><span style="font-size: 15px; color: rgb(51, 51, 51);">、</span><span style="font-size: 13px; color: rgb(199, 37, 78);">last_name</span><span style="font-size: 15px; color: rgb(51, 51, 51);">、</span><span style="font-size: 13px; color: rgb(199, 37, 78);">full_name</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  这种与名字相关的字段 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 给出定义来避免混乱。</span>


