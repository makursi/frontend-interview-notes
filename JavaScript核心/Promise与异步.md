# Promise与异步

Promise对象是一个表示“异步操作最终完成或失败”的对象及其结果值。

Promise对象上的构造函数参数有:  

- `executor`  作用是作为执行器函数立即执行 
- `resolve` 作用是标记Promise对象为"成功",传入成功值（任意值：字符串，对象，Promise）  
- `reject` 作用是标记Promise对象为"失败",传入失败原因(Error对象或字符串);

## 创建Promise对象

```javascript
const myPromise = new Promise((resolve, reject)=> {
  const ifSuccess = true;
  if (ifSuccess) {
    resolve("操作成功");
   } else {
    reject("操作失败");
   }
})
```

调用构造函数，传入一个executor执行器函数，函数自动接收
`relove` 函数和`rejcet` 函数。并分别再向reslove函数传入`操作成功` 和 `操作失败` 两个值。

## Promise对象的三种状态

通过上面我们像函数参数进行传值, myPromise对象会有三种不同的状态值分别为: 

- Pending(进行中)，初始状态; 
- fulfilled(已成功),操作成功完成；
- rejected(已失败)，操作失败；这两种状态值都不可再变。



## Promise对象的实例方法

Promise的核心实例方法有`then()`  , `catch()`  , `finally()` 这三个方法

### 1. then()

`then()` 用于处理 Promise 成功（fulfilled）后的结果，也可以通过第二个参数处理失败（rejected），不过实际开发中一般不推荐使用第二个参数，而是统一交给 `catch()` 处理。

语法：

```javascript
promise.then(onFulfilled, onRejected)
```

`then()` 最大的特点是**始终返回一个新的 Promise**，因此支持链式调用。

它的返回值有以下几种情况：

- 返回普通值，会自动包装成 `Promise.resolve(value)`。
- 返回 Promise，会等待这个 Promise 完成后再继续执行后面的链。
- 抛出异常，会返回一个 rejected 状态的 Promise，并交给后续的 `catch()` 处理。

---



### 2. catch()

`catch()` 用于统一处理 Promise 链中的异常。

`catch(onRejected)` 等价于 `then(undefined, onRejected)`，它的返回值也为一个新的Promise

语法：

```javascript
promise.catch(onRejected)
```

它本质上等价于：

```javascript
promise.then(null, onRejected)
```

`catch()` 可以捕获：

- Promise 的 `reject`
- `then()` 回调中抛出的异常
- Promise 链中前面所有未处理的错误

因此实际开发中通常只在链的最后写一个 `catch()` 来统一进行错误处理。

---



## 链式调用

链式调用的本质是每次Promise对象调用 `.then()` 都返回新 Promise，因此可以无限链式调用。

链式调用中的错误传播中, 链中任何一个环节出错，都会**跳过后续所有** `then`，直到遇到最近的 `catch`：

```js
Promise.resolve()
  .then(() => { throw new Error('第一步出错'); })
  .then(() => console.log('不会执行'))      // 被跳过
  .then(() => console.log('也不会执行'))    // 被跳过
  .catch(e => console.log('捕获:', e.message)) // 捕获: 第一步出错
  .then(() => console.log('恢复后继续'));    // 恢复后继续
```



## 值穿透问题



### 值穿透

值穿透是当 `.then()` 或 `.catch()` 接收的参数**不是函数**时，Promise 会将其忽略，并把**上一个 Promise 的值原样传递**给下一个 `.then()`。

```js
// then 的参数不是函数，发生值穿透
Promise.resolve(100)
  .then(null)          // null 不是函数，被忽略
  .then(undefined)     // undefined 不是函数，被忽略
  .then('hello')       // 字符串不是函数，被忽略
  .then(v => console.log(v)); // 100 —— 值穿透了！
```



### 值穿透的规范依据

根据 Promise/A+ 规范，`then` 的两个参数如果不是函数，就会被忽略，Promise 会采用 "默认回调"：

- 成功默认回调：`value => value`（原样返回）
- 失败默认回调：`reason => { throw reason }`（原样抛出）



## async/await

### 1.1 ECMAScript 规范定义

`async/await` 是 **ES2017（ES8）** 正式纳入标准的异步编程语法，由两个关键字组成：

- `async`：用于声明一个异步函数（Async Function），函数体内允许使用 `await`
- `await`：用于等待一个 Promise 的结果，只能在 `async` 函数内部使用

> **规范原文要点**：
>
> - Async Function 是函数对象的一种，其 `[[AsyncFunction]]` 内部槽为 `true`
> - `async` 函数的返回值**永远是一个 Promise**
> - `await` 表达式会暂停当前 async 函数的执行，等待 Promise 结算（settle）后恢复执行
> - 暂停期间不阻塞事件循环，引擎可以处理其他任务



### 1.2 语法定义

```javascript
// async 函数声明
async function name([param[, param[, ...param]]]) {
  // 可以使用 await
}

// async 函数表达式
const name = async function([...params]) { ... };

// async 箭头函数
const name = async ([...params]) => { ... };

// async 方法（对象/类）
const obj = {
  async method() { ... }
};
class Foo {
  async method() { ... }
}

// await 表达式
const result = await thenable; // 等待 Promise 或 thenable
```

---



## 二、核心作用



### 2.1 解决的问题


| 异步方案            | 问题                                             |
| --------------- | ---------------------------------------------- |
| 回调函数（Callback）  | 回调地狱、错误处理混乱、难以串行/并行控制                          |
| Promise         | 解决了回调地狱，但大量 `.then()` 链式仍然不够直观，错误处理分散          |
| **async/await** | 用**同步写法写异步逻辑**，代码可读性接近同步代码，错误处理统一用 `try/catch` |




### 2.2 核心价值

1. **语义化**：`await` 字面意思就是"等待"，代码读起来就是同步的执行顺序
2. **同步思维**：可以用普通变量接收异步结果，用 `try/catch` 捕获错误
3. **调试友好**：断点可以停在 `await` 之后，像调试同步代码一样
4. **与 Promise 完全兼容**：`async` 函数返回 Promise，可以和 `.then()`、`Promise.all()` 等自由组合



### 2.3 本质

`async/await` 是 **Generator + 自动执行器（co）** 的语法糖，底层仍然基于 Promise。编译器/引擎会将 async 函数转写成状态机，在每个 `await` 处暂停和恢复。

---



## 三、基本使用方法



### 3.1 async 函数的返回值

```javascript
// 1. 返回普通值 → 自动包装为 fulfilled Promise
async function foo() {
  return 42;
}
foo().then(v => console.log(v)); // 42

// 2. 返回 Promise → 直接返回该 Promise
async function bar() {
  return Promise.resolve('hello');
}
bar().then(v => console.log(v)); // "hello"

// 3. 抛出错误 → 返回 rejected Promise
async function baz() {
  throw new Error('出错了');
}
baz().catch(e => console.log(e.message)); // "出错了"

// 4. 没有 return → 返回 fulfilled Promise，值为 undefined
async function qux() {
  console.log('执行了');
}
qux().then(v => console.log(v)); // undefined
```



### 3.2 await 的基本用法

```javascript
// 模拟异步请求
function fetchUser() {
  return new Promise(resolve => {
    setTimeout(() => resolve({ id: 1, name: '张三' }), 1000);
  });
}

function fetchOrders(userId) {
  return new Promise(resolve => {
    setTimeout(() => resolve([{ id: 101, amount: 99 }]), 500);
  });
}

// 串行执行（前一个完成才执行下一个）
async function getUserData() {
  const user = await fetchUser();           // 等待1秒
  const orders = await fetchOrders(user.id); // 再等待0.5秒
  return { user, orders };
}

getUserData().then(data => console.log(data));
// 总耗时约 1.5 秒
```



### 3.3 错误处理：try/catch

```javascript
async function getData() {
  try {
    const user = await fetchUser();
    const orders = await fetchOrders(user.id);
    return { user, orders };
  } catch (error) {
    // 捕获 await 过程中任何一个 rejected Promise
    console.error('请求失败:', error.message);
    return null; // 错误恢复
  }
}
```

> **关键点**：`try/catch` 可以捕获 `await` 表达式抛出的错误，就像捕获同步错误一样。这是 async/await 相比 Promise 链式 `.catch()` 的一大优势。



### 3.4 并行执行

```javascript
// ❌ 错误写法：串行，总耗时 1.5 秒
async function serial() {
  const a = await fetchA(); // 1秒
  const b = await fetchB(); // 0.5秒
  return [a, b];
}

// ✅ 正确写法：并行，总耗时约 1 秒（取最长的那个）
async function parallel() {
  const promiseA = fetchA(); // 立即发起请求
  const promiseB = fetchB(); // 立即发起请求
  const a = await promiseA;
  const b = await promiseB;
  return [a, b];
}

// ✅ 更简洁的写法：Promise.all
async function parallelAll() {
  const [a, b] = await Promise.all([fetchA(), fetchB()]);
  return [a, b];
}
```



### 3.5 常见组合用法

```javascript
// 与 Promise.all / Promise.race / Promise.allSettled 配合
async function demo() {
  // 全部成功才继续，否则抛出第一个错误
  const [r1, r2, r3] = await Promise.all([task1(), task2(), task3()]);

  // 谁先完成就用谁的结果
  const fastest = await Promise.race([task1(), task2()]);

  // 等待全部完成，不管成功失败
  const results = await Promise.allSettled([task1(), task2()]);
}
```

---



## 四、注意事项与实现细节



### 4.1 await 只能在 async 函数内使用

```javascript
// ❌ 语法错误：await 不能在普通函数中使用
function bad() {
  await fetchUser(); // SyntaxError: await is only valid in async functions
}

// ❌ 注意：回调函数也需要 async
async function processItems(items) {
  items.forEach(item => {
    await save(item); // ❌ 报错！forEach 的回调不是 async 函数
  });
}

// ✅ 解决：用 for...of（串行）
async function processItems(items) {
  for (const item of items) {
    await save(item);
  }
}

// ✅ 解决：用 Promise.all（并行）
async function processItems(items) {
  await Promise.all(items.map(item => save(item)));
}
```



### 4.2 await 不阻塞事件循环

```javascript
async function demo() {
  console.log('1');
  await Promise.resolve();
  console.log('2'); // 这行进入微任务队列
}

console.log('0');
demo();
console.log('3');

// 输出顺序: 0 → 1 → 3 → 2
```

**原理**：`await` 暂停的是**当前 async 函数**，不是整个线程。引擎会继续执行调用栈中的其他同步代码，`await` 之后的代码被包装成微任务，等当前同步代码执行完后再恢复。

### 4.3 await 会自动调用 thenable 的 then

```javascript
// await 不仅可以等待 Promise，还可以等待任何 thenable 对象
const thenable = {
  then(resolve, reject) {
    setTimeout(() => resolve('thenable 结果'), 500);
  }
};

async function test() {
  const result = await thenable;
  console.log(result); // "thenable 结果"
}
```



### 4.4 await 非 Promise 值会立即返回

```javascript
async function test() {
  const a = await 42;        // 等价于 await Promise.resolve(42)
  const b = await 'hello';   // 直接返回
  const c = await { x: 1 };  // 直接返回
  console.log(a, b, c); // 42 hello {x: 1}
}
```

> 虽然非 Promise 值会立即 resolve，但 `await` 之后的代码仍然会被放到微任务队列中执行，不会同步执行。



### 4.5 async 函数中的 this 指向

```javascript
const obj = {
  value: 10,
  async getValue() {
    return this.value; // this 指向 obj，和普通方法一致
  }
};
obj.getValue().then(v => console.log(v)); // 10

// 注意：如果解构赋值会丢失 this
const { getValue } = obj;
getValue(); // this 为 undefined（严格模式），返回 rejected Promise
```



### 4.6 常见陷阱



#### 陷阱1：忘记 await

```javascript
async function bad() {
  const user = fetchUser(); // 忘记 await，user 是 Promise 对象
  console.log(user.name);   // undefined
}
```



#### 陷阱2：for 循环中的并行 vs 串行

```javascript
// 串行执行（一个接一个）
async function serial(items) {
  for (const item of items) {
    await process(item);
  }
}

// 并行执行（同时发起）
async function parallel(items) {
  await Promise.all(items.map(item => process(item)));
}
```



#### 陷阱3：async 构造函数不存在

```javascript
// ❌ 构造函数不能是 async
class Foo {
  async constructor() { ... } // SyntaxError
}

// ✅ 解决方案：静态工厂方法
class Foo {
  constructor(data) {
    this.data = data;
  }
  static async create() {
    const data = await fetchData();
    return new Foo(data);
  }
}
const foo = await Foo.create();
```



#### 陷阱4：顶层 await（ES2022）

```javascript
// 以前：顶层不能直接用 await
// ❌ 报错
const data = await fetchData();

// ES2022 支持顶层 await（仅在 ES Module 中）
// ✅ 模块顶层可以使用
const data = await fetchData();
```

---



## 五、面试题总结



### 高频考点1：async/await 原理

**题目**：请简述 async/await 的实现原理。

**参考答案**：

> async/await 是 Generator + 自动执行器的语法糖。`async` 函数在编译后会被转成一个状态机，每个 `await` 对应一个状态断点。引擎通过自动执行器递归调用 Generator 的 `next()` 方法：遇到 `await` 的 Promise 就等待它 resolve，然后把结果通过 `next(value)` 传回，继续执行下一段代码；如果 Promise reject，则通过 `throw(error)` 将错误抛回 Generator，可被 `try/catch` 捕获。`async` 函数整体返回一个 Promise。

---



### 高频考点2：执行顺序（事件循环）

**题目**：写出以下代码的输出顺序：

```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

async1();

new Promise(resolve => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
});

console.log('script end');
```

**参考答案**：

```
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

**解析**：

1. 同步代码先执行：`script start` → 调用 `async1()` → `async1 start` → 调用 `async2()` → `async2`
2. `await async2()` 暂停 async1，`async2()` 返回的 Promise 立即 resolve，`async1 end` 进入微任务队列
3. 继续同步：`promise1` → `script end`
4. 微任务队列按顺序执行：`async1 end` → `promise2`
5. 最后宏任务：`setTimeout`

---



### 高频考点3：串行与并行

**题目**：以下两种写法有什么区别？

```javascript
// 写法A
async function A() {
  const r1 = await fetch1();
  const r2 = await fetch2();
}

// 写法B
async function B() {
  const p1 = fetch1();
  const p2 = fetch2();
  const r1 = await p1;
  const r2 = await p2;
}
```

**参考答案**：

> - **写法A是串行**：`fetch1()` 完成后才发起 `fetch2()`，总耗时 = t1 + t2
> - **写法B是并行**：`fetch1()` 和 `fetch2()` 几乎同时发起，总耗时 ≈ max(t1, t2)
> - 写法B等价于 `await Promise.all([fetch1(), fetch2()])`
> - 选择哪种取决于两个请求是否有依赖关系：有依赖用串行，无依赖用并行以提升性能

---



### 高频考点4：错误处理

**题目**：async 函数中如何处理错误？如果 await 的 Promise reject 了但没有 catch 会怎样？

**参考答案**：

> 1. **try/catch**：最常用的方式，可以捕获 await 表达式的错误
> 2. **Promise.catch()**：`await fetch().catch(e => handle(e))`，在 await 时直接处理
> 3. **返回值检查**：用 `Promise.allSettled()` 区分成功和失败
>
> 如果 await 的 Promise reject 且没有任何 try/catch 或 .catch()，async 函数会返回一个 rejected Promise。如果这个返回的 Promise 也没有被 catch，会触发 `unhandledRejection` 事件（Node.js 中可能导致进程崩溃）。

```javascript
// 推荐：统一错误处理
async function safeFetch(url) {
  try {
    const res = await fetch(url);
    return await res.json();
  } catch (e) {
    console.error('请求失败:', e);
    return null;
  }
}
```

---



### 高频考点5：async 函数返回值

**题目**：以下代码输出什么？

```javascript
async function test() {
  return await Promise.resolve(1);
}
console.log(test() instanceof Promise);
test().then(v => console.log(v));
```

**参考答案**：

> 输出 `true`，然后输出 `1`。
>
> async 函数**永远返回 Promise**，即使内部 `return` 的是普通值或 await 的结果，也会被自动包装成 Promise。

---



### 高频考点6：forEach 中使用 await

**题目**：以下代码有什么问题？如何修复？

```javascript
async function process(arr) {
  arr.forEach(async item => {
    const result = await save(item);
    console.log(result);
  });
  console.log('全部完成');
}
```

**参考答案**：

> **问题**：
>
> 1. `forEach` 的回调是 async 函数，但 `forEach` 不会等待这些异步回调完成
> 2. `console.log('全部完成')` 会在所有 `save()` 完成之前执行
> 3. 回调中的错误无法被外层 try/catch 捕获
>
> **修复方案**：
>
> ```javascript
> // 串行
> async function processSerial(arr) {
>   for (const item of arr) {
>     const result = await save(item);
>     console.log(result);
>   }
>   console.log('全部完成');
> }
>
> // 并行
> async function processParallel(arr) {
>   await Promise.all(arr.map(async item => {
>     const result = await save(item);
>     console.log(result);
>   }));
>   console.log('全部完成');
> }
> ```

---



### 高频考点7：async/await vs Promise

**题目**：async/await 相比 Promise 有什么优缺点？

**参考答案**：


| 维度   | Promise            | async/await        |
| ---- | ------------------ | ------------------ |
| 可读性  | 链式调用，较多 `.then()`  | 同步写法，可读性更高         |
| 错误处理 | `.catch()` 分散在链中   | `try/catch` 统一处理   |
| 调试   | 断点在回调中，调用栈不直观      | 像同步代码一样调试          |
| 并行控制 | `Promise.all` 原生支持 | 需要配合 `Promise.all` |
| 条件分支 | 链式中较难写条件           | 普通 if/else 即可      |
| 适用场景 | 简单的异步链、并发控制        | 复杂的异步流程逻辑          |


> **结论**：async/await 不是替代 Promise，而是建立在 Promise 之上的语法糖。两者可以混用，实际开发中推荐以 async/await 为主，配合 `Promise.all/race/allSettled` 处理并发。

---



### 高频考点8：实现一个 sleep 函数

**题目**：用 async/await 实现一个 sleep 函数。

**参考答案**：

```javascript
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function demo() {
  console.log('开始');
  await sleep(1000);
  console.log('1秒后');
  await sleep(2000);
  console.log('再2秒后');
}
```

---



### 高频考点9：并发控制

**题目**：如何用 async/await 实现限制并发数的请求？

**参考答案**：

```javascript
async function pool(tasks, limit) {
  const results = [];
  const executing = [];

  for (const task of tasks) {
    const p = task().then(result => {
      executing.splice(executing.indexOf(p), 1);
      return result;
    });
    results.push(p);
    executing.push(p);

    if (executing.length >= limit) {
      await Promise.race(executing); // 等待任意一个完成
    }
  }

  return Promise.all(results);
}
```

---



## 面试总结

### 什么是Promise?
Promise对象是一个表示“异步操作最终完成或失败”的对象及其结果值。Promise对象上的构造函数参数有: `executor` 执行器函数立即执行,`resolve` 标记Promise对象为"成功",传入成功值, `reject` 作用是标记Promise对象为"失败",传入失败原因。

### Promise的三种状态值是什么?
Promise有三种状态值分别为：Pending(进行中)初始状态, fulfilled(已成功),操作成功完成, rejected(已失败),操作失败, 其中fulfilled和rejected这两种状态值都不可再变。

### .then()方法的链式调用
Promise的核心实例方法有`then()`  , `catch()`,它们的返回值**永远返回一个新的 Promise 对象**，链式调用的本质就是每次调用.then()返回新的Promise从而进行无限调用。

### 解释一下值穿透问题
值穿透问题为当 `.then()` 或 `.catch()` 接收的参数**不是函数**时，Promise 会将其忽略，并把**上一个 Promise 的值原样传递**给下一个 `.then()`。

### async/await是什么?

async/await是js官方标准的异步编程语法糖，有async, await两个关键字组成。async用于声明异步函数，函数体内允许使用await。await用于等待一个Promise的结果,只能在async函数内部使用

async/await的核心作用为用**同步写法写异步逻辑**，代码可读性接近同步代码，错误处理统一用 `try/catch`，提升代码阅读体验。

在函数名称前加入async标记该函数为异步函数，并在函数体内使用await定义异步操作。