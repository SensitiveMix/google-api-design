# Google API Design
Google API Documents Chinese Documents

## 目录
1. [简介](#1-Introduction)
2. [面向资源的设计](#2-Resource-Oriented-Design)
3. [资源名称](#3-Resource-Name)
4. [标准方法](#4-Standard-Methods)
5. [自定义方法](#5-Custom-Methods)
6. [错误处理](#6-Errors)
7. [命名规范](#7-Naming-Conventions)
8. [设计模式](#8-Common-Design-Patterns)
9. [使用 Proto3](#9-Protocol-Buffers-v3)
10. [版本管理](#10-Versioning)
11. [兼容性](#11-Compatibility)
12. [目录结构](#12-Directory-Structure)
13. [文件结构](#13-File-Structure)



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

<span style="font-size: 24px; color: rgb(51, 51, 51);">面向资源的设计</span>

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

*   用户集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*</span> 每个用户又拥有下列资源：

*   消息资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*/messages/*</span>
*   用户帖子资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*/threads/*</span>
*   标签资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*/labels/*</span>
*   修改历史资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*/history/*</span>
*   代表用户资料的资源: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*/profile</span>
*   代表用户设置的资源: <span style="font-size: 13px; color: rgb(199, 37, 78);">users/*/settings</span>

<span style="font-size: 17px; color: rgb(51, 51, 51);">Google Cloud Pub/Sub API</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">pubsub.googleapis.com 服务实现了Google Cloud Pub/Sub API, 其定义了下列资源模型：</span>

*   API服务: pubsub.googleapis.com
*   主题资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">projects/*/topics/*</span>
*   订阅资源集合: <span style="font-size: 13px; color: rgb(199, 37, 78);">projects/*/subscriptions/*</span>

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

<h1 id="4-Standard-Methods"><code>标准方法</code></h1>

[<span style="color: rgb(0, 56, 132);">标准方法</span>](https://segmentfault.com/a/1190000008939076) 此章节定义标准方法 List、Get、Create、Update 和 Delete。标准方法存在的意义是广泛的 API 中许多 API 方法具有非常相似的语义，通过将这些类似的 API 融合到标准方法中，我们可以显著降低复杂性并提高一致性。以 Google APIs 为例，超过 70% 是标准方法，这让它们更加易于学习和使用。

下表描述了如何将它们映射为 REST 方法，也称为 CRUD 方法：

| <span style="color: rgb(0, 0, 0);">方法</span> | <span style="color: rgb(0, 0, 0);">HTTP 映射</span> | <span style="color: rgb(0, 0, 0);">HTTP 请求体</span> | <span style="color: rgb(0, 0, 0);">HTTP 响应体</span> |
| --- | --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">List</span> | <span style="color: rgb(0, 0, 0);">GET <集合 URL></span> | <span style="color: rgb(0, 0, 0);">空</span> | <span style="color: rgb(0, 0, 0);">资源[1]列表</span> |
| <span style="color: rgb(0, 0, 0);">Get</span> | <span style="color: rgb(0, 0, 0);">GET <资源 URL></span> | <span style="color: rgb(0, 0, 0);">空</span> | <span style="color: rgb(0, 0, 0);">资源[1]</span> |
| <span style="color: rgb(0, 0, 0);">Create</span> | <span style="color: rgb(0, 0, 0);">POST <集合 URL></span> | <span style="color: rgb(0, 0, 0);">资源</span> | <span style="color: rgb(0, 0, 0);">资源[1]</span> |
| <span style="color: rgb(0, 0, 0);">Update</span> | <span style="color: rgb(0, 0, 0);">PUT 或 PATCH <资源 URL></span> | <span style="color: rgb(0, 0, 0);">资源</span> | <span style="color: rgb(0, 0, 0);">资源[1]</span> |
| <span style="color: rgb(0, 0, 0);">Delete</span> | <span style="color: rgb(0, 0, 0);">DELETE <资源 URL></span> | <span style="color: rgb(0, 0, 0);">空</span> | <span style="color: rgb(0, 0, 0);">空[2]</span> |

[1] List、Get、Create 和 Update方法支持字段掩码时，返回的资源 可能（may） 只包含部分数据。在某些情况下，API 平台会对所有方法原生支持字段掩码。

[2] 不立即删除资源（比如通过更新标志位或会执行时间较长的删除操作）的 Delete 方法返回的响应 应该（should） 包含长时间运行的操作或被修改的资源。

标准方法也 可以（may） 为不能在一个 API 调用周期完成的请求返回长期运行的操作。

下面章节详细描述了每一个标准方法。这些例子展示了在 .proto 文件中定义的方法，其中包含用于 HTTP 映射的特殊注释。你可以在 Google APIs 项目中找到许多使用标准方法的例子。

<span style="font-size: 28px; color: rgb(51, 51, 51);">List</span>

* * *

<span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法接收资源名和零个或多个其它参数做为输入，返回符合输入的资源列表。它也通常被用来搜索资源。</span>

<span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 适合取得没有缓存且大小有限的来自单个集合的数据。对于更广泛的情况，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">自定义方法</span>](http://tailnode.tk/2017/03/google-api-design-guide/custom-methods/)<span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">应该使用自定义的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">BatchGet</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法来实现批量获取（例如接收多个资源 ID然后返回对应的资源），而不是使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。但如果已存在能够提供同样功能的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法，你 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 为此目的重用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法。如果你在使用自定义的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">BatchGet</span><span style="font-size: 15px; color: rgb(51, 51, 51);">方法，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 将它映射成 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">HTTP GET</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">适用的常见模式：</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">分页</span>](https://cloud.google.com/apis/design/design_patterns#list_pagination)<span style="font-size: 15px; color: rgb(51, 51, 51);">、</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">结果排序</span>](https://cloud.google.com/apis/design/design_patterns#sorting_order)

<span style="font-size: 15px; color: rgb(51, 51, 51);">适用的命名约定：</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">过滤字段</span>](http://tailnode.tk/2017/04/google-api-design-guide/naming-conventions/#list_filter_field)<span style="font-size: 15px; color: rgb(51, 51, 51);">、</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">结果字段</span>](http://tailnode.tk/2017/04/google-api-design-guide/naming-conventions/#list_response)

<span style="font-size: 15px; color: rgb(51, 51, 51);">HTTP 映射：</span>

*   List<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 HTTP </span>GET<span style="font-size: 15px; color: rgb(51, 51, 51);"> 动词。</span>
*   应该（should） 把要列出集合的资源名字放在 URL path 参数中。如果集合名映射到 URL path 中，URL 模版的最后一段（[<span style="color: rgb(0, 154, 97);">集合 ID</span>](http://tailnode.tk/2017/03/google-api-design-guide/resource-names/#collectionid)） 必须（must） 是常量。
*   所有其他的请求信息字段 必须（shall） 映射到 URL 的 query 参数中。
*   没有请求体，API 配置中一定不能（must not） 定义 <span style="font-size: 13px; color: rgb(199, 37, 78);">body</span>。
*   响应体 应该（should） 包含资源列表和可选的元数据。

<span style="font-size: 13px; color: rgb(153, 153, 136);">// 列出指定书架上的所有图书</span><span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">ListBooks(ListBooksRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(ListBooksResponse) { // List 方法映射为 HTTP GET option (google.api.http) = { // `parent` 获取父资源名，例如 "shelves/shelf1" get: "/v1/{parent=shelves/*}/books" };}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListBooksRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 父资源名称，例如 "shelves/shelf1".</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">parent =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 返回值的最大条数</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">int32</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">page_size =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 从上一个 List 请求返回的 next_page_token 值（如果存在）</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">page_token =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">3</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListBooksResponse</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 字段名应该匹配方法名中的名词 "books"，根据请求中的 page_size 字段，将会返回最大数量的条目</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(51, 51, 51);">repeated</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Book books =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 用于取得下一页结果的值，没有时为空</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">next_page_token =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">Get</span>

* * *

<span style="font-size: 13px; color: rgb(199, 37, 78);">GET</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法接收资源名，零个或多个参数，返回指定的资源。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">HTTP 映射：</span>

*   Get<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 HTTP </span>GET<span style="font-size: 15px; color: rgb(51, 51, 51);"> 动词。</span>
*   表示资源名的请求信息字段 应该（should） 映射到 URL path 参数中。
*   所有其他的请求信息字段 必须（shall） 映射到 URL 的 query 参数中。
*   没有请求体，API 配置中一定不能（must not） 定义 <span style="font-size: 13px; color: rgb(199, 37, 78);">body</span>。
*   返回的资源 必须（shall） 填充整个响应体。

<span style="font-size: 13px; color: rgb(153, 153, 136);">// 取得指定的 book</span><span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">GetBook(GetBookRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(Book) { // Get 映射为 HTTP GET。资源名映射到 URL 中。没有请求体 option (google.api.http) = { // 注意 URL 中用于获取资源名的模板变量，例如 "shelves/shelf1/books/book2" get: "/v1/{name=shelves/*/books/*}" };}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">GetBookRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 此字段包含被请求资源的名字，例如："shelves/shelf1/books/book2"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">name =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">Create</span>

* * *

<span style="font-size: 13px; color: rgb(199, 37, 78);">Create</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法接收一个集合名、零个或多个参数，在指定集合中创建一个新的资源并将其返回。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 API 支持创建资源，它 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在所有可被创建的资源上具有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Create</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">HTTP 映射：</span>

*   Create<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 HTTP </span>POST<span style="font-size: 15px; color: rgb(51, 51, 51);"> 动词。</span>
*   请求消息中 应该（should） 含有名为 <span style="font-size: 13px; color: rgb(199, 37, 78);">parent</span> 的字段来接收新建资源的父资源名。
*   所有其他的请求信息字段 必须（shall） 映射到 URL 的 query 参数中。
*   请求 可以（may） 包含名为 <span style="font-size: 13px; color: rgb(199, 37, 78);"><resource>_id</span> 的字段名来允许调用者选择一个客户端分配的 ID。这个字段 必须（must） 映射到 URL query 参数中。
*   包含资源的请求信息字段 应该（should） 映射到请求体中。如果 <span style="font-size: 13px; color: rgb(199, 37, 78);">Create</span> 方法使用了 HTTP 配置中的 <span style="font-size: 13px; color: rgb(199, 37, 78);">body</span> 字段，那么 必须（must） 使用 <span style="font-size: 13px; color: rgb(199, 37, 78);">body: "<resource_field>"</span> 这种格式。
*   返回的资源 必须（shall） 填充到整个响应体。

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Create</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法支持客户端指定资源名，当资源名已存在时请求 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 失败（</span><span style="font-size: 15px; color: rgb(51, 51, 51);">推荐（recommended）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.ALREADY_EXISTS</span><span style="font-size: 15px; color: rgb(51, 51, 51);">）或者服务端分配另外的名字，并且文档中应该明确指出创建的资源名可能会与传入的名字不同。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">CreateBook(CreateBookRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(Book) { // Create 映射为 HTTP POST，URL path 做为集合名称 // HTTP 请求体中包含资源 option (google.api.http) = { // 通过 `parent` 获取父资源名，例如 "shelves/1" post: "/v1/{parent=shelves/*}/books" body: "book" };}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">CreateBookRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 将被创建的 book 的父资源名</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">parent =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// book 使用的 ID</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">book_id =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">3</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 资源 book 将被创建，字段名应该与方法名中的名词相匹配</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Book book =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">CreateShelf(CreateShelfRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(Shelf) { option (google.api.http) = { post: "/v1/shelves" body: "shelf" };}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">CreateShelfRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{ Shelf shelf =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">Update</span>

* * *

<span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法接收包含资源和零个或多个参数的请求，更新指定的资源和属性并返回更新后的资源。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">可修改的资源属性 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法来更新，除非属性中包含</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">资源名或父资源</span>](http://tailnode.tk/2017/03/google-api-design-guide/resource-names/)<span style="font-size: 15px; color: rgb(51, 51, 51);">。任何</span><span style="font-size: 15px; color: rgb(51, 51, 51);">重命名</span><span style="font-size: 15px; color: rgb(51, 51, 51);">或</span><span style="font-size: 15px; color: rgb(51, 51, 51);">移动</span><span style="font-size: 15px; color: rgb(51, 51, 51);">资源的操作 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不要（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 通过 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 执行，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（shall）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 通过自定义方法处理。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">HTTP 映射：</span>

*   标准的 <span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span> 方法 应该（should） 支持资源的部分更新，使用带有名为 <span style="font-size: 13px; color: rgb(199, 37, 78);">update_mask</span> 的 <span style="font-size: 13px; color: rgb(199, 37, 78);">FieldMask</span> 字段的 HTTP 动词 <span style="font-size: 13px; color: rgb(199, 37, 78);">PATCH</span> 来执行操作。
*   应该（should） 使用[<span style="color: rgb(0, 154, 97);">自定义方法</span>](http://tailnode.tk/2017/03/google-api-design-guide/custom-methods/)来实现更高级的 <span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span> 方法，例如追加重复的字段。
*   如果 <span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span> 方法仅支持资源的完整更新，则 必须（must） 使用 HTTP 动词 <span style="font-size: 13px; color: rgb(199, 37, 78);">PUT</span>实现。然而并不鼓励这样做，因为当添加新的资源字段时会有后向兼容的问题。
*   被修改资源的名称字段 必须（must） 映射到 URL path 参数中。此字段也 可以（may） 加在资源信息中。
*   包含资源的请求信息 必须（must） 映射到请求体中。
*   所有其他请求信息 必须（must） 映射到 URL query 参数中。
*   返回响应中的资源信息 必须（must） 是被修改的资源。

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 API 接收客户端分配的资源名，那么服务端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 允许客户端指定一个不存在的资源名来创建新的资源。否则，</span><span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 因为不存在的资源名而失败。当资源名不存在是唯一错误时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用错误码 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">NOT_FOUND</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">即使 API 的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法能够新建资源，它也 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 提供 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Create</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法。这是因为只有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Update</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法能够新建资源会让人迷惑。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">rpc UpdateBook(UpdateBookRequest) returns (Book) {</span> <span style="font-size: 13px; color: rgb(0, 153, 38);">//</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Update 映射为 HTTP PATCH。资源名映射到 URL path 参数中</span> <span style="font-size: 13px; color: rgb(0, 153, 38);">//</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">资源包含在 HTTP 请求体中 option (google.api.http) = {</span> <span style="font-size: 13px; color: rgb(0, 153, 38);">//</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">注意用于获取待更新 book 的资源名的 URL 模版变量 patch:</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"/v1/{book.name=shelves/*/books/*}"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">body:</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"book"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">};}message UpdateBookRequest {</span> <span style="font-size: 13px; color: rgb(0, 153, 38);">//</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">替换服务端上的 book 资源 Book book =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 153, 38);">//</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">用于资源更新的掩码</span> <span style="font-size: 13px; color: rgb(0, 153, 38);">//</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">`FieldMask` 的定义请参考 https:</span><span style="font-size: 13px; color: rgb(0, 153, 38);">//</span><span style="font-size: 13px; color: rgb(51, 51, 51);">developers.google.com</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/protocol-buffers/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">docs</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/reference/g</span><span style="font-size: 13px; color: rgb(51, 51, 51);">oogle.protobuf</span><span style="font-size: 13px; color: rgb(153, 153, 136);">#fieldmask</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">FieldMask update_mask =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">Delete</span>

* * *

<span style="font-size: 13px; color: rgb(199, 37, 78);">Delete</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法接收资源名和零个或多个参数，然后删除或准备删除指定资源。</span><span style="font-size: 13px; color: rgb(199, 37, 78);">Delete</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.Empty</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意，API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 依赖 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Delete</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法返回的任何信息，因为它不能重复调用。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">HTTP 映射：</span>

*   Delete<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 HTTP </span>DELETE<span style="font-size: 15px; color: rgb(51, 51, 51);"> 动词</span>
*   表示资源名的请求信息字段 应该（should） 映射到 URL path 参数中
*   所有其他的请求信息字段 必须（shall） 映射到 URL 的 query 参数中
*   没有请求体，API 配置 一定不要（must not） 定义 <span style="font-size: 13px; color: rgb(199, 37, 78);">body</span>
*   Delete<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法直接删除资源时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回空的响应</span>
*   Delete<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法初始化一个长时间的操作时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回这个操作</span>
*   Delete<span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法将资源标记为被删除时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回更新后的资源</span>

<span style="font-size: 13px; color: rgb(199, 37, 78);">Delete</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法的调用在效果上应该是幂等的，但并不需要有相同的返回值。任意次 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Delete</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 请求的结果 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是资源被删除，但只有第一次请求应该返回正确，后续的请求应该返回 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.NOT_FOUND</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(153, 0, 0);">DeleteBook</span><span style="font-size: 13px; color: rgb(51, 51, 51);">(DeleteBookRequest)</span> <span style="font-size: 13px; color: rgb(153, 0, 0);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(google.protobuf.Empty) {</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// Delete 映射为 HTTP DELETE。资源名映射到 URL path 参数中</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 没有请求体</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">option (google.api.http) = {</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 注意 URL 模板变量获取待删除资源的名称，例如 "shelves/shelf1/books/book2"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(51, 51, 51);">delete</span><span style="font-size: 13px; color: rgb(51, 51, 51);">:</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"/v1/{name=shelves/*/books/*}"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">};}message DeleteBookRequest {</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 待删除的资源名称，例如 "shelves/shelf1/books/book2"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">name =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>


<h1 id="5-Custom-Methods"><code>自定义方法</code></h1>

[<span style="font-size: 15px; color: rgb(0, 56, 132);">自定义方法</span>](https://segmentfault.com/a/1190000008943186) <span style="font-size: 15px; color: rgb(51, 51, 51);">此篇文章讨论如何在 API 设计中使用自定义方法。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">自定义方法指五个标准方法之外的 API 方法。</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 只有当标准方法不能完成需要的功能时才使用自定义方法。一般情况下，API 设计者 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在可行的情况下选择标准方法。标准方法有更简洁和明确定义的语义并且被大多数开发者熟知，所以它们更易于使用并且不易出错。另一个优势是 API 平台对标准方法的支持更好，比如计费、错误处理、日志和监控。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">自定义方法可以被关联到资源、集合或服务。它 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 接收任意请求并返回任意响应，并且也可以支持流式请求与响应。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">HTTP 映射</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">自定义方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用如下的通用映射方法：</span>

[<span style="font-size: 13px; color: rgb(0, 56, 132);">https://service.name/v1/some/resource/name:customVerb</span>](https://service.name/v1/some/resource/name:customVerb)

<span style="font-size: 15px; color: rgb(51, 51, 51);">使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">:</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 代替 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">/</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 来分隔资源名与自定义动词是为了支持任意 path 参数，例如，取消删除（undelete）文件能映射为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">POST /files/a/long/file/name:undelete</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">选择 HTTP 映射时 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（shall）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 遵守如下指南：</span>

*   自定义方法 应该（should） 使用 HTTP  <span style="font-size: 13px; color: rgb(199, 37, 78);">POST</span>  动词，因为 POST 有更加灵活的语义
*   自定义方法 不应该（should not） 使用 HTTP <span style="font-size: 13px; color: rgb(199, 37, 78);">PATCH</span>，但 可以（may） 使用其它 HTTP 动词。这种情况下，方法必须（must） 遵循该动词的标准 [<span style="color: rgb(0, 154, 97);">HTTP 语义</span>](https://tools.ietf.org/html/rfc2616#section-9)
*   注意，使用 HTTP <span style="font-size: 13px; color: rgb(199, 37, 78);">GET</span> 的自定义方法 必须（must） 是幂等的并且不能有副作用。例如，在资源上实现特殊查询的自定义方法应该使用 HTTP <span style="font-size: 13px; color: rgb(199, 37, 78);">GET</span>。
*   自定义方法中待操作的资源或集合名 应该（should） 映射到 URL path 参数
*   URL path 必须（must） 以冒号加上 自定义动词 做为后缀
*   如果自定义方法使用的 HTTP 动词允许 HTTP 请求体（<span style="font-size: 13px; color: rgb(199, 37, 78);">POST</span>、<span style="font-size: 13px; color: rgb(199, 37, 78);">PUT</span>、<span style="font-size: 13px; color: rgb(199, 37, 78);">PATCH</span> 或自定义的 HTTP 动词），这些自定义方法的 HTTP 配置 必须（must） 使用 <span style="font-size: 13px; color: rgb(199, 37, 78);">body: "*"</span> 并且所有剩余的请求信息 必须（shall） 映射到 HTTP 请求体中
*   如果自定义方法使用的 HTTP 动词不允许 HTTP 请求体（<span style="font-size: 13px; color: rgb(199, 37, 78);">GET</span>、<span style="font-size: 13px; color: rgb(199, 37, 78);">DELETE</span>），这些方法的 HTTP 配置 一定不要（must not） 使用 <span style="font-size: 13px; color: rgb(199, 37, 78);">body</span>，所有剩余的请求信息 必须（shall） 映射到 HTTP query 参数中

<span style="font-size: 15px; color: rgb(51, 51, 51);">警告</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：如果服务实现多个 API，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 小心地创建服务配置以避免 API 间的自定义动词冲突。</span>

// 服务级别的自定义方法

rpc Watch(WatchRequest) returns (WatchResponse) {

// 自定义方法映射到 HTTP POST，所有参数放到请求体中

option (google.api.http) = {

post: "/v1:watch"

body: "*"

};

}

// 集合级别的自定义方法

rpc ClearEvents(ClearEventsRequest) returns (ClearEventsResponse) {

option (google.api.http) = {

post: "/v3/events:clear"

body: "*"

};

}

// 资源级别的自定义方法

rpc CancelEvent(CancelEventRequest) returns (CancelEventResponse) {

option (google.api.http) = {

post: "/v3/{name=events/*}:cancel"

body: "*"

};

}

// 用于批量 get 的自定义方法

rpc BatchGetEvents(BatchGetEventsRequest) returns (BatchGetEventsResponse) {

// 批量 get 方法映射到 HTTP GET

option (google.api.http) = {

get: "/v3/events:batchGet"

};

}

<span style="font-size: 28px; color: rgb(51, 51, 51);">用例</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">一些可选择自定义方法的其他情况：</span>

*   重启虚拟机 设计的备选方案可以是“在重启资源集合中创建一个重启资源”（过于复杂）或者“虚拟机具有可变的状态，客户端能够将其从运行中改变为重启中”（会引入更多问题，比如是否有其他状态间的转变）。此外，重启是一个被熟知的概念，能够很好地转换为满足开发者需求的自定义方法
*   发送邮件 创建邮件信息并不一定要将它发送出去（草稿）。相对于备选方案（将消息移动到 Outbox 集合），自定义方法的优点是可以被 API 用户更多地发现并且更直接地理解它的概念
*   员工晋级 如果使用标准的 <span style="font-size: 13px; color: rgb(199, 37, 78);">update</span> 来实现，客户端必须重复进行管理流程的策略来保证正确的晋级

<span style="font-size: 15px; color: rgb(51, 51, 51);">一些标准方法比自定义方法更好的例子：</span>

*   使用不同的参数查询资源（使用标准的 <span style="font-size: 13px; color: rgb(199, 37, 78);">list</span> 方法和过滤）
*   简单的资源修改（使用带有字段掩码的标准 <span style="font-size: 13px; color: rgb(199, 37, 78);">update</span> 方法）
*   撤消通知（使用标准的 <span style="font-size: 13px; color: rgb(199, 37, 78);">delete</span> 方法）

<span style="font-size: 28px; color: rgb(51, 51, 51);">通用自定义方法</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">常用的自定义方法名称列表如下。 API 设计者在引入自已的名称之前应该考虑这些名称，以便于跨 API 的一致性</span>

| <span style="color: rgb(0, 0, 0);">方法名</span> | <span style="color: rgb(0, 0, 0);">自定义动词</span> | <span style="color: rgb(0, 0, 0);">HTTP 动词</span> | <span style="color: rgb(0, 0, 0);">备注</span> |
| --- | --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">Canecl</span> | <span style="color: rgb(0, 0, 0);">:cancel</span> | <span style="color: rgb(0, 0, 0);">POST</span> | <span style="color: rgb(0, 0, 0);">取消未完成的操作（构建、计算等）</span> |
| <span style="color: rgb(0, 0, 0);">BatchGet<复数名词></span> | <span style="color: rgb(0, 0, 0);">:batchGet</span> | <span style="color: rgb(0, 0, 0);">POST</span> | <span style="color: rgb(0, 0, 0);">批量取得多个资源（详情请查看 List 的描述）</span> |
| <span style="color: rgb(0, 0, 0);">Move</span> | <span style="color: rgb(0, 0, 0);">:move</span> | <span style="color: rgb(0, 0, 0);">GET</span> | <span style="color: rgb(0, 0, 0);">将资源从一个父资源移动到另一个中</span> |
| <span style="color: rgb(0, 0, 0);">Search</span> | <span style="color: rgb(0, 0, 0);">:search</span> | <span style="color: rgb(0, 0, 0);">GET</span> | <span style="color: rgb(0, 0, 0);">用于获取不符合 List 语义的数据</span> |
| <span style="color: rgb(0, 0, 0);">Undelete</span> | <span style="color: rgb(0, 0, 0);">:undelete</span> | <span style="color: rgb(0, 0, 0);">POST</span> | <span style="color: rgb(0, 0, 0);">恢复以前删除的资源，推荐的保留期为30天</span> |

<h1 id="6-Errors"><code>错误处理</code></h1>

[<span style="font-size: 15px; color: rgb(0, 56, 132);">错误处理</span>](https://segmentfault.com/a/1190000008943276) <span style="font-size: 15px; color: rgb(51, 51, 51);">本章简单介绍 Google API 的错误模型以及开发人员如何正确生成和处理错误的一般指导。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">Google API 使用了简单的协议无关的错误模型，这允许我们在不同 API，不同协议（例如 gRPC 或 HTTP），不同的错误上下文（如同步、批量处理、工作流错误）中提供相同的使用体验。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">错误模型</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">错误模型由 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">google.rpc.Status</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 定义。如下所示：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">package google.rpc;message Status {</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 容易被客户端处理的简单错误码，实际的错误码定义在`google.rpc.Code`</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">int32 code =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 易于开发者阅读的错误信息，它应该解释错误并且提供可行的解决方法</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">message =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 附加的错误信息，客户端能够通过它处理错误，例如重试的等待时间或者帮助页面链接</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">repeated google.protobuf.Any details =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">3</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">因为大部分 Google API 使用面向资源的设计，所以错误处理通过在大量资源上使用一小组标准错误也遵循了相同的设计原则。例如，服务端使用一个标准的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.NOT_FOUND</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 错误码加上特定的资源名来表示“未找到”错误，而不是定义不同种类的“未找到”错误。更少的错误状态减少了文档的复杂性，在客户端的库中提供更好的习惯性映射，在不限制可包含信息的情况下减少了客户端逻辑的复杂性。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">错误码</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">Google API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span>[<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 中定义的标准错误码。单独的 API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 避免定义附加错误码，因为开发者非常不喜欢为大量错误码编写处理逻辑。作为参考，每个 API 处理平均 3 个错误码意味着大部分程序逻辑在进行错误处理，这并不是好的开发体验。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">错误消息</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">错误消息应该帮助用户轻松并快速地 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">理解并解决</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> API 错误。通常情况请参考如下规则：</span>

*   不要假设用户非常了解你的 API。用户可能是客户端开发者、运维人员、IT 人员或者 app 的普通用户。
*   不要假设用户了解服务实现的细节或熟悉错误上下文（例如日志分析）。
*   如果可能的话，应构建错误消息，以便技术用户（但不一定是 API 的开发人员）可以对错误进行响应并更正。
*   保持错误信息的简短。如果需要的话，提供链接以便迷惑的用户能够提出问题得到反馈或得到更多信息。否则，请使用 details 字段来扩展错误消息。

<span style="font-size: 28px; color: rgb(51, 51, 51);">错误详情</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">Google API 为错误详情定义了一组标准错误负载，可以去 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">google/rpc/error_details.proto</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 查看。这里包含了 API 中最常见的错误，例如达到资源限额和错误的输入参数。与错误码相同，错误详情也应该使用标准的负载。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">只有在能够帮助程序代码处理错误时才可以为错误详情引入新的类型。如果错误信息只能够由人（非代码）处理，应当让开发者依赖错误消息的内容来手动处理，而不是引入新的错误详情类型。如果新的类型被引入，一定要为它们进行显式的注册。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">这里有一些 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">error_details </span><span style="font-size: 15px; color: rgb(51, 51, 51);">负载的示例：</span>

*   RetryInfo<span style="font-size: 15px; color: rgb(51, 51, 51);"> 描述了当客户端能够重试请求时，可能返回 </span>Code.UNAVAILABLE<span style="font-size: 15px; color: rgb(51, 51, 51);"> 或 </span>Code.ABORTED
*   QuotaFailure<span style="font-size: 15px; color: rgb(51, 51, 51);"> 描述了配额检查失败的原因，可能返回 </span>Code.RESOURCE_EXHAUSTED
*   BadRequest<span style="font-size: 15px; color: rgb(51, 51, 51);"> 描述了客户端的非法请求，可能返回 </span>Code.INVALID_ARGUMENT

<span style="font-size: 28px; color: rgb(51, 51, 51);">HTTP 映射</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">虽然 proto3 有原生的 JSON 编码，但 Google 的 API 平台使用如下的 JSON 格式进行错误响应，以允许向后兼容：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"error"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">: {</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"code"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">:</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">401</span><span style="font-size: 13px; color: rgb(51, 51, 51);">,</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"message"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">:</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"Request had invalid credentials."</span><span style="font-size: 13px; color: rgb(51, 51, 51);">,</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"status"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">:</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"UNAUTHENTICATED"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">,</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"details"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">: [{</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"@type"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">:</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"type.googleapis.com/google.rpc.RetryInfo"</span><span style="font-size: 13px; color: rgb(51, 51, 51);">, ... }] }}</span>

| <span style="color: rgb(0, 0, 0);">字段</span> | <span style="color: rgb(0, 0, 0);">描述</span> |
| --- | --- |
| <span style="color: rgb(0, 0, 0);">error</span> | <span style="color: rgb(0, 0, 0);">为了向后兼容 Google API 客户端库添加的额外层</span> |
| <span style="color: rgb(0, 0, 0);">code</span> | <span style="color: rgb(0, 0, 0);">Status.code 映射为 HTTP 状态码</span> |
| <span style="color: rgb(0, 0, 0);">message</span> | <span style="color: rgb(0, 0, 0);">对应 Status.message</span> |
| <span style="color: rgb(0, 0, 0);">status</span> | <span style="color: rgb(0, 0, 0);">对应 Status.code</span> |
| <span style="color: rgb(0, 0, 0);">details</span> | <span style="color: rgb(0, 0, 0);">对应 Status.details</span> |

<span style="font-size: 28px; color: rgb(51, 51, 51);">RPC 映射</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">不同的 RPC 协议用不同的方法映射到错误模型（error model）。对于 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">gRPC</span>](http://grpc.io/)<span style="font-size: 15px; color: rgb(51, 51, 51);">，生成的代码和所有语言的运行库都原生支持错误模型。你可以去 gRPC 的 API 文档中查看详情。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">客户端库的映射</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">Google 客户端库可能会选择按照不同的惯例来对不同语言进行不同的错误处理。例如，库 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">google-cloud-go</span>](https://github.com/GoogleCloudPlatform/google-cloud-go)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 会返回 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Status</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的实例，而 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">google-cloud-java</span>](https://github.com/GoogleCloudPlatform/google-cloud-java)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 则会抛出异常。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">错误信息本地化</span>

* * *

[<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Status</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 中的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">message</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 字段是面向开发者的，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是英语。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果需要向用户提供错误信息，请使用 </span>[<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.LocalizedMessage</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);">作为详情字段。</span>[<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.LocalizedMessage</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 可以被本地化，但请保证 </span>[<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Status</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 中是英文。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 服务应该默认使用认证用户的 locale 或 HTTP </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Accept-Language</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 头来决定本地化语言。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">错误处理</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">下表包含了所有在 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">google.rpc.Code</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 中定义的 gRPC 错误代码和产生原因的简单描述。可以通过查看返回状态码的描述并修改对应的代码来处理错误。</span>

| <span style="color: rgb(0, 0, 0);">HTTP</span> | <span style="color: rgb(0, 0, 0);">RPC</span> | <span style="color: rgb(0, 0, 0);">Description</span> |
| --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">200</span> | <span style="color: rgb(0, 0, 0);">OK</span> | <span style="color: rgb(0, 0, 0);">没有错误</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">INVALID_ARGUMENT</span> | <span style="color: rgb(0, 0, 0);">客户端使用了错误的参数，通过 error message 和 error details 查看更多信息</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">FAILED_PRECONDITION</span> | <span style="color: rgb(0, 0, 0);">当前的系统状态不能执行请求，例如删除非空目录</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">OUT_OF_RANGE</span> | <span style="color: rgb(0, 0, 0);">客户端指定无效范围</span> |
| <span style="color: rgb(0, 0, 0);">401</span> | <span style="color: rgb(0, 0, 0);">UNAUTHENTICATED</span> | <span style="color: rgb(0, 0, 0);">由于缺少、无效或过期的 OAuth 令牌，请求未通过身份验证</span> |
| <span style="color: rgb(0, 0, 0);">403</span> | <span style="color: rgb(0, 0, 0);">PERMISSION_DENIED</span> | <span style="color: rgb(0, 0, 0);">客户端没有足够的权限，这可能是因为 OAuth 令牌没有正确的范围，客户端没有权限或者 API 还没有开放</span> |
| <span style="color: rgb(0, 0, 0);">404</span> | <span style="color: rgb(0, 0, 0);">NOT_FOUND</span> | <span style="color: rgb(0, 0, 0);">指定的资源不存在，或者由于未公开的原因（如白名单）请求被拒绝</span> |
| <span style="color: rgb(0, 0, 0);">409</span> | <span style="color: rgb(0, 0, 0);">ABORTED</span> | <span style="color: rgb(0, 0, 0);">并发冲突，例如读写冲突</span> |
| <span style="color: rgb(0, 0, 0);">409</span> | <span style="color: rgb(0, 0, 0);">ALREADY_EXISTS</span> | <span style="color: rgb(0, 0, 0);">客户端试图创建的资源已经存在</span> |
| <span style="color: rgb(0, 0, 0);">429</span> | <span style="color: rgb(0, 0, 0);">RESOURCE_EXHAUSTED</span> | <span style="color: rgb(0, 0, 0);">超过资源限额或频率限制,客户端应该通过 google.rpc.QuotaFailure 查看更多信息</span> |
| <span style="color: rgb(0, 0, 0);">499</span> | <span style="color: rgb(0, 0, 0);">CANCELLED</span> | <span style="color: rgb(0, 0, 0);">客户端取消请求</span> |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">DATA_LOSS</span> | <span style="color: rgb(0, 0, 0);">不可恢复的数据丢失或损坏，客户端应该将此错误报告给用户</span> |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">UNKNOWN</span> | <span style="color: rgb(0, 0, 0);">服务端未知错误，一般是 BUG</span> |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">INTERNAL</span> | <span style="color: rgb(0, 0, 0);">服务端内部错误，一般是 BUG</span> |
| <span style="color: rgb(0, 0, 0);">501</span> | <span style="color: rgb(0, 0, 0);">NOT_IMPLEMENTED</span> | <span style="color: rgb(0, 0, 0);">服务端未实现此 API</span> |
| <span style="color: rgb(0, 0, 0);">503</span> | <span style="color: rgb(0, 0, 0);">UNAVAILABLE</span> | <span style="color: rgb(0, 0, 0);">服务端不可用，一般是服务端挂了</span> |
| <span style="color: rgb(0, 0, 0);">504</span> | <span style="color: rgb(0, 0, 0);">DEADLINE_EXCEEDED</span> | <span style="color: rgb(0, 0, 0);">请求超过最后期限，如果重复发生，请考虑减少请求的复杂性</span> |

<span style="font-size: 24px; color: rgb(51, 51, 51);">错误重试</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">当发生 500，503 和 504 错误时客户端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以指数级增长的间隔来重试请求。除非文档中进行了说明，最小的重试间隔应该是 1 秒。对于 429 错误，客户端应该以最小 30 秒的间隔重试。对于其他错误，重试操作可能并不可行，请先确保请求是幂等的并查看错误消息以获得指引。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">错误传播</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 API 服务依赖于其他服务，则不应盲目地将这些服务中的错误传播给客户端。翻译错误时，有如下建议：</span>

*   隐藏实现细节和机密信息
*   调整负责该错误的一方。例如，应把从其他服务接收到 INVALID ARGUMENT 错误转换为 INTERNAL 返回给调用者。

<span style="font-size: 28px; color: rgb(51, 51, 51);">生成错误</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">服务端产生的错误应该包含足够多的信息来帮助客户端开发者理解和解决问题。同时也要小心用户数据的安全和隐私，因为错误经常会被作为日志记录下来并被其他人查看，所以应避免在错误信息和错误详情中暴露敏感信息。例如，错误信息“Client IP address is not on whitelist 128.0.0.0/8”将用户不可访问的服务端策略暴露出去了。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了生成合适的错误，你首先应该熟悉 </span>[<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code</span>](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 来为每种错误条件选择最合适的错误。服务端程序可以并行检查多个错误条件，然后返回第一个。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">下表列出了每一种错误码和对应的错误信息示例。</span>

| <span style="color: rgb(0, 0, 0);">HTTP</span> | <span style="color: rgb(0, 0, 0);">RPC</span> | <span style="color: rgb(0, 0, 0);">错误信息示例</span> |
| --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">INVALID_ARGUMENT</span> | <span style="color: rgb(0, 0, 0);">Request field x.y.z is xxx, expected one of [yyy, zzz].</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">FAILED_PRECONDITION</span> | <span style="color: rgb(0, 0, 0);">Resource xxx is a non-empty directory, so it cannot be deleted.</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">OUT_OF_RANGE</span> | <span style="color: rgb(0, 0, 0);">Parameter 'age' is out of range [0, 125].</span> |
| <span style="color: rgb(0, 0, 0);">401</span> | <span style="color: rgb(0, 0, 0);">UNAUTHENTICATED</span> | <span style="color: rgb(0, 0, 0);">Invalid authentication credentials.</span> |
| <span style="color: rgb(0, 0, 0);">403</span> | <span style="color: rgb(0, 0, 0);">PERMISSION_DENIED</span> | <span style="color: rgb(0, 0, 0);">Permission 'xxx' denied on file 'yyy'.</span> |
| <span style="color: rgb(0, 0, 0);">404</span> | <span style="color: rgb(0, 0, 0);">NOT_FOUND</span> | <span style="color: rgb(0, 0, 0);">Resource 'xxx' not found.</span> |
| <span style="color: rgb(0, 0, 0);">409</span> | <span style="color: rgb(0, 0, 0);">ABORTED</span> | <span style="color: rgb(0, 0, 0);">Couldn’t acquire lock on resource ‘xxx’.</span> |
| <span style="color: rgb(0, 0, 0);">409</span> | <span style="color: rgb(0, 0, 0);">ALREADY_EXISTS</span> | <span style="color: rgb(0, 0, 0);">Resource 'xxx' already exists.</span> |
| <span style="color: rgb(0, 0, 0);">429</span> | <span style="color: rgb(0, 0, 0);">RESOURCE_EXHAUSTED</span> | <span style="color: rgb(0, 0, 0);">Quota limit 'xxx' exceeded.</span> |
| <span style="color: rgb(0, 0, 0);">499</span> | <span style="color: rgb(0, 0, 0);">CANCELLED</span> | <span style="color: rgb(0, 0, 0);">Request cancelled by the client.</span> |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">DATA_LOSS</span> | <span style="color: rgb(0, 0, 0);">请看提示</span> |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">UNKNOWN</span> | <span style="color: rgb(0, 0, 0);">请看提示</span> |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">INTERNAL</span> | <span style="color: rgb(0, 0, 0);">请看提示</span> |
| <span style="color: rgb(0, 0, 0);">501</span> | <span style="color: rgb(0, 0, 0);">NOT_IMPLEMENTED</span> | <span style="color: rgb(0, 0, 0);">Method 'xxx' not implemented.</span> |
| <span style="color: rgb(0, 0, 0);">503</span> | <span style="color: rgb(0, 0, 0);">UNAVAILABLE</span> | <span style="color: rgb(0, 0, 0);">请看提示</span> |
| <span style="color: rgb(0, 0, 0);">504</span> | <span style="color: rgb(0, 0, 0);">DEADLINE_EXCEEDED</span> | <span style="color: rgb(0, 0, 0);">请看提示</span> |

<span style="font-size: 15px; color: rgb(51, 51, 51);">提示</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：因为客户端不能修复服务端的错误，生成额外的错误详情并没有用处。为了避免通过 error condition 泄露敏感信息，推荐不要生成任何 error message 并且只生成 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.DebugInfo</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 错误详情。</span><span style="font-size: 13px; color: rgb(199, 37, 78);">DebugInfo</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 只能用于服务端日志，不要发送给客户端。</span>

<span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 定义了一组标准错误负载，它们优先于自定义的错误负载。下表列出了每个错误代码及其匹配的标准错误负载。</span>

| <span style="color: rgb(0, 0, 0);">HTTP</span> | <span style="color: rgb(0, 0, 0);">RPC</span> | <span style="color: rgb(0, 0, 0);">推荐的错误详情</span> |
| --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">INVALID_ARGUMENT</span> | <span style="color: rgb(0, 0, 0);">google.rpc.BadRequest</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">FAILED_PRECONDITION</span> | <span style="color: rgb(0, 0, 0);">google.rpc.PreconditionFailure</span> |
| <span style="color: rgb(0, 0, 0);">400</span> | <span style="color: rgb(0, 0, 0);">OUT_OF_RANGE</span> | <span style="color: rgb(0, 0, 0);">google.rpc.BadRequest</span> |
| <span style="color: rgb(0, 0, 0);">401</span> | <span style="color: rgb(0, 0, 0);">UNAUTHENTICATED</span> |  |
| <span style="color: rgb(0, 0, 0);">403</span> | <span style="color: rgb(0, 0, 0);">PERMISSION_DENIED</span> |  |
| <span style="color: rgb(0, 0, 0);">404</span> | <span style="color: rgb(0, 0, 0);">NOT_FOUND</span> |  |
| <span style="color: rgb(0, 0, 0);">409</span> | <span style="color: rgb(0, 0, 0);">ABORTED</span> |  |
| <span style="color: rgb(0, 0, 0);">409</span> | <span style="color: rgb(0, 0, 0);">ALREADY_EXISTS</span> |  |
| <span style="color: rgb(0, 0, 0);">429</span> | <span style="color: rgb(0, 0, 0);">RESOURCE_EXHAUSTED</span> | <span style="color: rgb(0, 0, 0);">google.rpc.QuotaFailure</span> |
| <span style="color: rgb(0, 0, 0);">499</span> | <span style="color: rgb(0, 0, 0);">CANCELLED</span> |  |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">DATA_LOSS</span> |  |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">UNKNOWN</span> |  |
| <span style="color: rgb(0, 0, 0);">500</span> | <span style="color: rgb(0, 0, 0);">INTERNAL</span> |  |
| <span style="color: rgb(0, 0, 0);">501</span> | <span style="color: rgb(0, 0, 0);">NOT_IMPLEMENTED</span> |  |
| <span style="color: rgb(0, 0, 0);">503</span> | <span style="color: rgb(0, 0, 0);">UNAVAILABLE</span> |  |
| <span style="color: rgb(0, 0, 0);">504</span> | <span style="color: rgb(0, 0, 0);">DEADLINE_EXCEEDED</span> |  |

<h1 id="7-Naming-Conventions"><code>命名规范</code></h1>

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了在长期和大量使用的 API 中提供统一的开发体验，API 中的所有名字 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> :</span>

*   简单
*   直观
*   一致

<span style="font-size: 15px; color: rgb(51, 51, 51);">此文章讨论了接口、资源、集合、方法和消息的名字。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">因为很多开发者的母语并不是英语，这些命名约定通过鼓励使用简单、统一的短词汇来命名方法和资源来保证大部分开发者能够容易理解 API。</span>

*   API 中的名字 应该（should） 使用美式英语，例如：license（不是 licence），color（不是 colour）。
*   使用常用的简短形式或缩写，例如： API 比 Application Programming Interface 更好。
*   尽量使用直接、熟悉的术语，例如：描述对资源的删除（销毁）时，delete 比 erase 更好。
*   对相同的概念使用相同的名字或术语，包括在 API 中共享的概念。
*   避免名字重用，对不同的概念要使用不同名字。
*   避免使用在 API 上下文中会造成混乱的过于通用的名字，它们会导致对 API 概念的误解。应该选择能够精确描述概念的名字。这对于定义一阶 API 元素的名称尤其重要。因为名字与上下文相关，所以并没有明确的名字黑名单。Instance, info 和 service 是会产生问题的名字。应该选择能够明确表达出 API 概念（例：instance 表示什么的实例？）并且容易与其他相关概念有区分（例：alert 的意思是规则，信号还是通知？）的名字。
*   谨慎使用会与常用编程语言中的关键字有冲突的名字。

<span style="font-size: 28px; color: rgb(51, 51, 51);">产品名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">产品名是指 API 的产品营销名称，例如 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">Google Calendar API</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。在 API、UI、文档、服务条款、结账单和商业合同中使用的产品名称 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 一致。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">Google API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">Google</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 作为产品名的前缀，除非它们像 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">Gmail</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 15px; color: rgb(51, 51, 51);">Nest</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 15px; color: rgb(51, 51, 51);">Youtube</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 这种有不同的品牌。一般来说产品名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 由产品和市场部门决定。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">下表列出了所有相关 API 名称及其一致性的示例，有关各自名称及其约定的更多详细信息，请继续往下看。</span>

| <span style="color: rgb(0, 0, 0);">API 名</span> | <span style="color: rgb(0, 0, 0);">示例</span> |
| --- | --- |
| <span style="color: rgb(0, 0, 0);">产品名</span> | <span style="color: rgb(0, 0, 0);">Google Calendar API</span> |
| <span style="color: rgb(0, 0, 0);">服务名</span> | <span style="color: rgb(0, 0, 0);">calendar.googleapis.com</span> |
| <span style="color: rgb(0, 0, 0);">包名</span> | <span style="color: rgb(0, 0, 0);">google.calendar.v3</span> |
| <span style="color: rgb(0, 0, 0);">接口名</span> | <span style="color: rgb(0, 0, 0);">google.calendar.v3.CalendarService</span> |
| <span style="color: rgb(0, 0, 0);">资源目录</span> | <span style="color: rgb(0, 0, 0);">//google/calendar/v3</span> |
| <span style="color: rgb(0, 0, 0);">API 名</span> | <span style="color: rgb(0, 0, 0);">calendar</span> |

<span style="font-size: 28px; color: rgb(51, 51, 51);">服务名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">服务名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是一个能够被解析为一个或多个网络地址的合法 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">DNS 名字</span>](http://www.ietf.org/rfc/rfc1035.txt)<span style="font-size: 15px; color: rgb(51, 51, 51);">。公有 Google API 的服务名遵循如下模式：</span><span style="font-size: 13px; color: rgb(199, 37, 78);">xxx.googleapis.com</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。例如：谷歌日历的服务名是 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">calendar.googleapis.com</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果一个 API 由多个服务组成，它们 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以能够提高可发现性的方法命名。一种方法是为这些服务名使用相同的前缀。例如服务 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">build.googleapis.com</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">buildresults.googleapis.com</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 都是 Google Build API 的一部分。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">包名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">在 .proto 文件中定义的包名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 与产品名和服务名相同。有版本号的包名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以版本号结尾。例如：</span>

<span style="font-size: 13px; color: rgb(153, 153, 136);">// Google Calendar API</span><span style="font-size: 13px; color: rgb(51, 51, 51);">package</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">google</span><span style="font-size: 13px; color: rgb(51, 51, 51);">.</span><span style="font-size: 13px; color: rgb(68, 85, 136);">calendar</span><span style="font-size: 13px; color: rgb(51, 51, 51);">.</span><span style="font-size: 13px; color: rgb(68, 85, 136);">v3</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">不与服务直接关联的抽象 API，例如 Google Watcher API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用与产品名相同的 proto 包名：</span>

<span style="font-size: 13px; color: rgb(153, 153, 136);">// Google Watcher API</span><span style="font-size: 13px; color: rgb(51, 51, 51);">package</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">google</span><span style="font-size: 13px; color: rgb(51, 51, 51);">.</span><span style="font-size: 13px; color: rgb(68, 85, 136);">watcher</span><span style="font-size: 13px; color: rgb(51, 51, 51);">.</span><span style="font-size: 13px; color: rgb(68, 85, 136);">v1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在 .proto 文件中指定的 Java 包名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 符合标准 Java 包名的前缀（</span><span style="font-size: 13px; color: rgb(199, 37, 78);">com.</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 13px; color: rgb(199, 37, 78);">edu.</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 13px; color: rgb(199, 37, 78);">net.</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 等）。例如：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">package google.calendar.v3</span><span style="font-size: 13px; color: rgb(153, 153, 136);">;</span><span style="font-size: 13px; color: rgb(51, 51, 51);">// 指定 Java 包的名字，使用标准前缀</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"com."</span><span style="font-size: 13px; color: rgb(51, 51, 51);">option java_package =</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"com.google.calendar.v3"</span><span style="font-size: 13px; color: rgb(153, 153, 136);">;</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">集合 ID</span>

* * *

[<span style="font-size: 15px; color: rgb(0, 154, 97);">集合 ID</span>](http://tailnode.tk/2017/03/google-api-design-guide/resource-names/#collectionid)<span style="font-size: 15px; color: rgb(51, 51, 51);"> </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用美式英语的、复数形式的、首字母小写的驼峰命名法，例如：</span><span style="font-size: 13px; color: rgb(199, 37, 78);">events</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 13px; color: rgb(199, 37, 78);">children</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 或 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">deletedEvents</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">接口名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了避免和形如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">pubsub.googleapis.com</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">资源名</span>](http://tailnode.tk/2017/04/google-api-design-guide/naming-conventions/#service_name)<span style="font-size: 15px; color: rgb(51, 51, 51);">混淆，术语 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">接口名</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 指的是在 .proto 文件中定义 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">service</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 时使用的名字：</span>

<span style="font-size: 13px; color: rgb(153, 153, 136);">// Library is the interface name.</span><span style="font-size: 13px; color: rgb(51, 51, 51);">service</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">Library</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">ListBooks(...)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(...);</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">...}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">你可以认为 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">服务名</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是指一组 API 的具体实现，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">接口名</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是指一个 API 的抽象定义。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">接口名称 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用直观的名词，如 Calendar 或 Blob。</span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 与编程语言中已有的任何概念或运行时库冲突（如：File）。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">当 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">接口名</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 与 API 中其他名字冲突时，应该为其加上前缀（如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Api</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 或 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Service</span><span style="font-size: 15px; color: rgb(51, 51, 51);">）来进行区分。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">方法名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">在其 IDL 规范中，服务 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 定义与集合和资源上的方法相对应的一个或多个RPC方法。方法名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 遵循像 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">VerbNoun</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 这样首字母大写的驼峰命名法的命名约定，其中名词（Noun）通常是资源类型。</span>

| <span style="color: rgb(0, 0, 0);">动词（Verb）</span> | <span style="color: rgb(0, 0, 0);">名词（Noun）</span> | <span style="color: rgb(0, 0, 0);">方法名</span> | <span style="color: rgb(0, 0, 0);">请求信息</span> | <span style="color: rgb(0, 0, 0);">响应信息</span> |
| --- | --- | --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">List</span> | <span style="color: rgb(0, 0, 0);">Book</span> | <span style="color: rgb(0, 0, 0);">ListBooks</span> | <span style="color: rgb(0, 0, 0);">ListBooksRequest</span> | <span style="color: rgb(0, 0, 0);">ListBooksResponse</span> |
| <span style="color: rgb(0, 0, 0);">Get</span> | <span style="color: rgb(0, 0, 0);">Book</span> | <span style="color: rgb(0, 0, 0);">GetBook</span> | <span style="color: rgb(0, 0, 0);">GetBookRequest</span> | <span style="color: rgb(0, 0, 0);">Book</span> |
| <span style="color: rgb(0, 0, 0);">Create</span> | <span style="color: rgb(0, 0, 0);">Book</span> | <span style="color: rgb(0, 0, 0);">CreateBook</span> | <span style="color: rgb(0, 0, 0);">CreateBookRequest</span> | <span style="color: rgb(0, 0, 0);">Book</span> |
| <span style="color: rgb(0, 0, 0);">Update</span> | <span style="color: rgb(0, 0, 0);">Book</span> | <span style="color: rgb(0, 0, 0);">UpdateBook</span> | <span style="color: rgb(0, 0, 0);">UpdateBookRequest</span> | <span style="color: rgb(0, 0, 0);">Book</span> |
| <span style="color: rgb(0, 0, 0);">Rename</span> | <span style="color: rgb(0, 0, 0);">Book</span> | <span style="color: rgb(0, 0, 0);">RenameBook</span> | <span style="color: rgb(0, 0, 0);">RenameBookRequest</span> | <span style="color: rgb(0, 0, 0);">RenameBookResponse</span> |
| <span style="color: rgb(0, 0, 0);">Delete</span> | <span style="color: rgb(0, 0, 0);">Book</span> | <span style="color: rgb(0, 0, 0);">DeleteBook</span> | <span style="color: rgb(0, 0, 0);">DeleteBookRequest</span> | <span style="color: rgb(0, 0, 0);">google.protobuf.Empty</span> |

<span style="font-size: 28px; color: rgb(51, 51, 51);">消息名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">RPC 方法的请求与响应消息 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以方法名分别加上 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Request</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Response</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的方式命名。除非方法的请求或响应类型如下：</span>

*   空消息（使用 <span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.Empty</span>）
*   资源类型
*   代表一种操作的资源

<span style="font-size: 28px; color: rgb(51, 51, 51);">枚举名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">枚举类型的名字 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用首字母大写的驼峰命名法。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">枚举值 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以下划线分隔且字母全部大写的方式来命名（例如：CAPITALIZED_NAMES_WITH_UNDERSCORES）。每个枚举值 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以分号而不是逗号结尾。第一个值 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 命名为 ENUM_TYPE_UNSPECIFIED，用于当没有明确指定枚举值时返回。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">enum</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">FooBar</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{ /</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/ 第一个表示默认值，并且一定等于 0 FOO_BAR_UNSPECIFIED = 0; FIRST_VALUE = 1; SECOND_VALUE = 2;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">字段名</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">在 .proto 文件中定义的字段名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以下划线分隔且字母全部小写的方式来命名（例如：lower_case_underscore_separated_names）。这些名字会遵守各编程语言的命名约定来映射到生成的代码中。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">重复字段名（Repeated field）</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 中的重复字段 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用合适的复数形式。这符合现有 Google API 的惯例以及外部开发人员的通常期望。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">时间和间隔</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">使用</span> <span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.Timestamp</span> <span style="font-size: 15px; color: rgb(51, 51, 51);">并且字段名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">time</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 结尾来表示独立于任一时区的时间点。例如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">start_time</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 13px; color: rgb(199, 37, 78);">end_time</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果表示一个活动的时间，字段名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">使用</span> <span style="font-size: 13px; color: rgb(199, 37, 78);">verb_time</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  格式，如  </span><span style="font-size: 13px; color: rgb(199, 37, 78);">create_time</span><span style="font-size: 15px; color: rgb(51, 51, 51);">, </span><span style="font-size: 13px; color: rgb(199, 37, 78);">update_time</span> <span style="font-size: 15px; color: rgb(51, 51, 51);">。避免使用动词的过去时形式，如</span> <span style="font-size: 13px; color: rgb(199, 37, 78);">created_time</span> <span style="font-size: 15px; color: rgb(51, 51, 51);">或</span> <span style="font-size: 13px; color: rgb(199, 37, 78);">last_updated_time</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">使用</span> <span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.Duration</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  来表示一个时间段。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">FlightRecord</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{ google.protobuf.Timestamp takeoff_time =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">; google.protobuf.Duration flight_duration =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果因为遗留系统或兼容原因要使用整形来表示时间相关的字段（包含墙上时间、时间段、延迟），字段名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">有如下形式：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">xxx_{</span><span style="font-size: 13px; color: rgb(0, 134, 179);">time</span><span style="font-size: 13px; color: rgb(51, 51, 51);">|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">duration</span><span style="font-size: 13px; color: rgb(51, 51, 51);">|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">delay</span><span style="font-size: 13px; color: rgb(51, 51, 51);">|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">latency</span><span style="font-size: 13px; color: rgb(51, 51, 51);">}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">_</span><span style="font-size: 13px; color: rgb(51, 51, 51);">{seconds|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">millis</span><span style="font-size: 13px; color: rgb(51, 51, 51);">|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">micros</span><span style="font-size: 13px; color: rgb(51, 51, 51);">|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">nanos</span><span style="font-size: 13px; color: rgb(51, 51, 51);">}</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">Email</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">int64</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">send_time_millis =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">int64</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">receive_time_millis =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果因为遗留系统或兼容原因要使用字符串来表示时间戳，字段名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 包含任何单位后缀。</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用形如  </span><span style="font-size: 13px; color: rgb(199, 37, 78);">2014-07-30T10:43:17Z</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  的 RFC 3339 格式。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">日期与时间</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用  </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.type.Date</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  并且字段名以  </span><span style="font-size: 13px; color: rgb(199, 37, 78);">_date</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  结尾来表示独立于时区与时间的日期。如果必须以字符串表示，应该使用形如 YYYY-MM-DD 的 ISO 8601 日期格式（例如 2014-07-30）。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.type.TimeOfDay</span><span style="font-size: 15px; color: rgb(51, 51, 51);">  并且字段名以 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">_time</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 结尾来表示独立于时区与日期的时间。如果必须以字符串表示，应该使用形如 HH:MM:SS[.FFF] 的 ISO 8601 的 24 小时时间格式（例如 14:55:01.672）。</span>

<span style="font-size: 13px; color: rgb(153, 0, 0);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">StoreOpening</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{ google.</span><span style="font-size: 13px; color: rgb(51, 51, 51);">type</span><span style="font-size: 13px; color: rgb(51, 51, 51);">.</span><span style="font-size: 13px; color: rgb(68, 85, 136);">Date</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">opening_date = 1; google.</span><span style="font-size: 13px; color: rgb(51, 51, 51);">type</span><span style="font-size: 13px; color: rgb(51, 51, 51);">.</span><span style="font-size: 13px; color: rgb(68, 85, 136);">TimeOfDay</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">opening_time = 2;}</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">数量</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">以整形表示的数量 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 包含单位。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">xxx_{bytes|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">width_pixels</span><span style="font-size: 13px; color: rgb(51, 51, 51);">|</span><span style="font-size: 13px; color: rgb(68, 85, 136);">meters</span><span style="font-size: 13px; color: rgb(51, 51, 51);">}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果表示物品的数量，字段名 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">_count</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 做为后缀，如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">node_count</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">List 过滤字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法支持过滤资源，包含过滤表达式的字段 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 命名为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">filter</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。例：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListBooksRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 父资源名</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">parent =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 过滤表达式</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">filter =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">List 响应</span>

* * *

<span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法的响应消息中包含资源列表的字段名字 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是资源名的复数形式。例如，方法 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">CalendarApi.ListEvents()</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 定义响应消息 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">ListEventsResponse</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，此消息中包含名为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">events</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的重复字段来用于返回资源列表。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">service</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">CalendarApi</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">ListEvents(ListEventsRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(ListEventsResponse) { option (google.api.http) = { get: "/v3/{parent=calendars/*}/events"; }; }}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListEventsRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">parent =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">int32</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">page_size =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">page_token =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">3</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListEventsResponse</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">repeated</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Event events =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">next_page_token =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">驼峰命名法</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">除了字段名和枚举值，在 .proto 文件中的所有定义 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用首字母大写的</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">驼峰命名法</span>](https://google.github.io/styleguide/javaguide.html#s5.3-camel-case)<span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">名字缩写</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">对于像 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">config</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">spec</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 这种被软件工程师熟知的缩写，在 API 定义中 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用缩写而不是其完整形式，这会使代码便于读写。在正式的文档中 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用其完整形式。例如：</span>

*   config (configuration)
*   id (identifier)
*   spec (specification)
*   stats (statistics)

<h1 id="8-Common-Design-Patterns"><code>设计模式</code></h1>

<span style="font-size: 28px; color: rgb(51, 51, 51);">空响应体</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">标准的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Delete</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.Empty</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 来实现全局一致性。它还可以防止客户端依赖于在重试期间不可用的附加元数据。因为随着时间推移对于自定义方法， 对于自定义方法，它们 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 具有自己的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">XxxResponse</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 消息，即使它们是空的，因为功能很可能随着时间的推移而增加，并且需要返回附加数据。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">范围字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">表示范围的字段 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用符合命名约定的半开半闭区间 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">[start_xxx, end_xxx)</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，例如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">[start_key, end_key)</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 或 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">[start_time, end_time)</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。C++ STL 和 Java 标准库经常使用半开半闭的语义。API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 避免使用表示区间的其他方法，例如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">(index, count)</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 或 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">[first, last]</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">资源标签</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">在面向资源的 API 中，资源结构由 API 定义。为了允许客户端向资源附加少量且简单的元数据（例如将一台虚拟机资源标记为数据库服务器），API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用在 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.api.LabelDescriptor</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 中描述的资源标签设计模式。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在资源定义中添加字段 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">map<string, string> labels</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">Book</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">name =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">; map<</span><span style="font-size: 13px; color: rgb(0, 134, 179);">string</span><span style="font-size: 13px; color: rgb(51, 51, 51);">,</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span><span style="font-size: 13px; color: rgb(51, 51, 51);">> labels =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">耗时操作</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果一个 API 方法需要花费较长时间运行，可以将其设计成向客户端返回一个表示长时间运行的资源，客户端可通过这个资源来获取操作的执行进度并取得执行结果。</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Operation</span>](https://github.com/googleapis/googleapis/blob/master/google/longrunning/operations.proto)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 定义了耗时操作的标准接口。</span><span style="font-size: 15px; color: rgb(51, 51, 51);">不能（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 为 API 使用自定义的耗时操作接口以免打破一致性。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">资源 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 作为响应消息直接返回，并且对资源操作的结果 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 反应在 API 中。例如：当创建资源时这个资源 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 显示在 LIST 和 GET 方法中，并且 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 指示出这个资源还没有准备好。如果方法不需要长期执行，当操作完成时 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Operation.response</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 字段应该包含直接返回的消息。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">列表分页</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">即使结果集很小，可 LIST 的集合也 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 支持分页。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">理由：尽管向已有的 API 添加分页功能从 API 的视角来看是纯粹的增加功能，但它实际会改变行为。不知道有分页功能的已有客户端会错误地将取到的第一页数据当成全部数据。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了在 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法中支持分页， API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（shall）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：</span>

*   在 <span style="font-size: 13px; color: rgb(199, 37, 78);">List</span> 方法的请求信息中定义一个 <span style="font-size: 13px; color: rgb(199, 37, 78);">string</span> 字段 <span style="font-size: 13px; color: rgb(199, 37, 78);">page_token</span>。客户端通过这个字段来请求指定的某一页。
*   在 <span style="font-size: 13px; color: rgb(199, 37, 78);">List</span> 方法的请求信息中定义一个 <span style="font-size: 13px; color: rgb(199, 37, 78);">int32</span> 字段 <span style="font-size: 13px; color: rgb(199, 37, 78);">page_size</span>。客户端通过这个字段来指定返回结果的最大数量。服务端可以进一步限制在单个页面中返回的最大结果数量。<span style="font-size: 13px; color: rgb(199, 37, 78);">page_size</span> 是 0 时，将由服务端决定返回结果的数量。
*   在 <span style="font-size: 13px; color: rgb(199, 37, 78);">List</span> 方法的响应信息中定义一个 <span style="font-size: 13px; color: rgb(199, 37, 78);">string</span> 字段 <span style="font-size: 13px; color: rgb(199, 37, 78);">next_page_token</span>。这个字段表示取得下一页的页码。空字符串表示没有更多数据了。

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了取得下一页的结果，客户端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（shall）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 将响应中的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">next_page_token</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 传入下次的请求：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">ListBooks(ListBooksRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(ListBooksResponse);</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListBooksRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">name =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">int32</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">page_size =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">page_token =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">3</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListBooksResponse</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">repeated</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Book books =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">next_page_token =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">当客户端在 query 参数传入除 page token 之外的参数时，如果 query 参数与 page token 不一致，服务 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 拒绝此请求。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">page token 的内容 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是对 web 安全的 BASE64 编码后的 protocol buffer，这样就不会有兼容性问题。page token 中存在敏感信息时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 将其加密。服务端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 通过以下方法来防止通过篡改 page token 来获取敏感信息的问题：</span>

*   根据后续请求指定 query 参数
*   在 page token 中仅引用服务端的状态
*   在 page token 中加密并签名 query 参数，并且在每次调用中对这些参数进行验证和鉴权

<span style="font-size: 15px; color: rgb(51, 51, 51);">分页功能也 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在响应中通过名为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">total_size</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 类型为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">int32</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的字段来提供查询资源的总数量。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">列出子集合</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 有时需要客户端对子集合进行 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List/Search</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 操作。例如一个 API 有</span><span style="font-size: 13px; color: rgb(199, 37, 78);">书架</span><span style="font-size: 15px; color: rgb(51, 51, 51);">集合，每个书架有</span><span style="font-size: 13px; color: rgb(199, 37, 78);">书</span><span style="font-size: 15px; color: rgb(51, 51, 51);">的集合，客户端想要在所有书架中搜索一本书。这种情况下推荐在子集合上使用标准的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">List</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，并且为父集合指定通配符 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">"-"</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。例如：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">GET https:</span><span style="font-size: 13px; color: rgb(0, 153, 38);">//</span><span style="font-size: 13px; color: rgb(51, 51, 51);">library.googleapis.com</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/v1/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">shelves</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/-/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">books?filter=xxx</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意：使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">"-"</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 而非 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">"*"</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是为了避免 URL 转义。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">从子集合中取得唯一资源</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">有时子集合中的资源具有在其父集合内唯一的标识符，在这种情况下通过 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Get</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 来取得某资源而不需要知道它的父集合可能是有用的。在这种情况下，建议使用标准 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Get</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，并为资源唯一的所有父集合指定通配符 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">"-"</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。例如：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">GET https:</span><span style="font-size: 13px; color: rgb(0, 153, 38);">//</span><span style="font-size: 13px; color: rgb(51, 51, 51);">library.googleapis.com</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/v1/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">shelves</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/-/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">books</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/{id}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">响应 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用资源的带有父集合标识符的规范名称。例如上面的请求应该返回名称类似 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">shelves/shelf713/books/book8141</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的资源，而不是 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">shelves/-/books/book8141</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">排序</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 API 方法允许客户端指定列表结果的排序顺序，请求消息中 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 包含如下字段：</span>

<span style="font-size: 13px; color: rgb(0, 134, 179);">string</span><span style="font-size: 13px; color: rgb(51, 51, 51);"> order_by = ...;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">这个值 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 遵循 SQL 语法：用逗号分隔的字段列表。例如：</span><span style="font-size: 13px; color: rgb(199, 37, 78);">"foo,bar"</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。默认升序排列。</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 给字段添加后缀 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">" desc"</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 来表示降序。例如：</span><span style="font-size: 13px; color: rgb(199, 37, 78);">"foo desc,bar"</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">多余的空格可以忽略，</span><span style="font-size: 13px; color: rgb(199, 37, 78);">"foo,bar desc"</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">" foo , bar desc "</span><span style="font-size: 15px; color: rgb(51, 51, 51);">是相等的。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">请求校验</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果 API 方法有副作用，并且需要仅验证请求而不产生副作用，请求消息 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 包含一个字段：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">bool </span><span style="font-size: 13px; color: rgb(51, 51, 51);">validate_only = ...</span><span style="font-size: 13px; color: rgb(153, 153, 136);">;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">当此字段设置为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">true</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 时，服务端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不要（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 执行任何有副作用的操作，而是对请求进行校验。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">校验成功时 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 要返回</span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.OK</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，并且使用相同请求信息的完整请求 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.INVALID_ARGUMENT</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。注意，可能因为其他错误（比如 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.ALREADY_EXISTS</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 或竞态条件）此请求还是会失败。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">请求重入</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">对于网络 API，幂等是很重要的，因为当有网络异常时它们能够安全地进行重试。然而一些 API 并不容易实现幂等性，例如需要避免不必要重复的创建资源操作。对于这类情况，请求信息 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 包含一个唯一 ID（例如 UUID），这样服务端能够通过此 ID 来检测重复，保证请求只被处理一次。</span>

<span style="font-size: 13px; color: rgb(153, 153, 136);">// 服务端用于检测重复请求的唯一 ID</span><span style="font-size: 13px; color: rgb(153, 153, 136);">// 此字段应该命名为 `request_id`</span><span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">request_id = ...;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">因为客户端很可能没有接收到之前的响应，所以当检测到重复请求后，服务端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 返回之前成功的响应。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">枚举默认值</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">每个枚举定义 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 值开始，用于当枚举值没有明确指定时。API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在文档中说明如何处理 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);">值。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果有通用的默认行为，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用枚举值 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。API 应该在文档中说明期待的行为。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">如果没有通用的默认行为，枚举值 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 命名为 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">ENUM_TYPE_UNSPECIFIED</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 并且和错误 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">INVALID_ARGUMENT</span><span style="font-size: 15px; color: rgb(51, 51, 51);">一起使用。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">enum</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">Isolation</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{ /</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/ 未指定 ISOLATION_UNSPECIFIED = 0; // 快照读。如果所有读写都不能在并发事务中逻辑地序列化，则会发生冲突 SERIALIZABLE = 1; // 快照读。并发事务向同一行写入时导致冲突 SNAPSHOT = 2; ...}// 当未指定时，服务器将使用 SNAPSHOT 或更高的隔离级别Isolation level = 1;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">一个惯用名称 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 用于 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 值，例如，</span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.rpc.Code.OK</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是指定不存在错误的惯用方法。在这种情况下，</span><span style="font-size: 13px; color: rgb(199, 37, 78);">OK</span><span style="font-size: 15px; color: rgb(51, 51, 51);">与枚举类型中的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">UNSPECIFIED</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在语义上是相等的。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在存在本质上合理和安全的默认情况下，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 值。例如，在[资源视图]()枚举中 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">BASIC</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 是 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 值。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">语法句法</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">在某些 API 设计中，有必要为某些数据格式定义简单的语法，例如可接受的文本输入。为了在不同 API 中提供一致的开发体验和减少学习曲线，API 设计者 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">ISO 14977</span>](http://www.iso.org/iso/catalogue_detail?csnumber=26153)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 扩展的 Backus-Naur 表格（EBNF）句法来定义这些语法。</span>

<span style="font-size: 13px; color: rgb(0, 0, 128);">Production</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">= name</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"="</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">[ Expression ]</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">";"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">;</span><span style="font-size: 13px; color: rgb(0, 0, 128);">Expression</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">= Alternative {</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"|"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Alternative } ;</span><span style="font-size: 13px; color: rgb(0, 0, 128);">Alternative</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">= Term { Term } ;</span><span style="font-size: 13px; color: rgb(0, 0, 128);">Term</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">= name | TOKEN | Group | Option | Repetition ;</span><span style="font-size: 13px; color: rgb(0, 0, 128);">Group</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">=</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"("</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Expression</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">")"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">;</span><span style="font-size: 13px; color: rgb(0, 0, 128);">Option</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">=</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"["</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Expression</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"]"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">;</span><span style="font-size: 13px; color: rgb(0, 0, 128);">Repetition</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">=</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"{"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Expression</span> <span style="font-size: 13px; color: rgb(221, 17, 68);">"}"</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">;</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意：</span><span style="font-size: 13px; color: rgb(199, 37, 78);">TOKEN</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 表示在语法之外定义的终端。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">整数类型</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">在API 设计中，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用像 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">uint32</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">fixed32</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 这种无符号整型，这是因为一些重要的编程语言和系统（例如 Java, JavaScript 和 OpenAPI）不能很好地支持它们并且更容易导致溢出的问题。另一个问题是，不同的 API 很可能对同一个资源使用不匹配的有符号和无符号类型。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在大小和时间这种负数没有意义的类型中 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用且仅使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">-1</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 来表示特定的意义，例如到在文件结尾（EOF）、无穷的时间、无资源限额或未知的年纪。当这样使用负数时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在文档中明确说明以防止混淆。API 生成器也应该在文档中记录隐式默认值 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">0</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 表示的行为。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">部分响应</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">客户端有时只需要响应信息中的特定子集。一些 API 平台提供了对部分响应的原生支持。Google API 平台通过响应字段掩码来提供支持。对于任一 REST API 调用，有一个隐式的系统 query 参数 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">$fields</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，它是 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.FieldMask</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的 JSON 表示。在返回给客户端之前，响应消息会被 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">$fields</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 字段过滤。此行为是在 API 平台自动执行的。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">GET https:</span><span style="font-size: 13px; color: rgb(0, 153, 38);">//</span><span style="font-size: 13px; color: rgb(51, 51, 51);">library.googleapis.com</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/v1/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">shelves?</span><span style="font-size: 13px; color: rgb(0, 128, 128);">$fields</span><span style="font-size: 13px; color: rgb(51, 51, 51);">=name</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">资源视图</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">为了减少网络流量，允许客户端限制服务器在其响应中返回的资源的哪些部分是有用的，返回资源的视图而不是全部资源表示。API 中的资源视图是通过向请求添加参数来实现的，该参数允许客户端在响应中指定要接收资源的哪个视图。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">此参数：</span>

*   应该（should） 是枚举类型
*   必须（must） 命名为 <span style="font-size: 13px; color: rgb(199, 37, 78);">view</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">枚举中的每个值定义了资源的哪部分（字段）在响应中会被返回。文档中 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 明确记录每个 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">view</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 值会返回什么。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">package</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">google.example.library.v1;</span><span style="font-size: 13px; color: rgb(51, 51, 51);">service</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">Library</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">rpc</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">ListBooks(ListBooksRequest)</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">returns</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">(ListBooksResponse) { option (google.api.http) = { get: "/v1/{name=shelves/*}/books" } };}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">enum</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">BookView</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 响应中只包含作者、标题、ISBN 和唯一的图书 ID。这是默认值。</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">BASIC =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">0</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 返回所有信息，包括书中的内容</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">FULL =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span><span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">ListBooksRequest</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">name =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 指定返回图书资源的哪些部分</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">BookView view =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">对应的 URL：</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">GET https:</span><span style="font-size: 13px; color: rgb(0, 153, 38);">//</span><span style="font-size: 13px; color: rgb(51, 51, 51);">library.googleapis.com</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/v1/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">shelves</span><span style="font-size: 13px; color: rgb(0, 153, 38);">/shelf1/</span><span style="font-size: 13px; color: rgb(51, 51, 51);">books?view=BASIC</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">可以在 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">标准方法</span>](http://tailnode.tk/2017/03/google-api-design-guide/standard-methods/)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 一章中查看更多关于方法定义、请求和响应的内容。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">ETag</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">ETag 是一个不透明的标识符，允许客户端进行条件请求。为了支持 ETag，API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在资源定义中包含一个字符串字段 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">etag</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，它的语义 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须(must)</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 与 ETag的常用用法相匹配。通常，</span><span style="font-size: 13px; color: rgb(199, 37, 78);">etag</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 包含由服务器计算出的资源指纹。更多详细信息，请参阅</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">维基百科</span>](https://en.wikipedia.org/wiki/HTTP_ETag)<span style="font-size: 15px; color: rgb(51, 51, 51);">和 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">RFC 7232</span>](https://tools.ietf.org/html/rfc7232#section-2.3)<span style="font-size: 15px; color: rgb(51, 51, 51);">。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">ETags 可以强验证或弱验证，其中弱验证的ETag 以 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">W /</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 为前缀。在这种情况下，强验证意味着具有相同 ETag 的两个资源具有相同的内容和相同的额外字段（Content-Type）。这意味着强验证的 ETag 允许缓存稍后组装的部分响应。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">相反，具有相同弱验证 ETag 值的资源意味着这些表示在语义上是等效的，但不一定每字节都相同，因此不适合于字节范围请求的响应缓存。</span>

<span style="font-size: 13px; color: rgb(153, 153, 136);">// 强验证的 ETag（包含引号）</span><span style="font-size: 13px; color: rgb(221, 17, 68);">"1a2f3e4d5b6c7c"</span><span style="font-size: 13px; color: rgb(153, 153, 136);">// 弱验证的 ETag（包含前缀和引号）</span><span style="font-size: 13px; color: rgb(51, 51, 51);">W/</span><span style="font-size: 13px; color: rgb(221, 17, 68);">"1a2b3c4d5ef"</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">输出字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 可能希望将由客户端提供的字段和只由服务端在特定资源上返回的字段进行区分。对于仅输出的字段，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（shall）</span><span style="font-size: 15px; color: rgb(51, 51, 51);">记录字段属性。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">请注意，如果客户端在请求中设置了仅输出（output only）字段，或者客户端使用仅输出字段指定了一个 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.FieldMask</span><span style="font-size: 15px; color: rgb(51, 51, 51);">，则服务器 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 接受该请求而不能出错。这意味着服务器 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 忽略仅输出字段的存在及其任何指示。这个建议的原因是因为客户端通常会将服务器返回的资源重用为另一个请求的输入，例如一个获取到的 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">Book</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 将在 UPDATE 方法中被再次使用。如果要验证仅输出字段，客户端需要做清除输出字段的额外工作。</span>

<span style="font-size: 13px; color: rgb(51, 51, 51);">message</span> <span style="font-size: 13px; color: rgb(51, 51, 51);"></span> <span style="font-size: 13px; color: rgb(68, 85, 136);">Book</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">{</span> <span style="font-size: 13px; color: rgb(0, 134, 179);">string</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">name =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">1</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;</span> <span style="font-size: 13px; color: rgb(153, 153, 136);">// 只用做输出</span> <span style="font-size: 13px; color: rgb(51, 51, 51);">Timestamp create_time =</span> <span style="font-size: 13px; color: rgb(0, 128, 128);">2</span><span style="font-size: 13px; color: rgb(51, 51, 51);">;}</span>

<h1 id="9-Protocol-Buffers-v3"><code>使用 proto3</code></h1>

[<span style="font-size: 15px; color: rgb(0, 56, 132);">使用 Proto3</span>](https://segmentfault.com/a/1190000009157513) <span style="font-size: 15px; color: rgb(51, 51, 51);">这一章讨论在 API 设计中如何使用 Protocol Buffer。为了简化开发体验并提高运行效率，gPRC API</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在 API 定义时使用 Protocol Buffers 第 3 版（proto3）。</span>

[<span style="font-size: 15px; color: rgb(0, 154, 97);">Protocol Buffer</span>](https://github.com/google/protobuf)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 是一个为了定义数据结构和编程接口的语言独立平台独立的简单的接口定义语言（IDL）。它支持二进制和文本格式，并且能够在不同的平台不同的协议中使用。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">proto3 是 Protocol Buffer 的最新版本，与 proto2 比有如下改变：</span>

*   原始字段（primitive fields）不再支持 <span style="font-size: 13px; color: rgb(199, 37, 78);">hasField</span>。未设置的原始字段有语言相关的默认值。

*   消息字段仍然可用，可以使用编译器生成的 <span style="font-size: 13px; color: rgb(199, 37, 78);">hasField</span> 方法或与 null 进行比较或与由具体实现定义的哨兵值比较。

*   不再支持用户自定义的字段默认值
*   枚举定义 必须（must） 以 0 开始
*   不再支持 required 字段
*   不再支持扩展（extensions），请使用 <span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf.Any</span>

*   由于向后兼容性和运行时兼容性的原因，<span style="font-size: 13px; color: rgb(199, 37, 78);">google/protobuf/descriptor.proto</span> 特殊例外

*   删除了组语法（group）

<span style="font-size: 15px; color: rgb(51, 51, 51);">删除这些特性是为了让 API 的设计更加简洁可靠和提高性能。例如在记录日志前经常需要过滤一些敏感字段，但当字段是 required 时，这种操作是不可能的。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">查看 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Protocol Buffers</span>](https://developers.google.com/protocol-buffers/)<span style="font-size: 15px; color: rgb(51, 51, 51);"> 获取更多信息。</span>

<h1 id="10-Versioning"><code>版本管理</code></h1>

<span style="font-size: 15px; color: rgb(51, 51, 51);">这一章是网络 API 的版本控制指南。因为一个 API 服务 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 提供多个 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">API 接口</span>](http://tailnode.tk/2017/04/google-api-design-guide/glossary/#interface)<span style="font-size: 15px; color: rgb(51, 51, 51);">，</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">API 版本</span>](http://tailnode.tk/2017/04/google-api-design-guide/glossary/#version)<span style="font-size: 15px; color: rgb(51, 51, 51);">策略应用在 API 接口上而不是 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">API 服务</span>](http://tailnode.tk/2017/04/google-api-design-guide/glossary/#service)<span style="font-size: 15px; color: rgb(51, 51, 51);">。为了方便，下面的 API 表示 API 接口。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">网络 API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 使用 </span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">Semantic Versioning</span>](http://semver.org/)<span style="font-size: 15px; color: rgb(51, 51, 51);">。对于版本号 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">MAJOR.MINOR.PATCH</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：</span>

1.  有不兼容的升级时，增加 <span style="font-size: 13px; color: rgb(199, 37, 78);">MAJOR</span>
2.  添加了能向后兼容的新功能时，增加 <span style="font-size: 13px; color: rgb(199, 37, 78);">MINOR</span>
3.  修改了能向后兼容的 BUG 时，增加 <span style="font-size: 13px; color: rgb(199, 37, 78);">PATCH</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">根据 API 版本的不同，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">major 版本号</span><span style="font-size: 15px; color: rgb(51, 51, 51);">使用不同的规则：</span>

*   对于 version 1(v1)，major 部分 应该（should） 加上 proto 的包名字，例如 <span style="font-size: 13px; color: rgb(199, 37, 78);">google.pubsub.v1</span>。如果包名包含稳定的类型并且接口不会有不兼容的改变，major 部分 可以（may） 忽略版本号，例如：<span style="font-size: 13px; color: rgb(199, 37, 78);">google.protobuf</span> 和 <span style="font-size: 13px; color: rgb(199, 37, 78);">google.longrunning</span>。
*   对于除 v1 外的所有版本，major 版本号 必须（must） 加上 proto 的包名字。例如 <span style="font-size: 13px; color: rgb(199, 37, 78);">google.pubsub.v2</span>。

<span style="font-size: 15px; color: rgb(51, 51, 51);">对于 pre-GA 的发布（例如 alpha 和 beta），推荐在版本号中添加后缀，后缀 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 以 pre-release 的版本名（例如 alpha、beta）和可选的 pre-release 版本号组成。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">版本进度的例子：</span>

| <span style="color: rgb(0, 0, 0);">Version</span> | <span style="color: rgb(0, 0, 0);">Proto Package</span> | <span style="color: rgb(0, 0, 0);">Description</span> |
| --- | --- | --- |
| <span style="color: rgb(0, 0, 0);">v1alpha</span> | <span style="color: rgb(0, 0, 0);">v1alpha1</span> | <span style="color: rgb(0, 0, 0);">v1 alpha 发布</span> |
| <span style="color: rgb(0, 0, 0);">v1beta1</span> | <span style="color: rgb(0, 0, 0);">v1beta1</span> | <span style="color: rgb(0, 0, 0);">v1 beta 第一次发布</span> |
| <span style="color: rgb(0, 0, 0);">v1beta2</span> | <span style="color: rgb(0, 0, 0);">v1beta2</span> | <span style="color: rgb(0, 0, 0);">v1 beta 第二次发布</span> |
| <span style="color: rgb(0, 0, 0);">v1test</span> | <span style="color: rgb(0, 0, 0);">v1test</span> | <span style="color: rgb(0, 0, 0);">带有假数据的内部测试版</span> |
| <span style="color: rgb(0, 0, 0);">v1</span> | <span style="color: rgb(0, 0, 0);">v1</span> | <span style="color: rgb(0, 0, 0);">major 的版本是 v1，可正式使用</span> |
| <span style="color: rgb(0, 0, 0);">v1.1beta1</span> | <span style="color: rgb(0, 0, 0);">v1p1beta1</span> | <span style="color: rgb(0, 0, 0);">对 v1 版的首次小版本（minor）修改的 beta 发布</span> |
| <span style="color: rgb(0, 0, 0);">v1.1</span> | <span style="color: rgb(0, 0, 0);">v1</span> | <span style="color: rgb(0, 0, 0);">小版本升级到 v1.1</span> |
| <span style="color: rgb(0, 0, 0);">v2beta1</span> | <span style="color: rgb(0, 0, 0);">v2beta1</span> | <span style="color: rgb(0, 0, 0);">v2 beta 第一次发布</span> |
| <span style="color: rgb(0, 0, 0);">v2</span> | <span style="color: rgb(0, 0, 0);">v2</span> | <span style="color: rgb(0, 0, 0);">major 的版本是 v2，可正式使用</span> |

<span style="font-size: 15px; color: rgb(51, 51, 51);">minor 和 patch 的版本号 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 表现在 API 配置和文档中，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不要（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 写在 proto 的包名中。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">注意</span><span style="font-size: 15px; color: rgb(51, 51, 51);">：Google API 平台目前没有原生支持 minor 和 patch。对于每一个 major 版本只有一套文件和客户端的库。API 作者需要通过文档和发布日志手动记录 minor 和 patch。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">新的 major 版本号 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不要（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 依赖 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">相同 API</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 之前的 major 版本。在了解相关联的依赖性和稳定性风险后，API </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 依赖其他 API。一个稳定的 API 版本 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 只依赖其他 API 的最新稳定版本。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">在一段时间内，相同 API 的不同版本 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在单个客户端中同时工作。这样才能帮助客户端从旧版 API 平滑迁移到新版 API。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">只有当没有依赖后旧版本 API 才能被删除。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">被多个 API 共享的通用稳定的数据类型（例如日期和时间） </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 定义在单独的 proto 包中。如果有必要进行不兼容的修改，则 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 引入新的类型或包含新 major 版本的包。</span>

* * *

<span style="font-size: 28px; color: rgb(51, 51, 51);">向后兼容</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">定义什么是向后兼容的修改是比较困难的。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">下面列出了一些，但如果你有任何疑问，点击</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">这里</span>](http://tailnode.tk/2017/04/google-api-design-guide/compatibility/)<span style="font-size: 15px; color: rgb(51, 51, 51);">查看详情。</span>

* * *

<span style="font-size: 24px; color: rgb(51, 51, 51);">保持向后兼容的修改</span>

*   向 API 服务中添加 API 接口
*   向 API 接口中添加方法
*   向方法添加 HTTP 绑定
*   向请求信息添加字段
*   向响应信息添加字段
*   向枚举添加值
*   添加只输出（output-only）的资源字段

* * *

<span style="font-size: 24px; color: rgb(51, 51, 51);">破坏向后兼容的修改</span>

*   删除/重命名服务、接口、字段名、方法或枚举值
*   修改 HTTP 绑定
*   修改字段类型
*   修改资源名的格式
*   修改已有请求的可见性（visible behavior）
*   在 HTTP 定义中修改 URL 格式
*   在资源消息中添加读/写字段

<h1 id="11-Compatibility"><code>兼容性</code></h1>

<span style="font-size: 15px; color: rgb(51, 51, 51);">本章提供了有关</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">版本控制</span>](http://tailnode.tk/2017/04/google-api-design-guide/versioning/)<span style="font-size: 15px; color: rgb(51, 51, 51);">部分中给出的破坏和保持兼容性修改的详细说明。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">并不总是绝对清楚什么是不兼容的修改，这篇指南 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 被当成参考性的，而不是覆盖到所有情况。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">下面列出的这些规则只涉及客户端兼容性，默认 API 作者了解部署（包括实现细节的变化）的需求。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">一般的目标是服务端升级 minor 或 patch 不能影响客户端的兼容性：</span>

*   代码兼容：针对 1.0 编写的代码在 1.1 上编译失败
*   二进制兼容：针对 1.0 编译的代码与 1.1 客户端的库链接/运行失败（具体的细节依赖客户端，不同情况有不同变化）
*   协议兼容：针对 1.0 构建的程序与 1.1 服务端通信失败
*   语义兼容：所有组件都能运行但产生意想不到的结果

<span style="font-size: 15px; color: rgb(51, 51, 51);">简而言之：旧的客户端应该与相同 major 版本的新服务端正常工作，并且能够轻松地升级到新的 minor 版本。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">由于客户端使用了自动生成和手写的代码，除了理论上的基于协议的考虑，还有一些实际的问题。通过生成新版本的客户端库来测试你的修改，并保证测试通过。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">下面的讨论将 proto 信息分为三类：</span>

*   请求信息（例如 <span style="font-size: 13px; color: rgb(199, 37, 78);">GetBookRequest</span>）
*   响应信息（例如 <span style="font-size: 13px; color: rgb(199, 37, 78);">ListBooksResponse</span>）
*   资源信息（例如 <span style="font-size: 13px; color: rgb(199, 37, 78);">Book</span>，包括在其他资源消息中使用的任何消息）

<span style="font-size: 15px; color: rgb(51, 51, 51);">这三类有不同的规则，例如请求信息只会从客户端发送到服务端，响应信息只会从服务端发送到客户端，但资源信息一般会在两者之间互相发送。尤其是可被修改的资源需要根据读取/修改/写入的循环来考虑。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">保持向后兼容的修改</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">向 API 服务中添加 API 接口</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">从协议的角度看，这种修改总是安全的。唯一需要考虑的是客户端库可能已经通过手写的代码使用了新 API 接口的名字。如果新接口与其它完全正交，这种情况不太可能发生。如果是已存接口的简化版本，则很可能引起冲突。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">向 API 接口中添加方法</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">除非添加了一个与现有客户端库中方法冲突的方法，这种修改没有问题。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">一个会破坏兼容性的例子：如果有 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">GetFoo</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法，C# 代码生成器已经创建了 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">GetFoo</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">GetFooAsync</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法。因此从客户端角度来看，在 API 接口中添加 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">GetFooAsync</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 方法将会破坏兼容性。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">向方法添加 HTTP 绑定</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">假设绑定没有引入任何歧义，使服务端响应以前被拒绝的 URL 是安全的。当将现有操作应用于新的资源名称时，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">可能（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 会这样做。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">向请求信息添加字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">添加请求字段可以是兼容的，只要不指定该字段的客户端在新版本中与旧版本表现相同。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">会导致错误的最明显例子是分页：如果 API 的 v1.0 版本不支持，除非 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">page_size</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 默认值是无穷大（这样是不好的）才能在 v1.1 中加入分页。否则 v1.0 的客户端原本希望通过一次请求取得所有结果，但实际只能取到一部分。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">向响应信息添加字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">只要不改变其他响应字段的行为，就可以扩展不是资源的响应消息（例如ListBooksResponse），而不会破坏兼容性。即使导致冗余，任何在旧的响应消息中的字段也应该存在于新的响应中并保持它原来的语义。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">例如，1.0 中的一个查询请求的响应有 bool 类型的字段 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">contained_duplicates</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 来指示因为重复而忽略掉的结果。在 1.1 中，我们在 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">duplicate_count</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 字段中提供更详细的信息，尽管从 1.1 版本来看是多余的，但 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">contained_duplicates</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 字段 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 要保留。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">向枚举添加值</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">只在请求信息中使用的枚举类型可以自由扩展来添加新元素。例如，使用</span>[<span style="font-size: 15px; color: rgb(0, 154, 97);">资源视图</span>](http://tailnode.tk/2017/04/google-api-design-guide/design-patterns/#)<span style="font-size: 15px; color: rgb(51, 51, 51);">时，新的视图能够添加到新 minor 版本中。客户端从来不需要接收此枚举，所以也不需要关心它。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">对于资源消息和响应消息，默认假设客户端应该处理它意识不到的枚举值。但是 API 作者应该意识到编写能够正确处理新枚举值的代码可能是困难的。</span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 在文档中记录当遇到未知枚举值时客户端的期望行为。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">proto3 允许客户端接收它们不关心的值并且当执行重新序列化消息时会保持值不变，所以这样就不会打破读取/修改/写入循环的兼容性。JSON 格式允许发送数值，其中该值的“名称”是未知的，但是服务端通常不会知道客户端是否真正知道特定值。因此 JSON 客户端可能知道它们已经收到了以前对他们未知的值，但他们只会看到名称或数字而不是两个都有。在读取/修改/写入循环中将相同的值返回给服务端不应该修改这个值，因为服务端应该理解这两种形式。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">添加只输出（output-only）的资源字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 添加仅由服务端提供的资源实体中的字段。服务端 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 验证请求中的值是否有效，但是如果该值被省略则 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不能（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 失败。</span>

<span style="font-size: 28px; color: rgb(51, 51, 51);">破坏向后兼容的修改</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">删除/重命名服务、接口、字段名、方法或枚举值</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">从根本上说，如果客户端代码使用了某些字段，那么删除或重命名它将会破坏兼容性，并且 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 增加 major 版本号。引用旧名称的一些语言（如 C# 和 Java）在编译时会失败， 另一些语言会引起运行时异常或数据丢失。协议格式的兼容性在这里是无关紧要的。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">修改 HTTP 绑定</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">这里的</span><span style="font-size: 13px; color: rgb(199, 37, 78);">修改</span><span style="font-size: 15px; color: rgb(51, 51, 51);">实际指</span><span style="font-size: 13px; color: rgb(199, 37, 78);">删除</span><span style="font-size: 15px; color: rgb(51, 51, 51);">和</span><span style="font-size: 13px; color: rgb(199, 37, 78);">添加</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。例如，你想要支持 PATCH，但已发布的版本支持 PUT，或者已经使用了错误的自定义动词，你 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">可以（may）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 添加新的绑定，但是 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不要（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 移除旧的，因为和删除服务的方法一样会破坏兼容性。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">修改字段类型</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">尽管新类型是协议兼容的，能够改变客户端库自动生成的代码，因此 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 要升级 major 版本。会导致需要编译的静态类型的语言在编译期就发生错误。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">修改资源名的格式</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">资源 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">一定不能（must not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 修改名字－这意味着集合名不能被修改。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">不像其他大多数破坏兼容性的修改，这会影响 major 版本号：如果客户端期望使用 v2.0 访问在 v1.0 中创建的资源（或反过来），则应该在两个版本中使用相同的资源名称。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">对资源名的验证也 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 改变，原因如下：</span>

*   如果验证变严格，之前成功能请求现在可能会失败
*   如果比之前文档中记录的验证要宽松，依据之前文档的客户端可能会被破坏。客户端很可能在其他地方保存了资源名，并且对字符集和名字的长度敏感。或者，客户端可能会执行自己的资源名称验证来保持与文档一致。（例如，当开始支持 EC2 资源的长 ID 时，[<span style="color: rgb(0, 154, 97);">亚马逊向用户发出了许多警告并提供了迁移的时间</span>](https://aws.amazon.com/blogs/aws/theyre-here-longer-ec2-resource-ids-now-available/)）

<span style="font-size: 15px; color: rgb(51, 51, 51);">请注意这样的修改只能在 proto 的文档中可见。因此当评审 CL 时审查除注释外的修改是不够的。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">修改已有请求的可见性（visible behavior）</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">客户端总是依赖 API 的行为和语义，</span><span style="font-size: 15px; color: rgb(51, 51, 51);">即使没有明确支持或记录此行为</span><span style="font-size: 15px; color: rgb(51, 51, 51);">。因为在大多数情况下修改 API 的行为和语义在客户端看来是破坏性的。如果某行为不是加密隐藏的，你 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 假设用户已经依赖它了。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">因为这个原因加密分页 token 是个好主意，以防止用户创建自己的 token，以及防止当 token 行为发生变化时可能带来的不兼容性。</span>

<span style="font-size: 24px; color: rgb(51, 51, 51);">在 HTTP 定义中修改 URL 格式</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">除了上面列出的资源名称的变化，这里还要考虑两种类型的修改：</span>

*   自定义方法名：虽然不是资源名称的一部分，但自定义方法名称是 REST 客户端 POST 请求 URL 的一部分。更改自定义方法名称不应该破坏 gRPC 客户端，但是公共 API 必须假定它们具有 REST 客户端。
*   资源参数名：从 <span style="font-size: 13px; color: rgb(199, 37, 78);">v1/shelves/{shelf}/books/{book}</span> 到 <span style="font-size: 13px; color: rgb(199, 37, 78);">v1/shelves/{shelf_id}/books/{book_id}</span> 的修改不会影响替代的资源名称，但可能会影响代码生成。

<span style="font-size: 24px; color: rgb(51, 51, 51);">在资源消息中添加读/写字段</span>

* * *

<span style="font-size: 15px; color: rgb(51, 51, 51);">客户端会经常执行读取/修改/写入的操作。大多数客户端不支持它们意识不到的字段值，特别是 proto3 不支持。你可以指定任意消息类型（而不是原始类型）中缺失的字段表示更新时不会被修改，但这样使删除这样的字段变的困难。原始类型（包括 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">string</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 和 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">bytes</span><span style="font-size: 15px; color: rgb(51, 51, 51);">）不能简单地使用这种方法，因为明确地设置 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">int32</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 的值为 0 和不对它设置值在 proto3 中并没有区别。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">使用字段掩码来进行所有更新操作不会有问题，因为客户端不会隐式覆盖其不知道的字段。然而这是一个不寻常的决定，因为大部分 API 允许全部资源被更新。</span>

<h1 id="12-Directory-Structure"><code>目录结构</code></h1>

<span style="font-size: 15px; color: rgb(51, 51, 51);">通常使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">.proto</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 文件定义 API，使用 </span><span style="font-size: 13px; color: rgb(199, 37, 78);">.yaml</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 文件做为配置。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">每个 API 服务 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">必须（must）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 有一个 API 目录来存放定义文件和构建脚本。</span>

<span style="font-size: 15px; color: rgb(51, 51, 51);">API 目录 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">应该（should）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 遵循如下的标准结构：</span>

*   API 目录
*   配置文件

*   {service}.yaml<span style="font-size: 15px; color: rgb(51, 51, 51);">：主服务的配置文件，</span>google.api.Service<span style="font-size: 15px; color: rgb(51, 51, 51);"> 的 YAML 格式</span>
*   prod.yaml<span style="font-size: 15px; color: rgb(51, 51, 51);">：产品环境配置文件</span>
*   staging.yaml<span style="font-size: 15px; color: rgb(51, 51, 51);">：Staging 环境配置文件</span>
*   test.yaml<span style="font-size: 15px; color: rgb(51, 51, 51);">：测试环境配置文件</span>
*   local.yaml<span style="font-size: 15px; color: rgb(51, 51, 51);">：本地环境配置文件</span>

*   接口定义

*   v[0-9]*/*<span style="font-size: 15px; color: rgb(51, 51, 51);">：每一个子目录包含 API 的一个主版本，主要存放原型文件和构建脚本</span>
*   {subapi}/v[0-9]*/*<span style="font-size: 15px; color: rgb(51, 51, 51);">：每一个 </span>{subapi}<span style="font-size: 15px; color: rgb(51, 51, 51);"> 目录包含子 API 的接口定义。每个子 API 可以有它独立的主版本号</span>
*   type/*<span style="font-size: 15px; color: rgb(51, 51, 51);">： 包含类型定义的原型文件，包括这些：在不同 API 间共享的类型、不同 API 版本间共享的类型或 API 与服务实现间共享的类型。一旦发布，</span>type/*<span style="font-size: 15px; color: rgb(51, 51, 51);"> 中定义的类型 </span><span style="font-size: 15px; color: rgb(51, 51, 51);">不应该（should not）</span><span style="font-size: 15px; color: rgb(51, 51, 51);"> 有破坏兼容性的修改。</span>

<h1 id="13-File-Structure"><code>文件结构</code></h1>

