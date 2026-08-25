

---

# 浏览器原理之 CORS（跨源资源共享）详解

## 一、浏览器的同源策略（Same-Origin Policy, SOP）

### 1.1 什么是同源策略

同源策略是浏览器最核心、最基础的安全机制，它限制了从一个源（origin）加载的文档或脚本如何与另一个源的资源进行交互。其核心目的是**阻隔恶意文档，减少可能被攻击的媒介**。

具体来说，它可以防止互联网上的恶意网站在浏览器中运行 JS 脚本，从第三方网络邮件服务（用户已登录）或公司内网（因没有公共 IP 地址而受到保护）窃取数据。

### 1.2 "同源"的判定标准

两个 URL 属于同源，当且仅当以下三个要素**完全一致**：

| 要素 | 说明 | 示例 |
| --- | --- | --- |
| **协议（Protocol）** | http / https / ftp 等 | `https://` |
| **域名（Domain）** | 完整主机名 | `example.com` |
| **端口（Port）** | 端口号 | `443` |

以 `http://store.company.com/dir/page.html` 为基准：

| 对比 URL | 是否同源 | 原因 |
| --- | --- | --- |
| `http://store.company.com/dir2/other.html` | ✅ 同源 | 协议、域名、端口均相同 |
| `http://store.company.com/dir/inner/another.html` | ✅ 同源 | 仅路径不同 |
| `https://store.company.com/secure.html` | ❌ 不同源 | 协议不同（http vs https） |
| `http://store.company.com:81/dir/etc.html` | ❌ 不同源 | 端口不同 |
| `http://news.company.com/dir/other.html` | ❌ 不同源 | 域名不同 |

### 1.3 同源策略限制了什么

同源策略主要限制以下操作：

- **`XMLHttpRequest` / `Fetch API`**：使用这些 API 的 Web 应用只能从加载应用程序的同一个源请求 HTTP 资源，除非响应包含正确的 CORS 响应头。
- **DOM 访问**：不同源的页面之间无法通过 JavaScript 互相访问 DOM（如 `iframe.contentDocument`）。
- **Cookie / LocalStorage / IndexedDB**：不同源之间无法读取对方的存储数据。

但同源策略**并非完全禁止**跨源请求。以下操作不受同源策略限制（或有特殊处理）：

- `<img>`、`<script>`、`<link>`、`<iframe>`、`<video>` 等 HTML 标签的资源加载
- 表单提交（form submit）
- 页面跳转（redirect）

> 
> 关键区别：同源策略限制的是**脚本读取跨源响应的内容**，而不是**阻止跨源请求本身的发送**。

---

## 二、CORS 的背景

### 2.1 矛盾的产生

随着 Web 应用的发展，前后端分离架构、微服务、CDN、第三方 API 等模式日益普遍，Web 应用越来越需要**跨源获取数据**。例如：

- 前端页面部署在 `https://www.example.com`，后端 API 部署在 `https://api.example.com`
- 网站需要调用第三方地图服务、支付接口、天气 API
- 前端需要从 CDN（不同域名）加载字体、图片等资源

然而，同源策略严格禁止脚本读取跨源响应。如果没有一种机制来"有控制地放开"同源策略，现代 Web 应用的架构将无法成立。

### 2.2 早期的变通方案

在 CORS 出现之前，开发者使用了各种"黑客手段"来绕过同源策略：

- **JSONP（JSON with Padding）**：利用 `<script>` 标签不受同源策略限制的特性，通过回调函数获取数据。但只支持 GET 请求，且存在安全风险。
- **服务器代理（Proxy）**：同源服务器作为代理，转发跨源请求。增加了服务器负担和延迟。
- **`document.domain`**：仅适用于子域名之间的场景，局限性大。
- **`window.postMessage`**：用于窗口间通信，但不直接解决 HTTP 请求跨源问题。

缺点: 这些方案功能受限，不够安全，会增加架构复杂度。

### 2.3 CORS 的诞生与标准化历程

CORS 的发展历程如下：

- **2004 年**：最初的提案名为"Access Control for Cross-Site Requests"
- **2009 年**：文档修订并更名为"Cross-Origin Resource Sharing"，成为 W3C 工作草案
- **2014 年**：CORS 成为 W3C 正式推荐标准（Recommendation）
- **后续**：CORS 协议被整合进 **Fetch Living Standard**（由 WHATWG 维护），成为现代浏览器 `fetch()` 和 `XMLHttpRequest` 的底层规范基础。

浏览器从 2009 年左右开始支持 CORS，到 2015 年 7 月起，该特性已在主流浏览器中广泛可用。

---

## 三、CORS 的定义

**跨源资源共享（Cross-Origin Resource Sharing，CORS）** 是一种基于 **HTTP 头**的机制，该机制通过允许服务器标示除了它自己以外的其他源（域、协议或端口），使得浏览器允许这些源访问加载自己的资源。

CORS 是一种机制，使用额外的 HTTP 头来告诉浏览器，让运行在一个 origin（域）上的 Web 应用被准许访问来自不同源服务器上的指定资源。

学术文献中的定义：CORS 是一种用于放松同源策略（SOP）所施加的安全规则的机制，对于那些依赖跨站数据交换来正常运行的网站来说，SOP 可能过于严格。CORS 通过一系列 HTTP 头，允许信任不同于网站域名的源。

### 核心要点

1. **基于 HTTP 头**：CORS 的全部能力通过一组 `Access-Control-*` 响应头和请求头实现，不需要修改 HTTP 协议本身。
2. **服务器主导**：是否允许跨源访问由**服务器**决定，浏览器只是执行者。服务器通过响应头声明策略，浏览器据此判断是否将响应内容暴露给脚本。
3. **浏览器强制**：CORS 校验由**浏览器**自动完成，开发者无法在前端代码中绕过。`XMLHttpRequest` 和 `Fetch API` 都遵循同源策略和 CORS 协议。

---

## 四、CORS 的核心作用

### 4.1 在安全与可用性之间取得平衡

CORS 的核心作用是**在保持同源策略安全底线的前提下，为合法的跨源数据交换提供标准化的、可控的开放通道**。

CORS 允许 Web 应用跨源共享数据，因此是 Web 平台最强大的功能之一，但它是在一个充满了"假定这种能力不可能"的现有应用的环境中演化出来的。

### 4.2 具体作用拆解

| 作用 | 说明 |
| --- | --- |
| **受控放开同源策略** | 不是完全取消同源策略，而是让服务器可以精确声明"哪些源、用什么方法、带什么头、是否带凭证"可以访问 |
| **保护用户数据安全** | 默认情况下跨源响应不暴露给脚本，只有服务器明确授权后才暴露，防止恶意网站窃取数据 |
| **支持现代 Web 架构** | 使前后端分离、微服务、CDN、第三方 API 等架构模式成为可能 |
| **预检机制防止副作用** | 对可能修改服务器数据的请求（如 PUT/DELETE/自定义头），先发 OPTIONS 预检，避免跨源请求对服务器数据产生未预期的影响 |
| **统一标准化方案** | 取代 JSONP、代理等非标准变通方案，提供统一、安全、全方法支持的跨域方案 |

### 4.3 重要澄清：CORS 不是什么

**CORS 不是针对跨源攻击（如 CSRF）的保护机制**。CORS 是对同源策略的扩展和放松，如果网站的 CORS 策略配置不当，反而可能带来跨域攻击风险。


---

## 五、CORS 的应用场景

### 5.1 前后端分离架构

最常见的场景。前端静态资源部署在一个域名（如 `https://www.example.com`），后端 API 部署在另一个域名（如 `https://api.example.com`），前端通过 `fetch` 或 `XMLHttpRequest` 调用后端接口。

### 5.2 第三方 API 集成

网站需要调用第三方服务的 API，例如：

- 地图服务（高德、百度、Google Maps）
- 支付接口（支付宝、微信支付、Stripe）
- 天气、股票、新闻等数据 API
- 社交媒体分享/登录接口（OAuth）

### 5.3 CDN 资源加载

Web 字体（`@font-face`）、图片、脚本等资源托管在 CDN 域名上，需要通过 CORS 允许主站域名访问。特别是 Web 字体，浏览器要求必须通过 CORS 协议加载跨源字体。

### 5.4 微服务架构

不同微服务部署在不同子域名或端口，服务之间通过 HTTP 调用，前端页面可能需要直接访问多个服务的 API。

### 5.5 开发环境

前端开发服务器（如 `http://localhost:3000`）与后端开发服务器（如 `http://localhost:8080`）端口不同，属于跨源，需要配置 CORS 才能联调。

### 5.6 WebGL / Canvas 跨源图片

在 Canvas 中绘制跨源图片，或在 WebGL 中使用跨源纹理时，如果没有正确配置 CORS，画布会被"污染"（tainted），无法通过 `getImageData()` 等方法读取像素数据。

### 5.7 跨源数据可视化与分析

仪表盘、BI 工具需要从多个不同源的数据源拉取数据进行整合展示。

---

## 六、CORS 的实现手段

CORS 的实现分为**浏览器端自动处理**和**服务器端配置**两部分。开发者主要工作在服务器端配置响应头。

### 6.1 CORS 的两类请求

CORS 将跨源请求分为两类，处理流程不同：

#### 6.1.1 简单请求（Simple Requests）

满足**所有**以下条件的请求被视为简单请求，浏览器直接发送实际请求，不发预检：

1. **方法**仅限：`GET`、`HEAD`、`POST`
2. **请求头**仅限 CORS 安全列表（CORS-safelisted request headers）：
   - `Accept`
   - `Accept-Language`
   - `Content-Language`
   - `Content-Type`（有额外限制）
   - `Range`（仅简单范围值如 `bytes=256-`）
3. **`Content-Type`** 的值仅限：
   - `text/plain`
   - `multipart/form-data`
   - `application/x-www-form-urlencoded`
4. `XMLHttpRequest.upload` 上没有注册事件监听器
5. 请求中没有使用 `ReadableStream` 对象

**简单请求的流程：**

```
浏览器                              服务器
  |                                   |
  |---- GET /data (带 Origin 头) ---->|
  |                                   |
  |<---- 响应 (带 Access-Control-    |
  |       Allow-Origin 等头) --------|
  |                                   |
  |  浏览器校验响应头，决定是否       |
  |  将响应内容暴露给 JS 脚本         |
```

**请求示例：**

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
Origin: https://foo.example
```

**响应示例：**

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: application/json
```

浏览器检查 `Access-Control-Allow-Origin` 是否包含请求的 `Origin`，如果匹配，则允许脚本读取响应；否则在控制台报错，且脚本无法获取响应内容。

#### 6.1.2 预检请求（Preflighted Requests）

不满足简单请求条件的跨源请求（如使用 `PUT`/`DELETE` 方法、自定义请求头、`Content-Type: application/json` 等），浏览器会**先发送一个 `OPTIONS` 方法的预检请求**，询问服务器是否允许实际请求。服务器确认允许后，浏览器才发送实际请求。

**预检请求的流程：**

```
浏览器                              服务器
  |                                   |
  |---- OPTIONS /doc (预检) --------->|
  |      Access-Control-Request-      |
  |      Method: POST                 |
  |      Access-Control-Request-      |
  |      Headers: X-PINGOTHER,        |
  |      Content-Type                 |
  |      Origin: https://foo.example  |
  |                                   |
  |<---- 预检响应 --------------------|
  |       Access-Control-Allow-       |
  |       Origin: https://foo.example|
  |       Access-Control-Allow-       |
  |       Methods: POST, GET          |
  |       Access-Control-Allow-       |
  |       Headers: X-PINGOTHER,       |
  |       Content-Type                |
  |       Access-Control-Max-Age: 86400|
  |                                   |
  |---- POST /doc (实际请求) -------->|
  |      (带实际数据和头)             |
  |                                   |
  |<---- 实际响应 --------------------|
```

**预检请求（OPTIONS）示例：**

```
OPTIONS /doc HTTP/1.1
Host: bar.other
Origin: https://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

**预检响应示例：**

```
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
```

### 6.2 CORS 相关的 HTTP 头

#### 6.2.1 请求头（浏览器自动设置，开发者无需手动设置）

| 请求头 | 说明 |
| --- | --- |
| `Origin` | 表明请求的源站 URL（协议+域名+端口，不含路径）。**所有访问控制请求中总是被发送**。值可以为 `null`。 |
| `Access-Control-Request-Method` | 用于预检请求，告诉服务器实际请求将使用的 HTTP 方法。 |
| `Access-Control-Request-Headers` | 用于预检请求，告诉服务器实际请求将携带的自定义请求头字段。 |

#### 6.2.2 响应头（服务器端配置，是 CORS 的核心）

| 响应头 | 说明 | 示例 |
| --- | --- | --- |
| `Access-Control-Allow-Origin` | 指定允许访问该资源的源。可以是具体源或 `*`（通配符，仅无凭证请求可用）。 | `Access-Control-Allow-Origin: https://foo.example` |
| `Access-Control-Allow-Methods` | 用于预检响应，指定允许的 HTTP 方法列表。 | `Access-Control-Allow-Methods: POST, GET, OPTIONS` |
| `Access-Control-Allow-Headers` | 用于预检响应，指定允许的请求头字段列表。 | `Access-Control-Allow-Headers: X-PINGOTHER, Content-Type` |
| `Access-Control-Expose-Headers` | 指定浏览器允许 JS 脚本访问的响应头列表（默认只能访问 6 个基本头）。 | `Access-Control-Expose-Headers: X-My-Custom-Header` |
| `Access-Control-Max-Age` | 指定预检请求结果的缓存时间（秒），在此期间无需重复发预检。 | `Access-Control-Max-Age: 86400` |
| `Access-Control-Allow-Credentials` | 指定是否允许浏览器在跨源请求中携带凭证（Cookie、HTTP 认证、客户端 SSL 证书）。值只能为 `true`。 | `Access-Control-Allow-Credentials: true` |

### 6.3 附带身份凭证的请求（Credentials）

默认情况下，跨源请求**不携带** Cookie 等身份凭证。如果需要携带凭证，需要同时满足：

1. **前端**：`XMLHttpRequest` 设置 `withCredentials = true`，或 `fetch` 设置 `credentials: 'include'`
2. **服务器**：响应头设置 `Access-Control-Allow-Credentials: true`

**关键约束**：当请求附带凭证时，服务器**不能**将以下响应头设为通配符 `*`，必须指定具体值：

- `Access-Control-Allow-Origin` → 必须是具体源，不能是 `*`
- `Access-Control-Allow-Headers` → 必须是具体头列表
- `Access-Control-Allow-Methods` → 必须是具体方法列表
- `Access-Control-Expose-Headers` → 必须是具体头列表

此外，当服务器根据请求的 `Origin` 动态设置 `Access-Control-Allow-Origin` 时（而非使用 `*`），响应头中应包含 `Vary: Origin`，告诉客户端服务器对不同的 Origin 返回不同内容。

### 6.4 服务器端实现示例

#### Nginx 配置

```
location /api/ {
    # 简单请求
    add_header Access-Control-Allow-Origin "https://www.example.com" always;
    add_header Access-Control-Allow-Credentials "true" always;
    
    # 预检请求处理
    if ($request_method = 'OPTIONS') {
        add_header Access-Control-Allow-Origin "https://www.example.com" always;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
        add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
        add_header Access-Control-Max-Age "86400" always;
        add_header Access-Control-Allow-Credentials "true" always;
        return 204;
    }
    
    proxy_pass http://backend;
}
```

#### Node.js / Express

```
const express = require('express');
const cors = require('cors');
const app = express();

const corsOptions = {
  origin: 'https://www.example.com',  // 允许的源
  credentials: true,                    // 允许携带凭证
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Custom-Header'],
  maxAge: 86400
};

app.use(cors(corsOptions));
```

#### Java / Spring Boot

```
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("https://www.example.com")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("Content-Type", "Authorization")
            .exposedHeaders("X-Custom-Header")
            .allowCredentials(true)
            .maxAge(3600);
    }
}

```

### 6.5 CORS 完整校验逻辑（浏览器端）

当浏览器收到跨源响应时，执行以下校验：

```
1. 检查响应是否包含 Access-Control-Allow-Origin
   ├─ 不包含 → 拒绝，脚本无法读取响应
   └─ 包含 → 继续

2. 校验 Access-Control-Allow-Origin 的值
   ├─ 值为 "*" 且请求无凭证 → 通过
   ├─ 值为具体源且与请求 Origin 匹配 → 通过
   └─ 不匹配 → 拒绝

3. 如果请求有凭证（credentials: include）
   ├─ 检查 Access-Control-Allow-Credentials 是否为 true
   │   ├─ 不是 → 拒绝
   │   └─ 是 → 继续
   └─ 确认 Access-Control-Allow-Origin 不是 "*"（前面已校验）

4. 对于预检请求，还需校验：
   ├─ Access-Control-Allow-Methods 是否包含实际请求方法
   └─ Access-Control-Allow-Headers 是否包含实际请求的所有自定义头

5. 全部通过 → 浏览器将响应内容暴露给 JS 脚本
   任一失败 → 控制台报错，脚本无法读取响应（但请求可能已到达服务器）
```

---

## 面试总结

浏览器的同源策略(SOP),是一个浏览器的安全核心机制,原理浏览器控制在一个源中加载的文档或脚本与另一个源进行交互。目的是阻挡恶意文档操作和攻击。

源的组成由: 协议 + 域名 + 端口号 三者共同组成。 当三者有其中任意一部分不同，就存在跨域操作。

同源策略主要限制了：1. 使用 Ajax 技术和 Fetch API 在Web应用中请求同一个源中的HTTP资源。2. 不同源的页面之间无法通过JS互相访问DOM。
3. 无法通过Cookie/LocalStorage/IndexedDB等API,读取对方源中的数据。

但表单提交，页面跳转, 特定HTML标签的资源加载(`<img>、<script>、<link>、<iframe>、<video>`) 不受影响。

同源策略限制脚本读取响应内容，不阻止跨源请求发送。

跨源资源共享(CORS), 是一个种基于HTTP头通过允许服务器标识除它自己以外的其他源，让浏览器允许这些源访问自己域下的资源。

实现途径分为**浏览器端自动处理**和**服务器端配置**两部分。主要内容为在服务器端配置响应头。

浏览器处理的请求有两种简单请求和预检请求, 简单请求包含一般请求的方法名和请求头, 流程为浏览器中有JS脚本向服务器发送请求，服务器响应在响应头中添加Access-Control-Allow-Origin等响应头。浏览器校验响应头控制响应内容的暴露。

预检请求就是浏览器发送一个OPTION方法预检请求，询问服务器是否允许实际请求。服务器确认允许后，浏览器才发送实际请求。