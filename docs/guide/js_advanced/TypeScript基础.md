# TypeScript 基础

## 前言

这里主要记录学习 TypeScript 过程中，易混淆知识点的总结笔记。

## 空值

使用 `void` 表示没有任何返回值的函数：
```ts
function alertName(): void {
    alert('My name is Tom')
}
```

声明一个 `void` 类型的变量没有什么用，因为你只能将它赋值为 `undefined` 和 `null`（只在 --[strictNullChecks](https://www.tslang.cn/docs/handbook/tsconfig-json.html) 未指定时）：
```ts
let unusable: void = undefined
```

## Null 和 Undefined

它们两个是来定义原始数据类型：
```ts
let n: null = null
let u: undefined = undefined
```

与 `void` 的区别是，`undefined` 和 `null` 是**所有类型的子类型**。也就是说 `undefined` 类型的变量，可以赋值给 `number` 类型的变量：
```ts
// 这样不会报错
let num: number = undefined

// 这样也不会报错
let u: undefined
let num: number = u
```

而 `void` 类型的变量不能赋值给 `number` 类型的变量：
```ts
// 这样会报错
let vo: void
let num: number = vo
```

## 未声明类型的变量

变量如果在声明的时候，未指定其类型，那么它会被识别为任意值类型：
```js
let name
name = 'huitoutunao'
name = 7
```
等价于
```ts
let name: any
name = 'huitoutunao'
name = 7
```

## 类型推论

举个例子：
```ts
let myNumber = 'seven'
myNumber = 7 // 报错
```
等价于
```ts
let myNumber: string = 'seven'
myNumber = 7
```
**如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 any 类型而完全不被类型检查。**

## 联合类型

举个例子：
```ts
let myNumber: string | number
myNumber = 'seven'
myNumber = 7

myNumber = true // 报错
```
允许 `myNumber` 的类型是 `string` 或 `number`, 但不能是其他类型。

### 访问联合类型的属性或方法

当 TypeScript 不确定一个联合类型的变量到底是哪个类型的时候，只能访问此联合类型的所有类型里共有的属性或方法：
```ts
function getArrLength(data: string | number): number {
    return data.length
}
// 运行上面代码报错
```

`length` 不是 `string` 和 `number` 的共有属性，所以会报错，修改如下：
```ts
function getArrLength(data: string | number): number {
    return data.toSting()
}
```

联合类型的变量在被赋值的时候，会根据类型推论的规则推断出一个类型：
```ts
let myNumber: string | number
myNumber = 'seven'
console.log(myNumber.length) // 5

myNumber = 7
console.log(myNumber.length) // 报错，此时 myNumber 被推论为 number 类型，而这个类型没有 length 属性
```

## 对象类型接口

举个简单例子：
```ts
interface Person {
    name: string;
    age: number;
}

let huitoutunao: Person = {
    name: 'huitoutunao',
    age: 25
}
```

上面的例子中，定义了一个接口 `Person`，接着定义了一个变量 `huitoutunao`，它的类型是 `Person`。这样，我们就约束了 `huitoutunao` 的形状必须和接口 `Person` 一致。**接口一般首字母大写。**

定义的变量必须和接口的属性一致。

### 可选属性

可选属性的含义是该属性可以不存在，但仍然不允许添加未定义的属性：
```ts
interface Person {
    name: string;
    age?: number;
}

// 正确
let huitoutunao: Person = {
    name: 'huitoutunao'
}

// 错误
let huitoutunao: Person = {
    name: 'huitoutunao',
    job: 'frontend'
}
```

### 任意属性

```ts
interface Person {
    name: string;
    age?: number;
    [propName: string]: any;
}

let huitoutunao: Person = {
    name: 'huitoutunao',
    gender: 'male'
}
```

使用 `[propName: string]` 定义了任意属性取 `string` 类型的值。

**一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集：**
```ts
interface Person {
    name: string;
    age?: number;
    [propName: string]: string;
}

// 报错
let huitoutunao: Person = {
    name: 'huitoutunao',
    age: 25,
    gender: 'male'
}
```

因为定义任意属性（字符串类型）的返回值必须是字符串类型或者是它的子集，但是 age（字符串类型）返回值是 `number` 类型，所以会报错。

也可以在任意属性中使用联合类型：
```ts
interface Person {
    name: string;
    age?: number;
    [propName: string]: string | number;
}

// 正确
let huitoutunao: Person = {
    name: 'huitoutunao',
    age: 25,
    gender: 'male'
}
```

### 只读属性

`readonly` 定义只读属性：
```ts
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}

let huitoutunao: Person = {
    id: 89757,
    name: 'huitoutunao',
    gender: 'male'
}

huitoutunao.id = 9527 // 报错
```

注意，只读的约束存在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候：
```ts
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}

let huitoutunao: Person = {
    name: 'huitoutunao',
    gender: 'male'
}

// 在对 huitoutunao 进行赋值的时候，没有给 id 赋值，所以报错了。
// 给 huitoutunao.id 赋值的时候，由于它是只读属性，所以报错了。
huitoutunao.id = 89757
```

## 数组的类型

举个简单例子：
```ts
let fibonacci: number[] = [1, 1, 2, 3, 5]

let fibonacci: number[] = [1, '1', 2, 3, 5] // 报错：数组的项中不允许出现其他的类型

fibonacci.push('9') // 报错：不允许添加除 number 类型的其他数据
```
**用 any 表示数组中允许出现任意类型：**
```ts
let list: any[] = ['huitoutunao', 25, { job: 'frontend' }]
```

### 数组的泛型

`Array<elemType>` 表示数组：
```ts
let fibonacci: Array<number> = [1, 1, 2, 3, 5]
```

### 类数组

类数组不能用普通的数组的方式来描述，应该使用接口：
```ts
function func() {
    let args: {
        [index: number]: number;
        length: number;
        callee: Function;
    } = arguments;
}
```

事实上常用的类数组都有自己的接口定义，如 `IArguments`：
```ts
interface IArguments {
    [index: number]: any;
    length: number;
    callee: Function;
}
```

## 函数的类型

### 函数声明

简单例子：
```ts
function sum(x: number, y: number): number {
    return x + y
}

sum(1, 2) // 正确
sum(1) // 报错
```

输入参数个数和类型须和定义的保持一致，函数返回值类型是 `number`。

### 函数表达式

简单例子：
```ts
let sum = function (x: number, y: number): number {
    return x + y
}

// 另一种写法
let sum = (x: number, y: number) => number = function(x, y) {
    return x + y
}
```

在 TypeScript 的类型定义中，`=>` 用来表示函数的定义，左边是输入类型，需要用括号括起来，右边是输出类型。

### 可选参数
```ts
function build(name: string, age?: number): string {
    return `${name}.${age}`
}

let a = build('tom', 24) // 正确
let b = build('jack') // 正确
```

**注意：可选参数后面不允许再出现必需参数了。**

### 参数默认值

举个例子：
```ts
function build(name: string, age: number = 18): string {
    return `${name}.${age}`
}

let a = build('tom', 24) // 正确
let b = build('jack') // 正确
```

**不受可选参数必须接在必需参数后面限制。**

### 剩余参数

举个例子：
```ts
function push(array: any[], ...items: any[]) {
    items.forEach(function(item) {
        array.push(item)
    })
}

let a = []
push(a, 1, 2, 3)
```

**注意：剩余参数只能是最后一个参数。**

### 重载

举个例子：
```ts
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string | void {
    if (typeof x === 'number') {
        return 11
    } else if (typeof x === 'string') {
        return '11'
    }
}
```

**注意：TypeScript 会优先从最前面的函数定义开始匹配，所以多个函数定义如果有包含关系，需要优先把精确的定义写在前面。**

## 类型断言

### 概念

类型断言可以用来手动指定一个值的类型。

### 语法

```ts
值 as 类型
或
<类型>值
```

推荐使用 `值 as 类型 `。

### 用途

#### 将一个联合类型断言为其中一个类型

举个例子：
```ts
interface Dog {
    name: string;
    run(): void;
}

interface Fish {
    name: string;
    swim(): void;
}

function isFish(animal: Fish | Dog) {
    if (typeof (animal as Fish).swim === 'function') {
        return true
    }
    return false
}
```

此时 `animal` 是 `Fish` 类型。虽然 Typescript 编译器可以成功通过，但是无法避免运行时错误，所以不要滥用类型断言。

#### 将一个父类断言为更加具体的子类

```ts
class childA extends Error {
    code: number = 1
}
class childB extends Error {
    statusCode: number = 200
}

function isApiError(error: Error) {
    if (typeof (error as childA).code === 'number') {
        return true
    }
    return false
}
```

在 TypeScript 中，`class` 也可以定义类型接口。

#### 将任何一个类型断言为 `any`

举个例子：
```ts
(window as any).foo = 1
```

**它极有可能掩盖了真正的类型错误，所以如果不是非常确定，就不要使用 as any。**

**总之，一方面不能滥用 as any，另一方面也不要完全否定它的作用，我们需要在类型的严格性和开发的便利性之间掌握平衡（这也是 TypeScript 的设计理念之一），才能发挥出 TypeScript 最大的价值。**

#### 将 `any` 断言为一个具体的类型

```ts
function getData(key: string): any {
    return (window as any).data[key]
}

interface Dog {
    name: string;
    run(): void;
}

const ming = getData('ming') as Dog
ming.run()
```

上面的例子中，我们调用完 `getData` 之后，立即将它断言为 `Dog` 类型。这样的话明确了 `ming` 的类型，后续对 `ming` 的访问时就有了代码补全，提高了代码的可维护性。


## 参考文献

- [TypeScript](https://www.tslang.cn/)
- [TypeScript 入门教程](https://ts.xcatliu.com/)
- [TypeScript Handbook](https://zhongsp.gitbooks.io/typescript-handbook/content/)