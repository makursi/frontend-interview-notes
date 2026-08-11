# 前端核心理论知识

前端面试备考知识库（个人笔记），按主题分目录收录高频考点、易错陷阱、对比表格与手写实现题。全部为 Markdown 文档。

## 知识索引

### JavaScript 核心

| 文档 | 覆盖要点 |
|------|----------|
| `JavaScript核心/this指向.md` | 四条绑定规则、优先级、箭头函数、面试思考题 |
| `JavaScript核心/原型与原型链.md` | `__proto__` / `prototype` / `constructor`、查找流程、继承方式对比 |
| `JavaScript核心/作用域与闭包.md` | 作用域链、闭包、内存泄漏、防抖节流场景 |
| `JavaScript核心/事件循环.md` | 宏任务/微任务、执行顺序、async/await |
| `JavaScript核心/Promise与异步.md` | Promise 实现原理、手写、异步控制 |
| `JavaScript核心/ES6特性/知识点1~6.md` | ES6+ 语法特性逐点梳理 |

### 手写题

| 文档 | 主题 |
|------|------|
| `手写题/深拷贝.md` `深比较.md` | 深拷贝与深比较 |
| `手写题/防抖节流.md` `柯里化.md` | 函数工具 |
| `手写题/call-apply-bind.md` `new.md` `instanceof.md` | this 与原型相关实现 |
| `手写题/Promise - Promise.all.md` `发布订阅.md` `并发控制调度器.md` | 异步与事件机制 |
| `手写题/数组去重.md` `数组扁平化.md` `模板字符串解析.md` `JSON.stringify-parse.md` `LRU 缓存.md` | 高频工具实现 |
| `手写题/继承（寄生组合式）.md` | 原型继承 |
| `手写题/简单版虚拟 DOM Diff.md` `简单版 Webpack - Vite 原理口述.md` | 框架/工程化原理 |
| `手写题/快速排序.md` `图片懒加载.md` | 算法与浏览器能力 |

### CSS 基础

- `盒模型.md` — 标准/怪异盒模型、box-sizing
- `布局.md` — flex、grid、居中方案
- `定位与层叠.md` — position、层叠上下文、z-index
- `其他高频.md` — 高频 CSS 面试点汇总

### 浏览器原理

- `输入URL到页面渲染全过程.md` — 从输入 URL 到页面渲染完整流程
- `渲染原理.md` — 渲染管线、回流重绘、合成层
- `浏览器缓存.md` — 强缓存/协商缓存、缓存策略
- `浏览器安全.md` — XSS、CSRF 等安全防护

### 计算机网络

- `HTTP_HTTPS.md` — HTTP 各版本、HTTPS 握手、常见状态码
- `TCP_UDP.md` — 三次握手四次挥手、TCP/UDP 对比
- `跨域.md` — 同源策略、CORS、跨域解决方案

### 框架

- `Vue高频.md` — Vue 核心原理、响应式、diff、生命周期
- `React高频.md` — React 核心原理、hooks、fiber、diff

### 工程化

- `Webpack 核心概念.md` `Webpack 构建流程、热更新原理.md` — Webpack 全流程
- `Vite 原理与 Webpack 的区别.md` — 构建工具对比
- `Babel 原理.md` `模块化发展.md` `常见 loader 与 plugin 的区别及作用.md` — 编译与模块化
- `Git常用命令.md` `CI_CD基本概念.md` — 工程协作
- `代码规范.md` — （待完善）

### 算法与数据结构

- 基础：`数组.md` `字符串.md` `链表.md` `栈-队列.md`
- 进阶：`树.md` `排序.md` `动态规划.md`

### 性能优化

- `首屏优化.md` `渲染优化.md` `运行时优化.md` — 渲染与运行性能
- `网络层面.md` `缓存策略.md` — 网络与缓存优化
- `构建优化.md` `工具.md` — 构建与监控工具
- `监控指标.md` — 性能指标（FP/FCP/LCP 等）
