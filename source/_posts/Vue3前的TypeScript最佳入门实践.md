---
title: Vue3.0前的TypeScript最佳入门实践
date: 2026-07-23 11:00:00
tags:
  - Vue
  - TypeScript
  - 入门
categories:
  - 教程
keywords: "Vue, TypeScript, 入门实践, 类型, 接口"
---

> 转载自：[掘金-前端劝退师](https://juejin.cn/post/6844903865255477261)

## 前言

其实`Vue`官方从`2.6.X`版本开始就部分使用`Ts`重写了。

我个人对更严格类型限制没有积极的看法，毕竟各类转类型的骚写法写习惯了。

然鹅最近的一个项目中，是`TypeScript`+ `Vue`，毛计喇，学之...…真香！

**注意此篇标题的"前"，本文旨在讲`Ts`混入框架的使用，不讲`Class API`**

## 使用官方脚手架构建

```bash
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

新的`Vue CLI`工具允许开发者使用 `TypeScript` 集成环境创建新项目。

只需运行`vue create my-app`。

然后，命令行会要求选择预设。使用箭头键选择`Manually select features`。

接下来，只需确保选择了`TypeScript`和`Babel`选项。

完成此操作后，它会询问你是否要使用`class-style component syntax`。

Vue CLI工具现在将安装所有依赖项并设置项目。

## 项目目录解析

通过`tree`指令查看目录结构后可发现其结构和正常构建的大有不同。

这里主要关注`shims-tsx.d.ts`和`shims-vue.d.ts`两个文件

两句话概括：

- `shims-tsx.d.ts`，允许你以`.tsx`结尾的文件，在`Vue`项目中编写`jsx`代码

- `shims-vue.d.ts`主要用于`TypeScript`识别`.vue`文件，`Ts`默认并不支持导入`vue`文件，这个文件告诉`ts`导入`.vue`文件都按`VueConstructor<Vue>`处理。

此时我们打开亲切的`src/components/HelloWorld.vue`，将会发现写法已大有不同

```vue
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
  @Prop() private msg!: string;
}
</script>
```

## TypeScript极速入门

### 基本类型和扩展类型

`Typescript`与`Javascript`共享相同的基本类型，但有一些额外的类型。

- 元组 `Tuple`

- 枚举 `enum`

- `Any` 与`Void`

#### 基本类型合集

```typescript
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;

let name: string = "bob";
let sentence: string = `Hello, my name is ${ name }.`;

let list: number[] = [1, 2, 3];
let list: Array<number> = [1, 2, 3];

let u: undefined = undefined;
let n: null = null;
```

#### 特殊类型

**元组 Tuple**

想象元组作为有组织的数组，你需要以正确的顺序预定义数据类型。

```typescript
const messyArray = [' something', 2, true, undefined, null];
const tuple: [number, string, string] = [24, "Indrek" , "Lasn"]
```

**枚举 enum**

`enum`类型是对JavaScript标准数据类型的一个补充。使用枚举类型可以为一组数值赋予友好的名字。

```typescript
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;

let colorName: string = Color[2];
console.log(colorName); // 输出'Green'
```

**Void**

在`Typescript`中，**你必须在函数中定义返回类型**。

```typescript
function warnUser(): void {
  console.log("This is my warning message");
}
```

**Any**

就是什么类型都行，当你无法确认在处理什么类型时可以用这个。

```typescript
let person: any = "前端劝退师"
person = 25
person = true 
```

**Never**

`Never`是你永远得不到的类型。具体的行为是：

- `throw new Error(message)`

- `return error("Something failed")`

- `while (true) {}`

#### 类型断言

可以用来手动指定一个值的类型。有两种写法，尖括号和`as`:

```typescript
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
let strLength: number = (someValue as string).length;
```

### 泛型：Generics

在`C#`和`Java`中，可以使用"泛型"来创建可复用的组件，并且组件可支持多种数据类型。

#### 泛型方法

```typescript
function gen_func1<T>(arg: T): T {
  return arg;
}

let gen_func2: <T>(arg: T) => T = function (arg) {
  return arg;
}
```

#### 泛型与Any

- `any`可以代替任意类型，但无法保证类型安全

- 泛型定义了参数类型，保证了类型安全

### 自定义类型：Interface vs Type alias

#### 相同点

都可以用来描述一个对象或函数，都允许拓展。

#### 不同点

**`type`可以而`interface`不行**

- `type`可以声明基本类型别名，联合类型，元组等类型

```typescript
type Name = string

type Pet = Dog | Cat

type PetList = [Dog, Pet]
```

**`interface`可以而`type`不行**

`interface`能够声明合并

```typescript
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}
// User接口为 { name: string, age: number, sex: string }
```

### 实现与继承：implements vs extends

`implement`实现接口，`extends`继承父类。

```typescript
interface IDeveloper {
  name: string;
  age?: number;
}

class dev implements IDeveloper {
  name = 'Alex';
  age = 20;
}
```

### 声明文件与命名空间：declare 和 namespace

```typescript
// shims-vue.d.ts
declare module '*.vue' {
  import Vue from 'vue';
  export default Vue;
}
```
