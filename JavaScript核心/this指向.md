# JavaScript this 指向完整详解

> 核心结论：this 不是在定义时确定，绝大多数情况是【调用时】确定；只有箭头函数例外，this 在定义时继承外层作用域 this。

## 一、全局 this

### 1. 浏览器全局作用域

全局顶层 this === window

```js
console.log(this === window); // true
this.name = "全局";
console.log(window.name); // "全局"
```

### 2. Node.js 环境

- REPL 交互终端：顶层 this 等价于全局对象 globalThis
- 模块文件（.js）：顶层 this = {} 空对象，不等于 globalThis

### 3. 严格模式补充

全局作用域下严格模式不影响全局 this，依然指向全局对象。

> 注意：globalThis 是 ES 标准，统一跨环境全局对象，推荐使用。

## 二、普通函数 this 的 4 大绑定规则（优先级从小到大）

优先级排序：

```
默认绑定 < 隐式绑定 < 显式绑定 (call/apply) < new 绑定
```

同一函数调用同时满足多条规则，优先级高的覆盖低的。

### 1. 默认绑定（Default Binding）

场景：函数独立调用，没有任何调用主体

规则：

- 非严格模式：this → 全局对象 (window/globalThis)
- 严格模式 use strict：this → undefined

```js
function fn() {
  console.log(this);
}
fn(); // 默认绑定，非严格模式 this = 全局

// 严格模式
function fn2() {
  "use strict";
  console.log(this); // undefined
}
fn2();
```

⚠️ 常见陷阱：函数赋值后单独调用、回调函数独立执行，都会触发默认绑定

```js
const obj = {
  a: 1,
  fn: function(){ console.log(this) }
}
const f = obj.fn;
f(); // 独立调用 → 默认绑定！不是obj
```



### 2. 隐式绑定（Implicit Binding）

场景：通过对象。方法调用 obj.fn()

规则：谁调用函数，this 就指向谁（调用时的前面那个对象）

```js
const obj = {
  name: "张三",
  say: function(){
    console.log(this.name);
  }
}
obj.say(); // this → obj  输出：张三
```

⚠️ 隐式丢失（高频面试坑）

```js
const fn = obj.say;
fn(); // 单独调用，隐式绑定丢失，降级为默认绑定
```

链式调用只看最后一层

```js
const obj1 = {
  name: "obj1",
  obj2: {
    name: "obj2",
    fn: function(){ console.log(this.name) }
  }
}
obj1.obj2.fn(); // this → obj2，只看紧邻调用的对象
```



### 3. 显式绑定（Explicit Binding）

使用三个方法强制指定 this：

- `func.call(thisArg, 参数1, 参数2)`
- `func.apply(thisArg, [参数数组])`
- `func.bind(thisArg)`

call / apply：立即执行函数

```js
function say() {
  console.log(this.name);
}
const person = { name: "李四" };
say.call(person);   // this = person
say.apply(person);  // this = person
```

区别：

- call：参数逐个传入
- apply：参数放到数组传入
- bind：硬绑定，返回新函数，不会立即执行

硬绑定优先级高于隐式绑定、默认绑定，无法被再次修改（call/apply 也改不了）

```js
const bindFn = say.bind(person);
bindFn(); // this永久锁定person
bindFn.call({name:"test"}); // 依然是person，bind硬绑定无法覆盖
```

特殊规则

如果给 call/apply/bind 传入：null / undefined
非严格模式下，会自动替换为全局对象；严格模式 this 就是传入的 null/undefined。

```js
say.call(null); // 非严格模式 this -> 全局
```



### 4. new 绑定（构造函数绑定）

场景：new 函数()

```js
function Person(name) {
  this.name = name;
}
const p = new Person("王五");
console.log(p.name); // 王五
```

new 执行过程（重点）

当 new Person()：

1. 创建一个全新的空对象 {}
2. 新对象的 `__proto__` 指向构造函数 Person.prototype
3. 将构造函数内部的 this 绑定到这个新对象
4. 执行构造函数代码（给 this 添加属性）
5. 如果构造函数没有手动返回一个引用类型（对象 / 数组 / 函数），自动返回新对象；
  如果手动返回引用类型，new 结果为该返回值，this 绑定失效。

```js
function Test() {
  this.a = 1;
  return {b:2}; // 返回对象，new失效
}
const t = new Test();
console.log(t.a); // undefined
console.log(t.b); // 2
```



### 绑定规则优先级总结

```
new 绑定 >
bind 硬绑定 >
call /apply 显式绑定 >
隐式绑定 obj.fn () >
默认绑定 独立调用
```

> 特例：箭头函数不适用以上四条规则！



## 三、箭头函数的 this



### 1. 核心特性

- ✅ 箭头函数不存在自己的 this！
- 不能使用四条绑定规则，不能用 call/apply/bind 修改 this，不能作为构造函数 new 调用
- 箭头函数的 this：定义时，继承【外层最近一层普通函数作用域】的 this；没有外层函数则继承全局 this。
- 捕获外层 this，一旦捕获永久不变，和怎么调用无关！



### 2. 示例演示

示例 1：外层普通函数

```js
const obj = {
  name: "对象",
  fn: function() {
    // 普通函数this → obj
    const arrow = () => {
      console.log(this.name); // 继承外层fn的this
    }
    arrow();
  }
}
obj.fn(); // 对象
```

示例 2：尝试 call 修改箭头函数 this —— 无效

```js
const arrowFn = () => console.log(this);
arrowFn.call({a:1}); // this依然是全局，不会变成{a:1}
```

示例 3：陷阱！直接写在对象字面量属性上

```js
const obj = {
  name: "测试",
  arrow: () => {
    console.log(this.name);
  }
}
obj.arrow(); 
// ❗ 输出空，不是obj！
// 原因：对象字面量本身不产生作用域，箭头函数外层是全局作用域，this=全局
```

示例 4：定时器经典场景（箭头函数解决 this 丢失）

```js
const obj = {
  name: "小明",
  say: function() {
    // 普通函数this = obj
    setTimeout(()=>{
      console.log(this.name); // 继承say的this → 小明
    }, 1000)
    // 如果换成普通function，定时器独立调用触发默认绑定this=window
  }
}
obj.say();
```



### 3. 箭头函数禁止操作

- 不能 new 箭头函数() → 报错，没有构造器
- 没有 arguments 对象（可用剩余参数 ...args 替代）
- 不存在 prototype 属性
- 不能用作生成器函数（不能加 yield）



### 4. 什么时候不要用箭头函数？

- 对象方法（需要 this 指向当前对象时不要用）
- DOM 事件处理函数（需要 this 指向 DOM 元素时）
- 原型上的方法

错误示范：

```js
const obj = {
  func: () => {
    console.log(this); // this永远不是obj
  }
}
```



## 四、快速判断 this 完整流程图

- 是否是箭头函数？
→ 是：this = 外层作用域 this，四条绑定全部无效
- 普通函数：
  - ① 是否 new 调用？→ new 绑定（新实例）
  - ② 是否 bind 硬绑定？→ bind 指定对象
  - ③ 是否 call/apply 调用？→ 指定对象
  - ④ 是否 obj.fn() 形式调用？→ 隐式绑定 obj
  - ⑤ 独立直接调用 → 默认绑定（全局 / 严格模式 undefined）



## 五、高频面试思考题

```js
var name = "全局";
const obj = {
  name: "局部",
  fn1: function(){ console.log(this.name) },
  fn2: ()=> console.log(this.name)
}

obj.fn1(); // ?
const f1 = obj.fn1;
f1(); // ?
obj.fn2(); // ?
obj.fn1.call({name:"临时"}); // ?
new obj.fn1(); // ?
```

答案：局部，全局，全局，全局，临时，undefined

## 面试题总结

普通函数 `this` **调用时确定**，四条绑定优先级从低到高：

默认绑定 < 隐式绑定 < call/apply 显式绑定 < bind 硬绑定 < new 绑定,同一调用满足多条规则，优先级高覆盖低。

- **默认绑定**：函数独立执行；非严格模式 this = 全局，严格模式 undefined
- **隐式绑定**：`obj.fn()`，this 等于调用点前面的对象；链式调用只看最后一层
- **显式绑定** call/apply 立即执行；bind 返回新函数硬绑定，无法再次修改
- **new 绑定**：new 调用函数，创建新对象，this 绑定新实例；若构造函数手动返回引用类型，new 绑定失效

箭头函数this规则为：

**箭头函数没有自己的 this，不能使用四条绑定规则**；call/apply/bind 无法修改 this

this 在**定义时捕获外层最近一层普通函数作用域的 this**，一旦捕获永久固定，和调用方式无关