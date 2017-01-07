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

对于Set结构，也可以使用数组解构赋值。事实上只要某种数据具有 Iterator 接口，都可以采用数组形式的结构赋值。
```js
// Generator 写法
function* fibs() {
    let a = 0,
        b = 1
    while (true) {
        yield a;                    // 不知道为啥这里一定要加分好才能生效
        [a, b] = [b, a + b]
    }
}

let [first, second, third, fourth, fifth, sixth, seven] = fibs()
console.log(first, second, third, fourth, fifth, sixth, seven)
```

### 默认值

结构赋值允许指定默认值
```js
let [a, b = 'y'] = ['x'],
    [foo = true] = []

console.log(a, b, foo);             // x y true

[a, b = 'a'] = ['b', undefined]
console.log(a, b)                   // b a
```
另外，ES6 内部使用严格相等运算符 `===`，所以如果不是严格等于`undefined`默认值是不会生效的。比如`null`是可以正确赋值的，默认值不会生效。

如果默认值是一个表达式，那么这个表达式是惰性求职的，再用到的时候才会求值
```js
function f() {
    console.log('f() is executed.')
    return 'Hello'
}

let [a = f()] = ['World']
console.log(a)      // World

let [b = f()] = [undefined]
console.log(b)
// f() is executed.
// Hello
```

默认值可以引用结构赋值的其他变量，但是被引用的变量必须已经声明
```js
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
```

### 对象的结构赋值

对象也可以用于对象，不同之处在于，数组必须按顺序赋值，对应的位置赋值对应的位置；而对象只需要属性名相同，就可以正确赋值。
```js
let {foo, bar} = {bar: 'World', foo: 'Hello'}
console.log(foo, bar)       // Hello World

let {baz} = {bar: 'World', foo: 'Hello'}
console.log(baz)            // undefined
```

如果变量名和属性名不一致，应该写成如下这样
```js
let {foo: f, bar: b} = {bar: 'World', foo: 'Hello'}
console.log(f, b)           // Hello World
```

这就说明对象解构赋值就是下面形式的简写
```js
let {foo: foo, bar: bar} = {bar: 'World', foo: 'Hello'}
console.log(foo, bar)           // Hello World
```
也就是说，对象的结构赋值的内部机制，是先找到同名属性，然后再给对应的变量赋值，真正被赋值的是后者，而不是前者。
```js
let {foo: bar} = {foo: 'aaa', bar: 'bbb'}
console.log(bar)            // aaa
console.log(foo)            // ReferenceError: foo is not defined
```

采用这种写法的时候，变量的声明和赋值的一体的，`let` 和 `const`来说，变量不能重复声明，如果赋值的变量以前声明过，就会报错。
```js
let foo
let {foo} = {foo: 1}            // Identifier 'foo' has already been declared

let baz
let {bar: baz} = {bar: 1}       // Identifier 'baz' has already been declared
```
`var`命令允许重复声明，则没有这个问题

```js
let foo
({foo} = {foo: 1})      // 括号是必须的，否则{}会被认为是代码块
console.log(foo)        // 1

let baz
({bar: baz} = {bar: 1})
console.log(baz)        // 1
```
括号是必须的，否则会报错，因为解析器将会把开头的大括号理解成一个代码块。

另外，和数组一样，结构也可以用于嵌套结构的对象
```js
let obj = {
    p: [
        'hello',
        {y: 'world'}
    ]
}

let {p: [x, {y}]} = obj
console.log(x, y)       // hello world
```
注意，这时`p`是模式，不是变量，因此不会被赋值。

另外对象解构也可以指定默认值，依然使用严格属性等于`undefined`来判断是否赋值；解构失败变量的值就是`undefined`

如果要将一个已经声明的变量用于结构赋值，就需要加上圆括号，和上面一样，开始的大括号会被解析成代码块

对象的解构赋值，可以很方便的将现有对象的方法，赋值到某个变量。
```js
let {log, sin, cos, pow} = Math
console.log(log(pow(Math.E, 20)))       // 2
console.log(sin(Math.PI / 2))           // 1
```

由于数组的 本质是特殊的对象，因此对数组进行对象属性的解构。
```js
let arr = [1, 2, 3]
let {0: f, [arr.length - 1]: l} = arr
console.log(f, l)       // 1 3
```
另外：方括号这种写法，属于“属性名表达式”

### 字符串解构赋值

字符串也可以解构赋值，字符串被转换成了一个类似数组的对象。
```js
let [a, b, c, d, e] = 'hello'
console.log(a, b, c, d, e)      // h e l l o
```

数组对象还有一个`length`对象，因此还可以对象解构赋值。
```js
let {length: foo} = 'hello'
console.log(foo)        // 5
```

### 数值和布尔值的解构赋值

解构赋值时，如果等号右边是数值和布尔值，则会首先转换为对象。
```js
let {toString: s} = 123
console.log(s === Number.prototype.toString)        // true
console.log(s)                                      // [Function: toString]

let {toString: t} = true
console.log(t === Boolean.prototype.toString)       // true
console.log(t)                                      // [Function: toString]
```
解构赋值的规则是，只要等号的右边的值不是对象，就先将其转换为对象。由于`undefined`和`null`无法转换为对象，所以对他们进行结构赋值都会报错。
```js
let {prop: x} = undefined       // Cannot match against 'undefined' or 'null'.
let {prop: y} = null            // Cannot match against 'undefined' or 'null'.
```

### 函数参数的解构赋值
函数的参数也可以使用解构赋值
```js
function add([x, y]) {
    return x + y
}

console.log(add([1, 3]))                                    // 4
console.log([[1, 2], [3, 4]].map(([x, y]) => x + y))        // [ 3, 7 ]
```

函数参数的解构也可以使用默认值。
```js
function move({x = 0, y = 0} = {}) {
    return [x,  y]
}

console.log(move({x: 1, y: 2}))         // [ 1, 2 ]
console.log(move({x: 1}))               // [ 1, 0 ]
console.log(move({}))                   // [ 0, 0 ]
console.log(move())                     // [ 0, 0 ]
```

另外，`undefined`就会触发默认值
```js
console.log([1, undefined, 3].map((x = 'YES') => x))        // [ 1, 'YES', 3 ]
```

### 用途
变量解构赋值的用途很多

#### 交换变量
```js
let x = 'a',
    y = 'b';
[x, y] = [y, x]
console.log(x, y)       // b a
```
#### 函数返回多个值
函数只能返回一个值，如果要返回多个值，只能将它们放在数组或者对象里返回，有了解构赋值，取出这些值就非常方便了。
```js
function fun1() {
    return [1, 2, 3]
}

let [a, b, c] = fun1()
console.log(a, b, c)            // 1 2 3

function fun2() {
    return {
        foo: 'foo',
        bar: 'bar'
    }
}
let {foo, bar} = fun2()
console.log(foo, bar)           // foo bar
```

#### 函数参数的定义
解构赋值可以方便地将一组参数与变量名对应起来。
```js
// 有序参数
function f1([x, y, z]) {
    // ...
}
f1([1, 2, 3])

// 无序的参数
function f2({x, y, z}) {
    // ...
}
f2({z: 1, y: 2, x: 3})
```

#### 提取JSON数据
解构赋值对提取JSON对象中的数据，尤其有用
```js
let json = {
    id: 12,
    msg: 'Hello world!',
    status: [100, 200]
}

let {id, msg, status} = json
console.log(id, msg, status)        // 12 'Hello world!' [ 100, 200 ]
```

#### 函数参数的默认值
指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。
```js
jQuery.ajax = function (url, {
    async = true,
    beforeSend = function () {},
    cache = true,
    complete = function () {},
    crossDomain = false,
    global = true,
    // ... more config
}) {
    // ... do stuff
}
```

#### 遍历Map解构
任何部署了 Iterator 接口的对象，都可以使用`for ... of`遍历
```js
let map = new Map()
map.set('a', 'Hello')
map.set('b', 'World')

// 获取键和值
for (let [k, v] of map) {
    console.log(k, v)
}

// 只获取键名
for (let [k] of map) {
    console.log(k)
}

// 只获取值
for (let [, v] of map) {
    console.log(v)
}
```

#### 输入模块指定的方法
加载模块时，往往需要指定输入那些方法。解构赋值使得输入语句非常清晰。
```js
const { SourceMapConsumer, SourceNode } = require("source-map");
```


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



