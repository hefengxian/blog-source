---
title: ES6学习笔记
date: 2016-12-13 15:13:02
categories:
- 日常笔记
tags:
- JavaScript
- ES6
---

ES6（ECMAScript 6.0）已经使用了很久了，但是有很多的东西还是没有学会；学东西不应该浅尝而止，应该系统的学习；记录一些自己的理解。

笔记主要是依据阮一峰老师的教程[ECMAScript 6入门](http://es6.ruanyifeng.com/)学习所得
<!--more-->

## 准备工作
如何运行ES6代码呢？因为要测试和运行新的特性；所以并不需要转换代码到ES5，我们这里直接使用`Node.js`来运行我们的 ES6 编写的代码；安装`Node.js`可以使用`nvm`非常的方便（上Github搜索nvm）。

假设已经拥有如下的环境
+ Ubuntu 16.04
+ Node.js v5.12.0

直接使用`node path/to/*.js`命令运行如下代码
```js
let foo = "Hello ES6"
console.log(foo)
```

会报如下错误
```
let foo = "Hello ES6"
^^^

SyntaxError: Block-scoped declarations (let, const, function, class) not yet supported outside strict mode
    at exports.runInThisContext (vm.js:53:16)
    at Module._compile (module.js:387:25)
    at Object.Module._extensions..js (module.js:422:10)
    at Module.load (module.js:357:32)
    at Function.Module._load (module.js:314:12)
    at Function.Module.runMain (module.js:447:10)
    at startup (node.js:148:18)
    at node.js:405:3
```

提示说我们不能在非严格模式下使用`let, const, function, class`；所以我们在脚本修改一下
```js
"use strict"

let foo = "Hello ES6"
console.log(foo)
```
<del>这样就可以正确的运行 ES6 代码了，以后代码默认前面都加了`"use strict"`</del>

**已经将node升级为当前最新版`v7.2.1`**

## let和const

### let
#### 基本用法
ES6 新增的关键字，用法就是用于声明变量，和`var`类似；区别在于声明的变量是局部的，只在`let`所在的代码块内有效。

比如`for`循环的计数器就可以很适合使用`let`
```js
for (let i = 0; i <= 10; i++) {
    console.log(i)      // 有效
}
// 如果使用的var声明将可以打印出 10
console.log(i)          // 无效 ReferenceError
```

#### 不存在变量提升
```js
console.log(foo)            // undefined
console.log(bar)            // ReferenceError: bar is not defined

var foo
let bar
```
上面代码中，`foo`用`var`声明，会发生变量提升，脚本开始运行的时，变量`foo`就已经存在了，只是没有值所以输出`undefined`；而使用`let`声明的变量并不会发生变量提升，所以在声明之前变量`bar`都是不存在的，这个时候使用它就会报错。

#### 暂时性死区（temporal dead zone, TDZ）
只要块级作用域内，存在`let`命令，它声明的变量就是绑定（binding）在这个区域，不受外部影响
```js
var foo = 123           // 全局变量

if (true) {
    console.log(foo)    // ReferenceError: foo is not defined
    let foo
}
```
ES6中规定，如果区块中存在`let`和`const`关键字，这个区块对这些关键字声明的变量从一开始就形成了封闭的作用域；如果在声明之前使用这个变量那么就会报错。
```js
if (true) {
    // 变量 foo 的 TDZ 开始
    foo = 123
    console.log(foo)

    let foo = 234   // 变量 foo 的 TDZ 结束
    console.log(foo)
}
```
以前`typeof variable`是一个绝对安全的操作，现在有了 TDZ，这个操作就不再是绝对安全的操作了
```js
typeof foo      // ReferenceError: bar is not defined
let foo
```
有些 TDZ 还不容易发现，例如
```js
function foo(x = y, y = 2) {
    return [x, y]
}

console.log(foo())  // ReferenceError: y is not defined
```
> 发现这段代码并不能在node下运行，因为当前版本v5.12.0不能支持参数解构赋值；参见[Node V6支持的新特性](http://wwsun.github.io/posts/nodejs-v6.html)，所以这里升级一下（当前最新版已经是v7.2.1）
> `nvm install v7`安装最新的v7.2.1，`nvm alias default v7`设置默认的版本为v7.2.1；另外v7是不需要严格模式才能执行ES6代码了，直接写ES6代码既可

因为在给`x`赋值的时候，`y`并没有定义；如果令`x = 2, y = x`这样就可以正常运行了
```js
function foo(x = 2, y = x) {
    return [x, y]
}

console.log(foo())      // [ 2, 2 ]
```
据说 ES6 这样做的目的是为了让大家养成良好的变成习惯（其实我是拒绝的！）

#### 不允许重复声明
```js
// SyntaxError: Identifier 'a' has already been declared
function foo() {
    let a = 234
    var a = 2
}

// SyntaxError: Identifier 'a' has already been declared
function bar() {
    let a = 123
    let a = 34
}

// SyntaxError: Identifier 'a' has already been declared
function foo(a) {
    let a = 1
}

// 正确运行
function foo(a) {
    {
        let a = 123
    }
}
```

### 块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这会带来一些不合理的问题

第一种，内层变量覆盖外层变量
```js
var foo = new Date()
function bar() {
    console.log(foo)
    if (false) {
        var foo = 123
    }
}
bar()       // undefined
```
第二种，for循环中内层的变量泄露成全局变量
```js
var string = 'Hello'
var arr = []
for (var i = 0; i < string.length; i++) {
    arr[i] = string[i]
}
console.log(i)          // 5
```

#### ES6的块级作用域
`let`实际上为JavaScript增加了块级作用域
```js
function foo() {
    let n = 1
    if (true) {
        let n = 10
    }
    console.log(n)      // 1
}
foo()
```
ES6 允许块级作用域任意嵌套，不同块可以定义名称相同的变量
```js
{{
    {let i = 1}
    console.log(i)  // ReferenceError: i is not defined
}}

{{  let i = 2
    {
        let i = 1
        console.log(i)      // 1
    }
    console.log(i)          // 2
}}
```
另外 ES6 的块级作用域的出现，使得广泛使用的“立即执行函数表达式 IIFE (Immediately Invoked Function Expression) ”不再必要了
```js
// IIFE 写法
(function () {
    // 逻辑
}())

// 块级作用域写法
{
    // 逻辑
}
```

#### do 表达式
不赘述，只是提案，Node.js v7 都不支持

### const关键字
`const`声明的是一个只读的常量。一旦声明值便不可以改动。因为`const`声明的值并不能改变，所以声明的时候必须初始化
```js
const PI = 3.1415926
console.log(PI)     // 3.1415926
PI = 3              // TypeError: Assignment to constant variable.
const MAX           // SyntaxError: Missing initializer in const declaration
```
`const`的特点和`let`一样，只在声明的块级作用域中有效；`const`声明的变量也不会提升，所以也是存在 TDZ 的；不能重复声明

另外：如果`const`声明的变量是引用型数据，那么常量只是保证引用数据的地址不变而不能保证内容不变
```js
const ARR = []
ARR.push('Hello')
console.log(ARR)        // [ 'Hello' ]
ARR = []                // TypeError: Assignment to constant variable.

const OBJ = {}
OBJ.age = 20
console.log(OBJ.age)    // 20
OBJ = {}                // TypeError: Assignment to constant variable.
```
如果想将对象也冻结，应该使用`Object.freeze`方法
```js
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
ES5 只有两种变量声明的方法：`var`和`function`关键字；ES6 却还添加了`let`,`const`,`import`,`class`；所以 ES6 一共有6种声明变量的方法

### 顶层对象的属性
顶层对象，在浏览器环境表示的是`window`对象，在node中表示`global`对象；在 ES5 中顶层对象的属性和全局变量是等价的
```js
// 在浏览器环境下
window.i = 1
console.log(i)           // 1

i = 2
console.log(window.i)   // 2
```
ES6 改变了这一点；一方面规定，为了保持兼容性，`var`和`function`关键字声明的是全局变量，依旧是顶层对象的属性；另一方面规定，`let`，`const`，`class`声明的全局变量不属于顶层对象的属性。意味着ES越往后面走全局变量将逐渐和顶层对象的属性分离

### global对象
ES5 的顶层对象，本身也是一个问题，因为它在各种实现里不一致
+ 浏览器中，顶层对象是`window`，但 node 和 Web worker 中没有`window`
+ 浏览器和 Web Worker 里面，`self`也指向顶层对象，但是Node没有`self`。
+ Node 里面，顶层对象是`global`，但其他环境都不支持。

ES6 中只是一个提案，不赘述。

## 变量的解构赋值
### 基本用法
ES6允许安装一定的模式，从数组和对象中提取值，对变量进行赋值，这杯称为解构（Destructuring），以前变量赋值只能这样
```js
var a = 1
var b = 2
var c = 3
```
ES6 中我们可以这样写
```js
var [a, b, c] = [1, 2, 3]
```
本质上这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值；下面是一些解构赋值的例子
```js
let [foo, [[bar], baz]] = [1, [[2], 3]]
console.log(foo, bar, baz)                  // 1 2 3

let [,, third] = ['A', 'B', 'C']
console.log(third)                          // C

let [x, , y] = [1, 2, 3]
console.log(x, y)                           // 1 3

let [head, ...tail] = [1, 2, 3, 4, 5]
console.log(head, tail)                     // 1 [ 2, 3, 4, 5 ]

let [i, j, ...k] = ['A']
console.log(i, j, k)                        // A undefined []
```
解构不成功，变量的值就会变成`undefined`；另外不完全解构，左边只是匹配右边一部分，这种情况依然是解构成功的。

如果等号右边不是可遍历的结构那么解构会报错。
```js
let [foo1] = 1              // TypeError: undefined is not a function
let [foo2] = false          // TypeError: undefined is not a function
let [foo3] = NaN            // TypeError: undefined is not a function
let [foo4] = undefined      // TypeError: undefined is not a function
let [foo5] = null           // TypeError: undefined is not a function
let [foo6] = {}             // TypeError: undefined is not a function
```
因为右边的值要么不具备Iterator接口（最后 1 个），要么就是转成对象之后不具备Iterator接口（前面 5 个）。另外解构适用于`var`、`let`、`const`。


## [todo]字符串的扩展
## [todo]正则的扩展
## [todo]数值的扩展
## [todo]数组的扩展
## [todo]函数的扩展
## [todo]对象的扩展
## [todo]Symbol
## [todo]Set和Map数据结构
## [todo]Proxy和Reflect
## [todo]Iterator 和 for ... of 循环
## [todo]Generator函数
## [todo]Promise函数
## [todo]异步操作和Async函数
## [todo]Class
## [todo]Decorator
## [todo]Module
## [todo]编程风格
## [todo]规则
## [todo]二进制
## [todo]SIMD



