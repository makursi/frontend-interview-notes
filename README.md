# 2026年前端大厂面试理论知识汇总

前端大厂面试备考知识库（个人笔记），按主题分目录收录高频考点、易错陷阱、对比表格与手写实现题，全部为 Markdown 文档，面向面试答题。

## 目录导航

- [JavaScript核心](#javascript核心)
  - [ES6特性](#es6特性)
- [CSS基础](#css基础)
- [浏览器原理](#浏览器原理)
- [计算机网络](#计算机网络)
- [框架](#框架)
- [工程化](#工程化)
- [算法与数据结构](#算法与数据结构)
- [性能优化](#性能优化)
- [手写题](#手写题)

## JavaScript核心

| 知识点 | 文件 |
| --- | --- |
| 事件循环 | [事件循环.md](JavaScript核心/事件循环.md) |
| this 指向 | [this指向.md](JavaScript核心/this指向.md) |
| 原型与原型链 | [原型与原型链.md](JavaScript核心/原型与原型链.md) |
| 作用域与闭包 | [作用域与闭包.md](JavaScript核心/作用域与闭包.md) |
| Promise 与异步 | [Promise与异步.md](JavaScript核心/Promise与异步.md) |

### ES6特性

| 知识点 | 文件 |
| --- | --- |
| let/const 与 var 的区别（块级作用域、暂时性死区、不挂载到 window） | [知识点1.md](JavaScript核心/ES6特性/知识点1.md) |
| 箭头函数、解构赋值、模板字符串、扩展运算符 | [知识点2.md](JavaScript核心/ES6特性/知识点2.md) |
| Set / Map / WeakSet / WeakMap | [知识点3.md](JavaScript核心/ES6特性/知识点3.md) |
| Proxy 与 Object.defineProperty | [知识点4.md](JavaScript核心/ES6特性/知识点4.md) |
| Iterator、for...of 与 for...in | [知识点5.md](JavaScript核心/ES6特性/知识点5.md) |
| Class（类） | [知识点6.md](JavaScript核心/ES6特性/知识点6.md) |

## CSS基础

| 知识点 | 文件 |
| --- | --- |
| 布局 | [布局.md](CSS基础/布局.md) |
| 定位与层叠 | [定位与层叠.md](CSS基础/定位与层叠.md) |
| 盒模型 | [盒模型.md](CSS基础/盒模型.md) |
| 其他高频（选择器优先级、rem/em/vw/vh/% 等） | [其他高频.md](CSS基础/其他高频.md) |

## 浏览器原理

| 知识点 | 文件 |
| --- | --- |
| 输入 URL 到页面渲染全过程 | [输入URL到页面渲染全过程.md](浏览器原理/输入URL到页面渲染全过程.md) |
| 渲染原理 | [渲染原理.md](浏览器原理/渲染原理.md) |
| 浏览器缓存 | [浏览器缓存.md](浏览器原理/浏览器缓存.md) |
| 浏览器安全 | [浏览器安全.md](浏览器原理/浏览器安全.md) |

## 计算机网络

| 知识点 | 文件 |
| --- | --- |
| HTTP/HTTPS | [HTTP_HTTPS.md](计算机网络/HTTP_HTTPS.md) |
| TCP/UDP | [TCP_UDP.md](计算机网络/TCP_UDP.md) |
| 跨域 | [跨域.md](计算机网络/跨域.md) |

## 框架

| 知识点 | 文件 |
| --- | --- |
| Vue 高频 | [Vue高频.md](框架/Vue高频.md) |
| React 高频 | [React高频.md](框架/React高频.md) |

## 工程化

| 知识点 | 文件 |
| --- | --- |
| Webpack 核心概念 | [Webpack 核心概念.md](工程化/Webpack核心概念.md) |
| Webpack 构建流程、热更新原理 | [Webpack构建流程、热更新原理.md](工程化/Webpack构建流程、热更新原理.md) |
| Vite 原理与 Webpack 的区别 | [Vite原理与Webpack的区别.md](工程化/Vite原理与Webpack的区别.md) |
| 常见 loader 与 plugin 的区别及作用 | [常见 loader 与 plugin 的区别及作用.md](工程化/常见loader与plugin的区别及作用.md) |
| 模块化发展 | [模块化发展.md](工程化/模块化发展.md) |
| Babel 原理 | [Babel原理.md](工程化/Babel原理.md) |
| Git 常用命令 | [Git常用命令.md](工程化/Git常用命令.md) |
| CI/CD 基本概念 | [CI_CD基本概念.md](工程化/CI_CD基本概念.md) |

## 算法与数据结构

| 知识点 | 文件 |
| --- | --- |
| 数组 | [数组.md](算法与数据结构/数组.md) |
| 字符串 | [字符串.md](算法与数据结构/字符串.md) |
| 链表 | [链表.md](算法与数据结构/链表.md) |
| 栈-队列 | [栈-队列.md](算法与数据结构/栈-队列.md) |
| 树 | [树.md](算法与数据结构/树.md) |
| 排序 | [排序.md](算法与数据结构/排序.md) |
| 动态规划 | [动态规划.md](算法与数据结构/动态规划.md) |

## 性能优化

| 知识点 | 文件 |
| --- | --- |
| 首屏优化 | [首屏优化.md](性能优化/首屏优化.md) |
| 渲染优化 | [渲染优化.md](性能优化/渲染优化.md) |
| 构建优化 | [构建优化.md](性能优化/构建优化.md) |
| 运行时优化 | [运行时优化.md](性能优化/运行时优化.md) |
| 网络层面 | [网络层面.md](性能优化/网络层面.md) |
| 缓存策略 | [缓存策略.md](性能优化/缓存策略.md) |
| 监控指标 | [监控指标.md](性能优化/监控指标.md) |
| 工具 | [工具.md](性能优化/工具.md) |

## 手写题

| 序号 | 知识点 | 文件 |
| --- | --- | --- |
| 1 | 深拷贝 | [深拷贝.md](手写题/深拷贝.md) |
| 2 | 防抖节流 | [防抖节流.md](手写题/防抖节流.md) |
| 3 | Promise / Promise.all | [Promise-Promise.all.md](手写题/Promise-Promise.all.md) |
| 4 | call/apply/bind | [call-apply-bind.md](手写题/call-apply-bind.md) |
| 5 | new | [new.md](手写题/new.md) |
| 6 | instanceof | [instanceof.md](手写题/instanceof.md) |
| 7 | 数组扁平化 | [数组扁平化.md](手写题/数组扁平化.md) |
| 8 | 数组去重 | [数组去重.md](手写题/数组去重.md) |
| 9 | 柯里化 | [柯里化.md](手写题/柯里化.md) |
| 10 | 发布订阅 | [发布订阅.md](手写题/发布订阅.md) |
| 11 | 深比较 | [深比较.md](手写题/深比较.md) |
| 12 | LRU 缓存 | [LRU缓存.md](手写题/LRU缓存.md) |
| 13 | 并发控制调度器 | [并发控制调度器.md](手写题/并发控制调度器.md) |
| 14 | 继承（寄生组合式） | [继承（寄生组合式）.md](手写题/继承（寄生组合式）.md) |
| 15 | 快速排序 | [快速排序.md](手写题/快速排序.md) |
| 16 | 简单版虚拟 DOM Diff | [简单版虚拟DOMDiff.md](手写题/简单版虚拟DOMDiff.md) |
| 17 | JSON.stringify/parse | [JSON.stringify-parse.md](手写题/JSON.stringify-parse.md) |
| 18 | 模板字符串解析 | [模板字符串解析.md](手写题/模板字符串解析.md) |
| 19 | 图片懒加载 | [图片懒加载.md](手写题/图片懒加载.md) |
| 20 | 简单版 Webpack / Vite 原理口述 | [简单版Webpack-Vite原理口述.md](手写题/简单版Webpack-Vite原理口述.md) |
