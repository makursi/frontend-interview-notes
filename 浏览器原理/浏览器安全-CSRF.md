---

# 浏览器安全：CSRF（跨站请求伪造）详解

## 一、CSRF 是什么

**CSRF（Cross-Site Request Forgery，跨站请求伪造）** 是一种攻击手段：攻击者诱导已登录的用户，在其不知情的情况下，向目标网站发送一个伪造的 HTTP 请求，从而以用户的身份执行非本意的操作（如转账、改密码、发帖子等）。

它的核心特点是：**攻击者不需要拿到用户的 Cookie，只需要用户"已经登录"即可**——利用浏览器"自动携带 Cookie"这一机制来冒充用户操作。

---

## 二、CSRF 攻击原理

### 2.1 攻击成立的三个前提

一个网站存在 CSRF 风险，需要同时满足以下条件：

1. **使用 HTTP 请求来改变服务器状态**（如转账、修改资料等写操作）。
2. **仅依赖 Cookie 来验证用户身份**（请求中只带 Session Cookie，没有其他校验）。
3. **请求参数是攻击者可预测的**（比如转账接口只需要 `recipient` 和 `amount` 两个参数，攻击者可以构造）。

### 2.2 攻击流程

1. 用户浏览并登录信任网站 A（如银行）。
2. 网站 A 验证通过，在用户浏览器中写入 A 的 Cookie（含 Session ID）。
3. 用户在**未登出** A 的情况下，访问了攻击者网站 B。
4. 网站 B 要求浏览器向第三方站点 A 发出一个请求（如自动提交表单）。
5. 浏览器根据 B 的请求，**自动带上**用户在 A 中产生的 Cookie，访问 A。
6. 网站 A 无法区分这个请求是用户本人发出的还是 B 伪造的，于是按用户权限处理请求——攻击完成。



### 2.3 常见攻击方式

**方式一：自动提交表单（POST）**

攻击者构造一个隐藏表单，页面加载时用 JS 自动提交：

```
<form action="https://my-bank.example.org/transfer" method="POST" id="evilForm">
  <input type="hidden" name="recipient" value="attacker">
  <input type="hidden" name="amount" value="1000">
</form>
<script>document.getElementById('evilForm').submit();</script>
```

用户访问该页面时，浏览器会带着银行的 Cookie 自动发出 POST 请求。

**方式二：图片标签触发 GET 请求**

如果目标网站用 GET 请求执行状态变更，攻击者甚至不需要表单：

```
<img src="https://my-bank.example.org/transfer?recipient=attacker&amount=1000">
```

浏览器加载图片时就会自动发出这个 GET 请求。因此 **不要用 GET 请求做状态变更操作**。

---



## 三、CSRF 防御手段



### 3.1 CSRF Token（同步令牌模式）



#### 原理

服务器在渲染页面时，嵌入一个**随机、不可预测**的值（CSRF Token）。当合法页面发起状态变更请求时，必须带上这个 Token；服务器校验 Token 与会话中存储的是否一致，不一致则拒绝。攻击者无法获取到这个 Token，因此无法构造有效请求。

#### 实现步骤

1. 用户登录后，服务器为当前会话生成一个随机 Token，存入 Session（或 Redis）。
2. 渲染页面时，将 Token 嵌入表单隐藏域或页面 meta 标签中。
3. 用户提交表单或发 AJAX 请求时，将 Token 作为参数或自定义请求头一并提交。
4. 服务器收到请求后，比对提交的 Token 与 Session 中的是否一致。

```
<!-- 表单隐藏域 -->
<form action="/transfer" method="post">
  <input type="hidden" name="CSRFToken" value="OWY4NmQwODE4ODRjN2Q2NTlh...">
  <input name="recipient">
  <input name="amount">
</form>
```

```
// AJAX 请求中通过自定义头携带
fetch('/transfer', {
  method: 'POST',
  headers: { 'X-CSRF-Token': token, 'Content-Type': 'application/json' },
  body: JSON.stringify({ recipient: 'joe', amount: 10 })
});
```



#### Token 的关键要求

CSRF Token 必须满足：

- **每个用户会话唯一**（不能全局通用）
- **保密、不可预测**（用密码学安全的随机数生成）
- **不要放在 URL 中**（会泄露到浏览器历史、Referer、日志）
- **不要放在 Cookie 中传输**（同步令牌模式下）
- 通过自定义请求头传输比表单隐藏域更安全（因为自定义头受同源策略保护）



#### 扩展：双重提交 Cookie

如果服务器不想维护 Session（无状态架构），可以用双重提交 Cookie 模式：服务器把 Token 写入一个**非 HttpOnly 的 Cookie**，前端 JS 读取该 Cookie 并把 Token 同时放到请求参数/头中。服务器校验"Cookie 中的 Token"和"请求体/头中的 Token"是否一致。由于攻击者无法读取跨域 Cookie，也就无法在请求中放入正确的 Token。

> OWASP 推荐使用**签名版双重提交 Cookie（Signed Double-Submit Cookie）**，将 Token 与会话 ID 通过 HMAC 绑定，防止 Cookie 注入攻击。

---



### 3.2 SameSite Cookie 属性 —— 浏览器层面的纵深防御



#### 原理

`SameSite` 是 Cookie 的一个属性（类似 `HttpOnly`、`Secure`），由 RFC6265bis 定义，用于控制浏览器在**跨站请求**中是否携带该 Cookie。从源头减少 CSRF 攻击的可能性。

#### 三个取值


| 值        | 行为                                                                   | 适用场景                                      |
| -------- | -------------------------------------------------------------------- | ----------------------------------------- |
| `Strict` | 任何跨站请求都**不携带** Cookie，即使是点击普通链接跳转也不携带                                | 对安全性要求极高的操作（如转账），但会影响用户体验——从外部链接点进来会显示未登录 |
| `Lax`    | 跨站请求中，仅在**顶级导航 + 安全方法（GET）**时携带 Cookie；POST、AJAX、iframe、img 等跨站请求不携带 | 大多数网站的会话 Cookie，兼顾安全与体验。现代浏览器的**默认值**     |
| `None`   | 任何跨站请求都携带 Cookie（必须同时设置 `Secure`，即仅 HTTPS）                           | 需要跨站携带 Cookie 的场景（如第三方嵌入、OAuth）           |




#### `Lax` 的细节与局限

`Lax` 比 `Strict` 保护弱，原因在于：

- 攻击者可以触发**顶级导航**（如表单自动提交本身就是顶级导航），如果表单用 GET 提交，`Lax` 下仍会携带 Cookie。
- 一些框架支持"方法覆盖"（method override），攻击者可以用 GET 发送但服务器按 POST 处理。

建议：**用于判断"是否显示登录态页面"的 Cookie 用** `Lax`**，用于状态变更请求的 Cookie 尽量用** `Strict`。

#### SameSite 的局限性

1. **它防的是"跨站"而非"跨源"**：`https://foo.example.org` 和 `https://bar.example.org` 被视为同一站点（same site），但属于不同源（different origin）。如果某个子域名被攻破，SameSite 无法防护。
2. **老浏览器不支持**（如旧版 IE），存在兼容问题。
3. **不能单独作为唯一防御**：OWASP 明确指出，除非满足特定条件（仅面向现代浏览器、状态变更端点有 Origin/Referer 校验等），否则 SameSite 应作为**纵深防御的一层**，与 Token 等方案配合使用。

---



### 3.3 Referer / Origin 校验 —— 低成本辅助手段



#### 原理

服务器检查 HTTP 请求头中的 `Referer`（或 `Origin`），判断请求是否来自本站域名。如果来源不可信，则拒绝请求。

```
// Express 示例
app.use((req, res, next) => {
  const referer = req.get('Referer');
  if (!referer || !referer.startsWith('https://my-site.com')) {
    return res.status(403).end();
  }
  next();
});
```



#### Origin 头优先于 Referer

建议的校验策略：

1. **如果** `Origin` **头存在**，优先校验 `Origin` 是否匹配目标源。`Origin` 头在 HTTPS 请求中更可靠，且不会像 Referer 那样包含路径等敏感信息。
2. **如果** `Origin` **头不存在**，再回退校验 `Referer` 头中的主机名是否匹配。



#### 优缺点

**优点：**

- 实现简单，几乎零改造成本。
- 对未认证请求（如登录前的操作）也能起到防护作用。

**缺点：**

- **Referer 可能被浏览器省略**：某些浏览器配置、隐私模式、HTTPS→HTTP 跳转等场景下不发送 Referer。
- **Referer 可被伪造**：在某些情况下（如配合子域名 XSS）可以篡改。
- **不能单独使用**：通常作为 Token / SameSite 之外的辅助层。

---



## 四、防御方案对比与最佳实践


| 防御方案                  | 可靠性 | 改造成本 | 能否单独使用 | 备注                |
| --------------------- | --- | ---- | ------ | ----------------- |
| **CSRF Token**        | 高   | 中    | 可以     | 最主流、最稳妥，主流框架均内置支持 |
| **SameSite Cookie**   | 中   | 低    | 不建议单独用 | 浏览器层面免费防护，作为纵深防御  |
| **Referer/Origin 校验** | 低-中 | 极低   | 不能     | 辅助手段，Referer 可能缺失 |


**推荐的最佳实践：**

1. 优先检查框架是否有**内置 CSRF 防护**（如 Django 的 `csrf_token`、Angular 的双重提交 Cookie），直接使用。
2. 有状态应用使用**同步令牌模式**，无状态应用使用**签名版双重提交 Cookie**。
3. 会话 Cookie 设置 `SameSite=Strict`（或至少 `Lax`）。
4. 对状态变更端点增加 **Origin/Referer 校验**作为纵深防御。
5. **不要用 GET 请求做状态变更**。
6. 高危操作（转账、改密码）增加**用户交互验证**（短信验证码、二次确认）。
7. **注意：XSS 可以绕过所有 CSRF 防护**——如果网站存在 XSS 漏洞，攻击者可以直接读取页面中的 Token 并发起请求，因此 XSS 防护同样重要。



## 面试总结

CSRF全称为跨站请求伪造。本质为:攻击者诱导已登录用户，在未知情情况下向目标网站发送伪造的HTTP请求,以用户的身份对用户已登录的网站执行非用户本意的操作。

### 攻击前提: 
1. 使用HTTP请求改变**服务器状态** 2. 依赖Cookie验证身份，网站本地保留用户的信息。3. 目标网站的请求参数可以被攻击者预测。

### 攻击流程:

1.首先用户浏览并登陆信任网站A
2.网站A验证用户通过，并在用户的浏览器本地写入A的Cookie
3.用户在未登出网站A(即网站A保留用户的个人信息)的情况下,访问了攻击者提供的网站B。
4.网站B要求浏览器向第三方网站A发送请求。
5.浏览器根据B请求，自动带上用户在网站A中产生的Cookie，访问网站A。
6. 网站A无法区分该请求是否为用户本人发出的还是B进行伪造的，依旧对请求进行处理。
7. 攻击完成

### 防御手段

1. 使用CSRF Token，原理：为合法页面发生状态变更时，提供一个随机，不可预测的值。
目标网站服务器会校验该Token与会话存储是否一致，不一致就拒绝。攻击者无法获取到该Token,所以无法构造有效请求。

工作流程: 
    1.用户登录后，服务器为当前会话生成CSRF Token,然后存入Session中。
    2.渲染页面时，将Token嵌入表单隐藏域或页面meta标签中。
    3.用户提交表单或发AJAX请求时，将已经生成的Token作为参数或自定义请求头一并提交。
    4.服务器收到请求，对比提交的token和session中的是否一致。

2. 在后端响应头中添加设置SameSite Cookie属性。原理为: 在后端对前端请求的响应报文中，设置响应头SameSite属性为: Strict 或 Lax属性，以控制浏览器在跨站请求中是否携带该Cookie,从源头减少CSRF攻击可能性。

lax 和 Strict的区别与应用场景: Strict比Lax更加严格，适用于对安全性要求极高的操作，会严重影响用户体验，当用户从外部连接进入时会失去登陆状态，被网站要求重新登陆。

lax宽松，是大多数网站的会话Cookie，现代浏览器中的默认值。