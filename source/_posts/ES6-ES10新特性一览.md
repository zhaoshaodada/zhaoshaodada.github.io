---
title: ES6、ES7、ES8、ES9、ES10新特性一览
date: 2026-07-22 12:00:00
tags:
  - ES6
  - JavaScript
categories:
  - 教程
keywords: "ES6,ES7,ES8,ES9,ES10,JavaScript,新特性"
---

> 本文转载自简书文章 [ES6、ES7、ES8、ES9、ES10新特性一览](https://www.jianshu.com/p/065766773752)，作者：Vicky丶Amor

ES全称ECMAScript，ECMAScript是ECMA制定的标准化脚本语言。目前JavaScript使用的ECMAScript版本为[ECMA-417](https://www.ecma-international.org/publications/standards/Ecma-417.htm)。关于ECMA的最新资讯可以浏览 [ECMA news](https://www.ecma-international.org/news/index.html)查看。

ECMA规范最终由[TC39](https://github.com/tc39)敲定。TC39由包括浏览器厂商在内的各方组成，他们开会推动JavaScript提案沿着一条严格的发展道路前进。 从提案到入选ECMA规范主要有以下几个阶段：

- Stage 0: strawman——最初想法的提交。
- Stage 1: proposal（提案）——由TC39至少一名成员倡导的正式提案文件，该文件包括API事例。
- Stage 2: draft（草案）——功能规范的初始版本，该版本包含功能规范的两个实验实现。
- Stage 3: candidate（候选）——提案规范通过审查并从厂商那里收集反馈
- Stage 4: finished（完成）——提案准备加入ECMAScript，但是到浏览器或者Nodejs中可能需要更长的时间。

## ES6新特性（2015）

ES6的特性比较多，在 ES5 发布近 6 年（2009-11 至 2015-6）之后才将其标准化。两个发布版本之间时间跨度很大，所以ES6中的特性比较多。 在这里列举几个常用的：

- 类
- 模块化
- 箭头函数
- 函数参数默认值
- 模板字符串
- 解构赋值
- 延展操作符
- 对象属性简写
- Promise
- Let与Const

### 1.类（class）

对熟悉Java，object-c，c#等纯面向对象语言的开发者来说，都会对class有一种特殊的情怀。ES6 引入了class（类），让JavaScript的面向对象编程变得更加简单和易于理解。

```javascript
class Animal {
  // 构造函数，实例化的时候将会被调用，如果不指定，那么会有一个不带参数的默认构造函数.
  constructor(name, color) {
    this.name = name;
    this.color = color;
  }
  // toString 是原型对象上的属性
  toString() {
    console.log('name:' + this.name + ',color:' + this.color);
  }
}

var animal = new Animal('dog', 'white'); //实例化Animal
animal.toString();

console.log(animal.hasOwnProperty('name'));  // true
console.log(animal.hasOwnProperty('toString'));  // false
console.log(animal.__proto__.hasOwnProperty('toString'));  // true

class Cat extends Animal {
  constructor(action) {
    // 子类必须要在constructor中指定super 函数，否则在新建实例的时候会报错.
    // 如果没有置顶consructor,默认带super函数的constructor将会被添加
    super('cat', 'white');
    this.action = action;
  }
  toString() {
    console.log(super.toString());
  }
}

var cat = new Cat('catch')
cat.toString();

// 实例cat 是 Cat 和 Animal 的实例，和Es5完全一致。
console.log(cat instanceof Cat);  // true
console.log(cat instanceof Animal);  // true
```

### 2.模块化(Module)

ES5不支持原生的模块化，在ES6中模块作为重要的组成部分被添加进来。模块的功能主要由 export 和 import 组成。每一个模块都有自己单独的作用域，模块之间的相互调用关系是通过 export 来规定模块对外暴露的接口，通过import来引用其它模块提供的接口。同时还为模块创造了命名空间，防止函数的命名冲突。

#### 导出(export)

ES6允许在一个模块中使用export来导出多个变量或函数。

**导出变量**

```javascript
//test.js
export var name = 'Rainbow'
```

心得：ES6不仅支持变量的导出，也支持常量的导出。`export const sqrt = Math.sqrt` 导出常量

ES6将一个文件视为一个模块，上面的模块通过 export 向外输出了一个变量。一个模块也可以同时往外面输出多个变量。

```javascript
//test.js
var name = 'Rainbow';
var age = '24';
export { name, age };
```

**导出函数**

```javascript
// myModule.js
export function myModule(someArg) {
  return someArg;
}
```

#### 导入(import)

定义好模块的输出以后就可以在另外一个模块通过import引用。

```javascript
import { myModule } from 'myModule';  // main.js
import { name, age } from 'test';  // test.js
```

心得：一条import 语句可以同时导入默认函数和其它变量。

```javascript
import defaultMethod, { otherMethod } from 'xxx.js';
```

### 3.箭头（Arrow）函数

这是ES6中最令人激动的特性之一。`=>`不只是关键字function的简写，它还带来了其它好处。箭头函数与包围它的代码共享同一个`this`，能帮你很好的解决this的指向问题。有经验的JavaScript开发者都熟悉诸如`var self = this;`或`var that = this`这种引用外围this的模式。但借助`=>`，就不需要这种模式了。

#### 箭头函数的结构

箭头函数的箭头=>之前是一个空括号、单个的参数名、或用括号括起的多个参数名，而箭头之后可以是一个表达式（作为函数的返回值），或者是用花括号括起的函数体（需要自行通过return来返回值，否则返回的是undefined）。

```javascript
// 箭头函数的例子
() => 1
v => v + 1
(a, b) => a + b
() => {
  alert("foo");
}
e => {
  if (e == 0) {
    return 0;
  }
  return 1000 / e;
}
```

心得：不论是箭头函数还是bind，每次被执行都返回的是一个新的函数引用，因此如果你还需要函数的引用去做一些别的事情（譬如卸载监听器），那么你必须自己保存这个引用。

#### 卸载监听器时的陷阱

**错误的做法**

```javascript
class PauseMenu extends React.Component {
  componentWillMount() {
    AppStateIOS.addEventListener('change', this.onAppPaused.bind(this));
  }
  componentWillUnmount() {
    AppStateIOS.removeEventListener('change', this.onAppPaused.bind(this));
  }
  onAppPaused(event) {
  }
}
```

**正确的做法**

```javascript
class PauseMenu extends React.Component {
  constructor(props) {
    super(props);
    this._onAppPaused = this.onAppPaused.bind(this);
  }
  componentWillMount() {
    AppStateIOS.addEventListener('change', this._onAppPaused);
  }
  componentWillUnmount() {
    AppStateIOS.removeEventListener('change', this._onAppPaused);
  }
  onAppPaused(event) {
  }
}
```

除上述的做法外，我们还可以这样做：

```javascript
class PauseMenu extends React.Component {
  componentWillMount() {
    AppStateIOS.addEventListener('change', this.onAppPaused);
  }
  componentWillUnmount() {
    AppStateIOS.removeEventListener('change', this.onAppPaused);
  }
  onAppPaused = (event) => {
    //把函数直接作为一个arrow function的属性来定义，初始化的时候就绑定好了this指针
  }
}
```

需要注意的是：不论是bind还是箭头函数，每次被执行都返回的是一个新的函数引用，因此如果你还需要函数的引用去做一些别的事情（譬如卸载监听器），那么你必须自己保存这个引用。

### 4.函数参数默认值

ES6支持在定义函数的时候为其设置默认值：

```javascript
function foo(height = 50, color = 'red') {
  // ...
}
```

不使用默认值：

```javascript
function foo(height, color) {
  var height = height || 50;
  var color = color || 'red';
  //...
}
```

这样写一般没问题，但当参数的布尔值为`false`时，就会有问题了。比如，我们这样调用foo函数：

```javascript
foo(0, "")
```

因为0的布尔值为`false`，这样height的取值将是50。同理color的取值为'red'。

所以说，`函数参数默认值`不仅能是代码变得更加简洁而且能规避一些问题。

### 5.模板字符串

ES6支持`模板字符串`，使得字符串的拼接更加的简洁、直观。

不使用模板字符串：

```javascript
var name = 'Your name is ' + first + ' ' + last + '.'
```

使用模板字符串：

```javascript
var name = `Your name is ${first} ${last}.`
```

在ES6中通过`${}`就可以完成字符串的拼接，只需要将变量放在大括号之中。

### 6.解构赋值

解构赋值语法是JavaScript的一种表达式，可以方便的从数组或者对象中快速提取值赋给定义的变量。

#### 获取数组中的值

从数组中获取值并赋值到变量中，变量的顺序与数组中对象顺序对应。

```javascript
var foo = ["one", "two", "three", "four"];

var [one, two, three] = foo;
console.log(one); // "one"
console.log(two); // "two"
console.log(three); // "three"

//如果你要忽略某些值，你可以按照下面的写法获取你想要的值
var [first, , , last] = foo;
console.log(first); // "one"
console.log(last); // "four"

//你也可以这样写
var a, b; //先声明变量

[a, b] = [1, 2];
console.log(a); // 1
console.log(b); // 2
```

如果没有从数组中的获取到值，你可以为变量设置一个默认值。

```javascript
var a, b;

[a = 5, b = 7] = [1];
console.log(a); // 1
console.log(b); // 7
```

通过解构赋值可以方便的交换两个变量的值。

```javascript
var a = 1;
var b = 3;

[a, b] = [b, a];
console.log(a); // 3
console.log(b); // 1
```

#### 获取对象中的值

```javascript
const student = {
  name: 'Ming',
  age: '18',
  city: 'Shanghai'
};

const { name, age, city } = student;
console.log(name); // "Ming"
console.log(age); // "18"
console.log(city); // "Shanghai"
```

### 7.延展操作符(Spread operator)

`延展操作符...`可以在函数调用/数组构造时, 将数组表达式或者string在语法层面展开；还可以在构造对象时, 将对象表达式按key-value的方式展开。

#### 语法

函数调用：

```javascript
myFunction(...iterableObj);
```

数组构造或字符串：

```javascript
[...iterableObj, '4', ...'hello', 6];
```

构造对象时,进行克隆或者属性拷贝（ECMAScript 2018规范新增特性）：

```javascript
let objClone = { ...obj };
```

#### 应用场景

在函数调用时使用延展操作符

```javascript
function sum(x, y, z) {
  return x + y + z;
}
const numbers = [1, 2, 3];

//不使用延展操作符
console.log(sum.apply(null, numbers));

//使用延展操作符
console.log(sum(...numbers)); // 6
```

构造数组

没有展开语法的时候，只能组合使用 push，splice，concat 等方法，来将已有数组元素变成新数组的一部分。有了展开语法, 构造新数组会变得更简单、更优雅：

```javascript
const stuendts = ['Jine', 'Tom'];
const persons = ['Tony', ...stuendts, 'Aaron', 'Anna'];
console.log(persons) // ["Tony", "Jine", "Tom", "Aaron", "Anna"]
```

和参数列表的展开类似, `...` 在构造字数组时, 可以在任意位置多次使用。

数组拷贝

```javascript
var arr = [1, 2, 3];
var arr2 = [...arr]; // 等同于 arr.slice()
arr2.push(4);
console.log(arr2) // [1, 2, 3, 4]
```

展开语法和 Object.assign() 行为一致, 执行的都是浅拷贝(只遍历一层)。

连接多个数组

```javascript
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
var arr3 = [...arr1, ...arr2]; // 将 arr2 中所有元素附加到 arr1 后面并返回
//等同于
var arr4 = arr1.concat(arr2);
```

#### 在ECMAScript 2018中延展操作符增加了对对象的支持

```javascript
var obj1 = { foo: 'bar', x: 42 };
var obj2 = { foo: 'baz', y: 13 };

var clonedObj = { ...obj1 };
// 克隆后的对象: { foo: "bar", x: 42 }

var mergedObj = { ...obj1, ...obj2 };
// 合并后的对象: { foo: "baz", x: 42, y: 13 }
```

#### 在React中的应用

通常我们在封装一个组件时，会对外公开一些 props 用于实现功能。大部分情况下在外部使用都应显示的传递 props 。但是当传递大量的props时，会非常繁琐，这时我们可以使用 `...(延展操作符,用于取出参数对象的所有可遍历属性)` 来进行传递。

一般情况下我们应该这样写

```javascript
<CustomComponent name='Jine' age={21} />
```

使用 ... ，等同于上面的写法

```javascript
const params = {
  name: 'Jine',
  age: 21
}
<CustomComponent {...params} />
```

配合解构赋值避免传入一些不需要的参数

```javascript
var params = {
  name: '123',
  title: '456',
  type: 'aaa'
}

var { type, ...other } = params;

<CustomComponent type='normal' number={2} {...other} />
//等同于
<CustomComponent type='normal' number={2} name='123' title='456' />
```

### 8.对象属性简写

在ES6中允许我们在设置一个对象的属性的时候不指定属性名。

不使用ES6

```javascript
const name = 'Ming', age = '18', city = 'Shanghai';

const student = {
  name: name,
  age: age,
  city: city
};
console.log(student); //{name: "Ming", age: "18", city: "Shanghai"}
```

对象中必须包含属性和值，显得非常冗余。

使用ES6

```javascript
const name = 'Ming', age = '18', city = 'Shanghai';

const student = {
  name,
  age,
  city
};
console.log(student); //{name: "Ming", age: "18", city: "Shanghai"}
```

对象中直接写变量，非常简洁。

### 9.Promise

Promise 是异步编程的一种解决方案，比传统的解决方案callback更加的优雅。它最早由社区提出和实现的，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。

不使用ES6

嵌套两个setTimeout回调函数：

```javascript
setTimeout(function () {
  console.log('Hello'); // 1秒后输出"Hello"
  setTimeout(function () {
    console.log('Hi'); // 2秒后输出"Hi"
  }, 1000);
}, 1000);
```

使用ES6

```javascript
var waitSecond = new Promise(function (resolve, reject) {
  setTimeout(resolve, 1000);
});

waitSecond
  .then(function () {
    console.log("Hello"); // 1秒后输出"Hello"
    return waitSecond;
  })
  .then(function () {
    console.log("Hi"); // 2秒后输出"Hi"
  });
```

上面的的代码使用两个then来进行异步编程串行化，避免了回调地狱。

### 10.支持let与const

在之前JS是没有块级作用域的，const与let填补了这方便的空白，const与let都是块级作用域。

使用var定义的变量为函数级作用域：

```javascript
{
  var a = 10;
}

console.log(a); // 输出10
```

使用let与const定义的变量为块级作用域：

```javascript
{
  let a = 10;
}

console.log(a); // -1 or Error"ReferenceError: a is not defined"
```

## ES7新特性（2016）

ES2016添加了两个小的特性来说明标准化过程：

- 数组includes()方法，用来判断一个数组是否包含一个指定的值，根据情况，如果包含则返回true，否则返回false。
- a ** b指数运算符，它与 Math.pow(a, b)相同。

### 1.Array.prototype.includes()

`includes()` 函数用来判断一个数组是否包含一个指定的值，如果包含则返回 `true`，否则返回`false`。

`includes` 函数与 `indexOf` 函数很相似，下面两个表达式是等价的：

```javascript
arr.includes(x)
arr.indexOf(x) >= 0
```

接下来我们来判断数字中是否包含某个元素：

在ES7之前的做法

使用`indexOf()`验证数组中是否存在某个元素，这时需要根据返回值是否为-1来判断：

```javascript
let arr = ['react', 'angular', 'vue'];

if (arr.indexOf('react') !== -1) {
  console.log('react存在');
}
```

使用ES7的includes()

使用includes()验证数组中是否存在某个元素，这样更加直观简单：

```javascript
let arr = ['react', 'angular', 'vue'];

if (arr.includes('react')) {
  console.log('react存在');
}
```

### 2.指数操作符

在ES7中引入了指数运算符`**`，`**`具有与`Math.pow(..)`等效的计算结果。

不使用指数操作符

使用自定义的递归函数calculateExponent或者Math.pow()进行指数运算：

```javascript
function calculateExponent(base, exponent) {
  if (exponent === 1) {
    return base;
  }
  else {
    return base * calculateExponent(base, exponent - 1);
  }
}

console.log(calculateExponent(2, 10)); // 输出1024
console.log(Math.pow(2, 10)); // 输出1024
```

使用指数操作符

使用指数运算符`**`，就像+、-等操作符一样：

```javascript
console.log(2 ** 10); // 输出1024
```

## ES8新特性（2017）

- async/await
- `Object.values()`
- `Object.entries()`
- String padding: `padStart()`和`padEnd()`，填充字符串达到当前长度
- 函数参数列表结尾允许逗号
- `Object.getOwnPropertyDescriptors()`
- `ShareArrayBuffer`和`Atomics`对象，用于从共享内存位置读取和写入

### 1.async/await

ES2018引入异步迭代器（asynchronous iterators），这就像常规迭代器，除了`next()`方法返回一个Promise。因此`await`可以和`for...of`循环一起使用，以串行的方式运行异步操作。例如：

```javascript
async function process(array) {
  for await (let i of array) {
    doSomething(i);
  }
}
```

### 2.Object.values()

`Object.values()`是一个与`Object.keys()`类似的新函数，但返回的是Object自身属性的所有值，不包括继承的值。

假设我们要遍历如下对象`obj`的所有值：

```javascript
const obj = { a: 1, b: 2, c: 3 };
```

不使用Object.values() : ES7

```javascript
const vals = Object.keys(obj).map(key => obj[key]);
console.log(vals); // [1, 2, 3]
```

使用Object.values() : ES8

```javascript
const values = Object.values(obj);
console.log(values); // [1, 2, 3]
```

从上述代码中可以看出`Object.values()`为我们省去了遍历key，并根据这些key获取value的步骤。

### 3.Object.entries()

`Object.entries()`函数返回一个给定对象自身可枚举属性的键值对的数组。

接下来我们来遍历上文中的`obj`对象的所有属性的key和value：

不使用Object.entries() : ES7

```javascript
Object.keys(obj).forEach(key => {
  console.log('key:' + key + ' value:' + obj[key]);
})
//key:a value:1
//key:b value:2
//key:c value:3
```

使用Object.entries() : ES8

```javascript
for (let [key, value] of Object.entries(obj)) {
  console.log(`key: ${key} value:${value}`)
}
//key:a value:1
//key:b value:2
//key:c value:3
```

### 4.String padding

在ES8中String新增了两个实例函数`String.prototype.padStart`和`String.prototype.padEnd`，允许将空字符串或其他字符串添加到原始字符串的开头或结尾。

**String.padStart(targetLength, [padString])**

- targetLength: 当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。
- padString: (可选)填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断，此参数的缺省值为 " "。

```javascript
console.log('0.0'.padStart(4, '10')) // 10.0
console.log('0.0'.padStart(20)) //                 0.0
```

**String.padEnd(targetLength, padString)**

- targetLength: 当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。
- padString: (可选)填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断，此参数的缺省值为 " "。

```javascript
console.log('0.0'.padEnd(4, '0')) // 0.00
console.log('0.0'.padEnd(20, '*')) // 0.0******************
```

---

> **说明**：原文后续还包含 ES8 剩余特性（函数参数列表结尾允许逗号、`Object.getOwnPropertyDescriptors()`、`ShareArrayBuffer` 和 `Atomics` 对象）以及 ES9（2018）、ES10（2019）的新特性内容。由于网页内容抓取长度限制，本文暂未完整收录，完整内容请查阅[原文](https://www.jianshu.com/p/065766773752)。
