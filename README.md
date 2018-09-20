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

