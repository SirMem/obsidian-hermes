---
creation_date: 2026-03-10
aliases:
  - mcp-server-bocha
  - Java MCP案例
  - Bocha MCP Server
tags:
  - java
  - spring-boot
  - mcp
  - spring-ai
  - retrofit
  - architecture
status: growing
belongs_to_canvas: []
---

# Java MCP案例：mcp-server-bocha 从零理解架构与调用链

## 一、先别急着看代码，先搞懂这个项目到底是干什么的

很多人第一次看到这种项目，会被一堆词吓到：

- MCP
- Spring AI
- Tool
- Port
- Retrofit
- Web Search

如果直接冲进代码，很容易越看越乱。

所以我们先用一句最朴素的话讲清楚：

> 这个项目本质上是在做一个“给大模型调用的 Java 工具服务”，它把“博查搜索 API”包装成了一个 MCP 工具，让外部 AI 可以通过这个服务去联网搜索网页。

也就是说，这个项目不是一个普通网站，也不是一个传统 CRUD 后台，而是一个：

> **“把外部搜索能力封装成 MCP Tool 的 Java 服务端案例”。**

你可以把它想成一个“中间翻译层”：

- 上游：大模型 / MCP Client
- 中间：这个 Java 服务
- 下游：Bocha 的真实 HTTP 搜索接口

这个服务干的事就是：

> 上面有人来调用工具，我负责接住请求、做参数校验、转成 HTTP 请求去调用博查，再把结果整理后返回。

---

## 二、什么是 MCP？你现在可以怎么理解？

如果你完全零基础，那你先不要背官方定义。

你先把 MCP 理解成：

> **一种让大模型能够“按统一方式调用外部工具”的协议或接口约定。**

大白话一点说：

- 大模型本身会聊天
- 但它不能天然访问你写的 Java 方法
- 也不能天然知道“这个工具有哪些参数、返回什么格式”

MCP 的作用，就是把这些工具能力用一种标准方式暴露出来，让模型可以像“调用工具”一样去调用它。

所以这个项目的重点不是“做搜索网站”，而是：

> **把 Java 代码里的一个搜索方法，注册成 MCP Tool。**

这样别的 AI 客户端就能通过 MCP 协议调用这个工具。

---

## 三、这个项目从大方向上分成几层？

如果你刚学架构，我建议你不要一上来就陷进每个类名里，而是先看大分层。

这个项目虽然不大，但结构其实挺清楚，可以粗略理解成 5 层。

### 1. 启动与装配层
核心类：
- `McpServerBochaApplication`

这一层负责：
- 启动 Spring Boot
- 创建 HTTP 调用对象
- 把工具注册给 MCP

也就是说，它像一个“总装厂”。

---

### 2. Tool 暴露层
核心类：
- `BochaTool`

这一层负责：
- 定义真正暴露给 MCP 的工具方法
- 给方法打上 `@Tool`
- 把请求交给后面的业务层

你可以把它理解成：

> **MCP 世界看到的门面。**

---

### 3. 业务接口层（Port）
核心接口和实现：
- `IBochaPort`
- `BochaPort`

这一层负责：
- 定义“搜索能力”这个业务接口
- 实现参数校验
- 组装 HTTP 请求
- 解析 HTTP 返回
- 把外部接口结果转换成 Tool 返回格式

这层非常关键，因为它是：

> **Tool 层和 HTTP 调用层之间的业务中转站。**

---

### 4. 外部 HTTP 调用层
核心接口：
- `IBochaHttp`

这一层负责：
- 描述博查搜索 API 的 HTTP 调用方式
- 指定路径、请求头、请求体

这层的本质是：

> **把“远程 API”抽象成一个 Java 接口。**

---

### 5. 配置与数据模型层
核心类：
- `BochaProperties`
- `BochaSearchToolRequest`
- `BochaSearchToolResponse`
- `BochaWebSearchHttpRequest`
- `BochaWebSearchHttpResponse`
- `application.yml`

这一层负责：
- 读取配置
- 承接请求参数
- 承接返回结果
- 在不同层之间传数据

所以这个项目虽然不大，但并不是一坨写在一起，而是有比较明显的职责拆分。

---

## 四、这个项目最核心的依赖有哪些？

看 Java 项目，第一步通常就是看 `pom.xml`，因为它能告诉你这个项目“打算靠什么活着”。

这个项目的依赖，不算多，但每个都很关键。

### 1. Spring Boot
父项目：
- `spring-boot-starter-parent:3.4.3`

它的作用你可以先理解成：

> 给整个项目提供基础运行环境。

比如：
- 启动应用
- 自动装配 Bean
- 配置管理
- 测试支持

也就是说，没有 Spring Boot，这个项目就不会像现在这样“启动起来就是一个服务”。

---

### 2. Spring AI MCP Server
关键依赖：
- `spring-ai-mcp-server-webflux-spring-boot-starter`
- `spring-ai-mcp-server-spring-boot-starter`

这两个依赖是这个项目最“像 MCP 项目”的地方。

它们的作用可以大白话理解成：

> 帮你把 Java 里的工具方法，按 MCP Server 的方式暴露出去。

如果没有它们，`@Tool` 这种能力就没法顺畅接入 MCP 生态。

---

### 3. Retrofit
关键依赖：
- `retrofit`
- `converter-jackson`
- `adapter-rxjava2`

这里真正用上的核心是 `retrofit` 和 `converter-jackson`。

Retrofit 你可以把它理解成：

> **一种“把 HTTP 调用写得像 Java 接口调用”的工具。**

正常发 HTTP 请求，你可能要自己拼 URL、Header、Body、解析 JSON。

但用了 Retrofit 之后，你只要定义一个接口，比如：

```java
@POST("v1/web-search")
Call<BochaWebSearchHttpResponse> webSearch(...)
```

程序运行时，它会帮你生成真正的 HTTP 调用对象。

这就是为什么 `IBochaHttp` 看起来像接口，却能真的发请求。

---

### 4. Lombok
作用：
- 自动生成 getter/setter
- 自动生成日志对象等样板代码

所以你会看到很多类上写了：
- `@Data`
- `@Slf4j`
- `@Builder`

这不是魔法，而是 Lombok 在帮你省样板代码。

---

### 5. spring-dotenv
这个依赖用来帮项目读取 `.env` 或环境变量。

在这个项目里，最关键的就是：

- `BOCHA_API_KEY`

因为真正调用博查接口时，需要 API Key。

---

## 五、程序启动时到底发生了什么？

理解架构最好的方式，不是背类名，而是看：

> **应用启动时，系统把什么东西准备好了？**

入口类是：
- `McpServerBochaApplication`

它里面做了两件非常关键的事。

### 第一件事：创建 HTTP 客户端
代码里有一个 Bean：

- `createHttp(BochaProperties bochaProperties)`

它做的事情是：
1. 从配置里拿 `baseUrl`
2. 用 Retrofit 构造一个 HTTP 客户端
3. 生成 `IBochaHttp` 的实现对象

你可以这样理解：

> 项目启动时，先造好一个“专门拿来调用博查搜索接口的电话机”。

以后谁要调用博查 API，就用这个电话机。

---

### 第二件事：把工具注册给 MCP
另一个 Bean：

- `createTool(BochaTool bochaTool)`

它做的事情是：
1. 拿到 `BochaTool` 这个对象
2. 交给 `MethodToolCallbackProvider`
3. 让里面带 `@Tool` 的方法被识别成 MCP 工具

大白话就是：

> 把这个 Java 类里的方法，登记成“可被外部模型调用的工具”。

所以启动类的本质不是写业务，而是：

> **装配整个系统，把“HTTP能力”和“Tool能力”接上线。**

---

## 六、真正暴露给 MCP 的工具是谁？

答案是：
- `BochaTool`

这里你会看到一个非常关键的方法：

- `webSearch(BochaSearchToolRequest toolRequest)`

它上面有一个注解：

```java
@Tool(description = "通过博查进行联网网页搜索，必须提供 query 和 freshness")
```

这句的意义非常大。

它相当于在告诉 MCP / 模型：

- 这里有个工具叫 `webSearch`
- 它的用途是联网网页搜索
- 它需要 `query` 和 `freshness`

也就是说，`BochaTool` 这一层，做的不是底层 HTTP 细节，而是：

> **把业务能力包装成“模型能理解、能调用”的工具入口。**

这个类本身很薄，只做两件事：
- 打日志
- 把请求交给 `bochaPort`

这也是一个很常见的好味道：

> 门面层尽量薄，真正逻辑下沉到后面。

---

## 七、真正的业务处理发生在哪里？

真正的核心逻辑，在：
- `BochaPort`

这个类是整个项目最值得细看的地方，因为它真正把“工具请求”翻译成了“外部 HTTP 请求”。

它干了几类事情。

### 1. 参数校验
它会检查：
- `toolRequest` 是否为空
- `query` 是否为空
- `freshness` 是否为空
- `apiKey` 是否配置

如果这些有问题，它不会盲目往下走，而是直接返回一个结构化的错误结果。

这一步非常重要，因为它说明这个项目不是“直接透传”，而是有自己的防线。

---

### 2. 组装 HTTP 请求对象
它把 Tool 请求：
- `BochaSearchToolRequest`

转换成 HTTP 请求：
- `BochaWebSearchHttpRequest`

这里你会发现两个请求对象长得像，但不是同一个。

这恰恰说明架构上做了分层：

- Tool 层有 Tool 层自己的 DTO
- HTTP 层有 HTTP 层自己的 DTO

为什么这样做？

因为：

> 上层输入格式，不一定要和下游 API 格式完全绑死。

这是解耦思路。

---

### 3. 调用真实博查 API
接着它会调用：
- `bochaHttp.webSearch(...)`

这里就是正式走 Retrofit 发 HTTP 请求了。

传进去的关键东西有：
- `Authorization: Bearer + apiKey`
- JSON 请求体

然后执行：
- `call.execute()`

也就是说，这里走的是**同步调用**。

---

### 4. 处理 HTTP 返回结果
这一步也很关键。

它不是直接把 Bocha 返回原样扔出去，而是做了多层处理：

- HTTP 是否成功
- 响应体是否为空
- 业务 code 是否为 `200`
- `message` / `msg` 哪个字段有值
- `data` 里是否还有一层嵌套
- `webPages.value` 是否为空

最后再把真正需要的数据抽出来，转换成：
- `BochaSearchToolResponse`

也就是说，这个类还承担了：

> **外部响应清洗 + 结果格式统一**

---

## 八、为什么这里要有两套 DTO？

刚学架构的人，经常会觉得：

“请求不就是请求吗？为什么要写两套对象？”

这个项目正好可以帮你理解。

它至少有两类 DTO：

### 1. Tool DTO
- `BochaSearchToolRequest`
- `BochaSearchToolResponse`

这是给 MCP / 上游调用者看的。

它强调的是：
- 工具需要什么参数
- 工具返回什么结果
- 字段描述是什么

所以它上面会有很多：
- `@JsonProperty`
- `@JsonPropertyDescription`

这些不是给 Java 自己看的，而更像是在定义“工具的输入输出契约”。

---

### 2. HTTP DTO
- `BochaWebSearchHttpRequest`
- `BochaWebSearchHttpResponse`

这是给真实博查接口看的。

它强调的是：
- 下游 API 要什么字段
- 下游 API 会返回什么结构

所以这里面会出现很多很底层、很贴 API 的字段，比如：
- `webPages`
- `summary`
- `rankingResponse`
- `images`
- `videos`

虽然当前工具只主要用到了网页搜索部分，但 DTO 把别的结构也提前接住了。

---

## 九、这个项目的调用链到底是怎么走的？

这部分是你最该掌握的，因为它把“架构图”变成了“动态过程”。

如果从外到内看，一次完整调用大概是这样：

```text
MCP Client / 大模型
    -> 调用 MCP 工具 webSearch
    -> BochaTool.webSearch(...)
    -> IBochaPort / BochaPort.webSearch(...)
    -> IBochaHttp.webSearch(...)
    -> Bocha API: POST /v1/web-search
    -> 返回 HTTP 响应
    -> BochaPort 解析并转换
    -> BochaTool 返回 ToolResponse
    -> MCP Client / 大模型拿到搜索结果
```

如果你再口语化一点理解：

1. 模型说：帮我搜一下这个内容
2. MCP 找到 `BochaTool` 这个工具
3. `BochaTool` 把活交给 `BochaPort`
4. `BochaPort` 检查参数、准备请求
5. `BochaPort` 调用 `IBochaHttp` 发请求到 Bocha
6. Bocha 返回搜索结果
7. `BochaPort` 把结果清洗成统一格式
8. 工具把结果返回给模型

这就是这个项目最核心的一条调用链。

---

## 十、配置是怎么进来的？

配置类是：
- `BochaProperties`

配置文件是：
- `application.yml`

这里定义了：

```yaml
bocha:
  base-url: https://api.bocha.cn/
  api-key: ${BOCHA_API_KEY}
```

也就是说：

- `base-url` 写在配置文件里
- `api-key` 从环境变量里取

然后通过：
- `@ConfigurationProperties(prefix = "bocha")`

Spring 会自动把这些值注入到 `BochaProperties` 里。

你可以把它理解成：

> 配置文件里的内容，被自动装进了一个 Java 配置对象。

这样别的类就不需要手动读 yaml 了，直接注入 `BochaProperties` 就行。

---

## 十一、这个项目有没有测试？测试测了什么？

有一个测试类：
- `BochaSearchTest`

它做的事情很简单：

1. 构造一个 `BochaSearchToolRequest`
2. 填入查询词和 freshness
3. 直接调用 `bochaTool.webSearch(request)`
4. 打印返回结果

这个测试更像是一个：

> **集成测试 / 冒烟测试**

它不是细粒度单元测试，而是在验证：

- Spring 能不能把项目启动起来
- Tool 能不能被注入
- 整条调用链能不能跑通

不过你也要注意，`pom.xml` 里把测试默认跳过了：
- `skipTests=true`

所以它有测试类，但构建时默认不执行。

---

## 十二、如果从零基础角度总结，这个项目最值得你学什么？

我觉得这个 Java MCP 案例最值得你学的，不是某个 API 细节，而是下面这 5 点。

### 1. 学会把“外部能力”封装成工具
这个项目不是直接把 HTTP 接口暴露出去，而是先包装成 Tool。

这是一种更适合 AI 时代的服务封装方式。

---

### 2. 学会看分层
虽然项目不大，但已经有：
- 启动装配层
- Tool 层
- Port 业务层
- HTTP 调用层
- DTO / 配置层

你以后看别的项目，也要先找这些层次，而不是被文件名带着跑。

---

### 3. 学会看调用链
一个项目不是“有哪些类”，而是：

> 请求来了以后，怎么一步步流动。

你现在应该能看懂：
- Tool 接请求
- Port 做业务
- Http 发请求
- DTO 传数据
- Properties 供配置

---

### 4. 学会看依赖背后的职责
不是背依赖名，而是要知道：
- Spring Boot 负责运行环境
- Spring AI MCP 负责工具暴露
- Retrofit 负责 HTTP 调用
- Lombok 负责样板代码简化
- dotenv 负责环境变量支持

---

### 5. 学会区分“业务接口”和“外部接口”
这个项目里：
- `IBochaPort` 是业务层接口
- `IBochaHttp` 是下游 HTTP 接口

这两个名字都叫接口，但角色完全不一样。

这点很多初学者一开始很容易混。

---

## 十三、如果把整个项目压缩成一句最适合你记住的话

如果你今天只记一句，我希望你记这句：

> **`mcp-server-bocha` 是一个基于 Spring Boot + Spring AI MCP 的 Java 工具服务，它把博查搜索 API 包装成了一个可被大模型调用的 MCP Tool。**

再展开一点就是：

- MCP 负责“让模型能调工具”
- Tool 负责“暴露业务能力”
- Port 负责“承接业务逻辑”
- Retrofit 负责“访问真实博查接口”
- DTO 和配置类负责“在各层之间传递数据和配置”

你把这五句记住，这个项目的骨架就不会乱。

---

## 十四、我现在对这个案例的最终理解

这不是一个传统的“做页面、做数据库”的 Java 项目，而是一个非常典型的：

- AI 工具化
- 服务封装化
- 分层清晰
- 调用链短而完整

的小型 MCP Server 案例。

它最适合零基础学习的地方就在于：

- 项目不大
- 链路完整
- 涉及到 Spring Boot、配置、HTTP、接口分层、Tool 注册
- 又能帮助你理解“AI 工具服务”到底是怎么落地的

所以拿它入门 MCP Java 项目，其实挺合适。

---

## 十五、画图式理解：把架构和调用链看成一张图

如果只看类名，这个项目很容易显得碎；但如果把它画成图，你就会发现它其实就是一个非常清晰的“中间翻译层”。

### 1. 全景图：这个项目在中间干什么

```text
        大模型 / MCP Client
                |
                |  调用工具 webSearch
                v
        +--------------------+
        |     BochaTool      |
        |  MCP工具暴露层      |
        +--------------------+
                |
                |  交给业务层处理
                v
        +--------------------+
        |     BochaPort      |
        | 参数校验 + 组装请求  |
        | 解析响应 + 转换结果  |
        +--------------------+
                |
                |  调用远程 HTTP API
                v
        +--------------------+
        |     IBochaHttp     |
        | Retrofit HTTP接口   |
        +--------------------+
                |
                |  POST /v1/web-search
                v
        +--------------------+
        |     Bocha API      |
        | 真实联网搜索服务     |
        +--------------------+
                |
                |  返回搜索结果 JSON
                v
        +--------------------+
        |     BochaPort      |
        | 清洗结果并转成工具返回 |
        +--------------------+
                |
                v
        +--------------------+
        |     BochaTool      |
        +--------------------+
                |
                v
        大模型 / MCP Client
```

这张图最核心的意思是：

> `mcp-server-bocha` 本质上是一个“中间翻译层”：上面接 MCP 工具调用，下面接真实的 Bocha 搜索 API。

### 2. 项目内部结构图

```text
+--------------------------------------------------+
|              McpServerBochaApplication           |
| 启动 Spring Boot / 注册 Bean / 注册 MCP Tool      |
+--------------------------------------------------+

+----------------------+    +----------------------+
|      BochaTool       | -> |      IBochaPort      |
|   工具暴露层          |    |   业务接口层          |
+----------------------+    +----------------------+
                                      |
                                      v
                           +----------------------+
                           |      BochaPort       |
                           |   业务实现层          |
                           +----------------------+
                                      |
                                      v
                           +----------------------+
                           |      IBochaHttp      |
                           |   HTTP 调用层         |
                           +----------------------+
                                      |
                                      v
                           +----------------------+
                           |      Bocha API       |
                           |   第三方搜索服务       |
                           +----------------------+

配套支撑层：
- BochaProperties
- Tool Request/Response DTO
- HTTP Request/Response DTO
- application.yml
```

### 3. 最核心的调用链图

```text
[1] MCP Client / 大模型
        |
        | 调用工具 webSearch(query, freshness)
        v
[2] BochaTool.webSearch(...)
        |
        | 把请求交给业务层
        v
[3] BochaPort.webSearch(...)
        |
        | 1. 校验 query / freshness / apiKey
        | 2. 转成 HTTP 请求对象
        v
[4] IBochaHttp.webSearch(...)
        |
        | 发 POST 请求到 https://api.bocha.cn/v1/web-search
        v
[5] Bocha API
        |
        | 返回 JSON
        v
[6] BochaPort
        |
        | 1. 判断 HTTP 是否成功
        | 2. 判断业务 code 是否成功
        | 3. 提取 webPages.value
        | 4. 转成 ToolResponse
        v
[7] BochaTool
        |
        v
[8] MCP Client / 大模型拿到结果
```

### 4. DTO 流动图

```text
BochaSearchToolRequest
   -> BochaWebSearchHttpRequest
      -> BochaWebSearchHttpResponse
         -> BochaSearchToolResponse
```

这张图对应的是：
- 上游用 Tool DTO
- 下游 HTTP 用 HTTP DTO
- 中间由 `BochaPort` 负责转换

### 5. 最适合记忆的简图

```text
                [大模型 / MCP Client]
                         |
                         v
                 [BochaTool 工具入口]
                         |
                         v
                 [BochaPort 业务核心]
                  /      |                        /       |                        v        v         v
         参数校验   请求转换   结果清洗
                         |
                         v
                 [IBochaHttp HTTP调用]
                         |
                         v
                   [Bocha Search API]
                         |
                         v
                 [搜索结果返回给模型]
```

如果把整个项目再压缩成 4 句话：

1. `mcp-server-bocha` 不是普通网站，而是一个 MCP 工具服务。
2. `BochaTool` 是工具入口。
3. `BochaPort` 是业务核心。
4. `IBochaHttp` 是对外 HTTP 调用层。

## 十五、接下来最适合继续学什么

如果继续往下学，我建议你下一步可以继续拆下面几个方向：

1. **把这个项目画成一张调用链图**
2. **逐个讲 DTO 到底是怎么设计的**
3. **讲 `@Tool` 在 Spring AI MCP 里到底起什么作用**
4. **带你把这个项目本地跑起来**
5. **和普通 Spring Boot 接口项目做对比**

---

## 十六、接下来最适合继续学什么

- [[MCP]]
- [[Spring AI]]
- [[Spring Boot]]
- [[Retrofit]]
- [[Tool Calling]]
- [[Java 项目分层]]
- [[配置绑定]]
- [[DTO]]
- [[端口适配器]]
