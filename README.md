# Google API Design
Google API Documents Chinese Documents

## 目录
1. [简介](#1-Introduction)


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
