---
title: "再看 JavaScript 继承"
date: 2019-12-21T11:47:50+08:00
draft: false
tags: ["JavaScript", "前端", "入门"]
categories: ["从小白开始的前端学习之旅"]
author: "Harvey Zhang"
toc: true
---

学完了整个 JavaScript 基础篇章之后，今天又看了一下所谓的继承，发现我之前理解的继承跟大家一直提到的那个继承不是一回事，于是写一篇文章记录一下。

本文将着重讨论基于原型的继承，也会简单写一下如何用 class 继承。

<!--more-->

---

# Key Points

- 在 JavaScript 中，**函数** `Function` 也是一种 **对象** `Object`

- 关于函数
    1. 所有函数都自带 `prototype`
    2. `prototype` 中自带 `constructor`
    3. `constructor` 里面的东西就是函数的内容
    4. 构造函数首字母大写（约定俗成）

- `对象.__proto__ === 其构造函数.prototype`

---

理解一下上面几句话。

首先，关于函数也是一种对象这个说法，我们在后面会有相关的说明；

其次是关于函数的几个描述，我们可以做几个实验来验证一下：

```js
    let arr = [1, 2, 3]
    let obj = { name: 'Harvey', age: '22'}
    let fn = function(){ console.log('hi') }
    
    console.dir(arr)
    /*
    {
      0: 1,
      1: 2,
      2: 3,
      length: 3,
      __proto__: Array(0)
    }
    */
    
    console.dir(obj)
    /*
    {
      name: "Harvey",
      age: "22",
      __proto__: Object
    }
    */
    
    console.dir(fn)
    /*
    {
      length: 0,
      name: "fn",
      arguments: null,
      caller: null,
      prototype: {
        constructor: ƒ ()
        __proto__: Object
      }
      __proto__: ƒ ()
    }
    */
    
    fn.prototype.constructor === fn // true
```

可以很清楚的看到，比起别的对象，函数确实是比较特别的，他天生就带有一个 `prototype` 属性，而且 `prototype` 中的 `constructor` 就是这个函数本身。

我们也可以再验证一下最后一句话：

```js
    function Person(){}
    let me = new Person
    me.__proto__ === Person.prototype // true
```

知道了这样几个前置的知识，我们就可以往下讨论了。


# 原型链

原型链的精髓其实就是刚才已经提到过的一句话：

> `对象.__proto__ === 其构造函数.prototype`

### 普通对象的原型链

> **普通对象的原型是 Object**

这句话要从以下几点来理解：

1. 创建一个对象可以按这种方式写： `let obj = new Object({ name: 'Harvey', age: '22' })`
2. Object 实际上是一个构造函数，他构造了 obj
3. `obj.__proto__ === Object.prototype`

因此我们可以简单表示一下这个普通对象的原型链：

`obj -> Object.prototype`

## 数组的原型链

> **数组的原型是 Array**

这句话要从以下几点来理解：

1. 创建一个数组可以按这种方式写： `let arr = new Array(1, 2, 3)`
2. Array 实际上是一个构造函数，他构造了 arr
3. `arr.__proto__ === Array.prototype`

事实上，我们还会发现：

```js
    arr.__proto__.__proto__ === Object.prototype // true
    Array.prototype.__proto__ === Object.prototype // true
```

也就是说，一个数组的原型链要稍微复杂一些：

`arr -> Array.prototype -> Object.prototype`

等等，这意味着 `Object.prototype` 构造了 `Array.prototype`？这个问题我们稍后再谈。

## 函数的原型

> **函数的原型是 Function**

这句话要从以下几点来理解：

1. 创建一个数组可以按这种方式写： `let fn = new Function( (), { console.log('hi') } )`
2. Function 实际上是一个构造函数，他构造了 fn
3. `fn.__proto__ === Function.prototype`

也就是说，一个函数的原型链也要稍微复杂一些：

`fn -> Function.prototype -> Object.prototype`

## 修改原型链

通过直接修改 `__proto__` 就可以达到修改原型链的目的

```js
    let obj1 = { a: 1 }
    let obj2 = { b: 2 }
    let obj3 = { c: 3 }
    obj2.__proto__ = obj1
    obj3.__proto__ = obj2
```

这样，他们的原型链就变成了：
`obj3 -> obj2 -> obj1 -> Object.prototype`

但是这种方法是不推荐的，我们更推荐使用 `Object.create()` 方法，他的使用方法如下：

```js
    let obj1 = { a: 1 }
    let obj2 = Object.create(obj1)
    obj2.b = 2
    let obj3 = Object.create(obj2)
    obj3.c = 3
```

这样与上面直接修改 `__proto__` 效果是一样的。


# 继承

这里我们就发现，如果我们按照上面的方法修改原型链，得到的原型链 `obj3 -> obj2 -> obj1 -> Object.prototype` 有点问题！

他不满足 `对象.__proto__ === 其构造函数.prototype`！

这是当然的，因为 `obj1` 本来就不是 `obj2` 的构造函数。话说回来，`obj1` 根本就不是函数，他连 `prototype` 都没有。

所以说这里我们要明确一点，平时大家所说的继承（或者说类的继承），其实更多的是一种 **狭义的继承**。

他指的 **不是** 我们按照上面的方式 **单纯对原型链进行的修改**，**而是** 一种在 **构造函数之间** 的，有 `prototype` 存在的继承。

*PS. 尽管在某些时候，我们使用 `Object.create()` 将原型链进行简单修改也被称为继承，但不是我们这里讨论的继承*

那么问题来了，这种在 **构造函数之间的继承** 应该怎么写呢？

## 第一步：使用 `call` 来调用父类构造函数

```js
    function Person(姓名) {
      this.姓名 = 姓名
    }
    Person.prototype.自我介绍 = function() {
      console.log(`你好，我是 ${this.姓名}`)
    }
    
    function Student(姓名, 学号){
      Person.call(this, 姓名)
      this.学号 = 学号
    }
```

可以通过 `console.dir()` 来分别看看 `Person` 和 `Student`

```js
    // Person
    {
      name: "Person",
      prototype: {
        自我介绍: ƒ (),
        constructor: ƒ Person(姓名),
        __proto__: Object,
      },
      __proto__: ƒ () // Function.prototype
    }
    
    // Student
    {
      name: "Student",
      prototype: {
        constructor: ƒ Student(姓名, 学号),
        __proto__: Object,
      },
      __proto__: ƒ () // Function.prototype
    }
```

我们尝试创建一个 `Student` 对象 `小明`

```js
    let 小明 = new Student('小明', 123456)
    /*
    {
      姓名: "小明",
      学号: 123456,
    
      __proto__: {
        constructor: ƒ Student(姓名, 学号),
        __proto__: Object,
      }
    
    }
    */
    
    小明.自我介绍()
    // Uncaught TypeError: 小明.自我介绍 is not a function
```

> `小明` 如何拿到 `学号` ？

在 `new Student('小明', 123456)` 的时候，系统会去调用 `Student` 函数，并且把 `小明` 这个对象作为 `this` 传进去；

相当于在函数中执行了 `小明.学号 = 123456`。

> `小明` 如何拿到 `姓名` ？

同样，系统调用 `Student` 函数，看到了 `Person.call(this, 姓名)`（相当于 `Person.call(小明, '小明')`），意思是让 `Person` 中的 `this` 为 `小明`，并且传一个参数 `'小明'` 给 `Parent`；

然后将会调用 `Person` 函数，在 `Person` 中执行 `this.姓名 = 姓名`（相当于 `小明.姓名 = '小明'`）。

> 为什么 `小明` 不能使用 `自我介绍` ？

因为可以看到，`小明` 这个对象中没有 `自我介绍` 属性，他的 `__proto__` 中也没有，因此他找不到 `自我介绍`

## 第二步：建立原型链

```js
Student.prototype = Object.create(Person.prototype)
```

我们再打印出 `Student` 来看看

```js
    // Student
    {
      name: "Student",
      prototype: {
        __proto__:{
          自我介绍: ƒ (),
          constructor: ƒ Person(姓名),
          __proto__: Object,
        }
      },
      __proto__: ƒ () // Function
    }
```

我们发现 `prototype` 里面的 `constructor` 不见了，并且里面的 `__proto__` 变得跟 `Person.prototype` 一样了

这个时候再 `new` 一个 `小花` 看看：

```js
    let 小花 = new Student('小花', 234567)
    /*
    {
      姓名: "小花",
      学号: 234567,
      __proto__: {
    
        __proto__: {
          自我介绍: ƒ (),
          constructor: ƒ Person(姓名),
          __proto__: Object,
        }
    
      }
    }
    */
    
    小花.自我介绍()
    // '你好，我是 小花'
```

小花现在会自我介绍了，但是还有个小问题，我们发现 `小花.__proto__` 中的 `constructor` 不见了！我们得赶紧修复一下。

## 第三步：解决 constructor 的问题

```js
    Student.prototype.constructor = Student
```

看看现在 `Student` 的样子

```js
    // Student
    {
      name: "Student",
      prototype: {
        constructor: ƒ Student(姓名, 学号),
        __proto__:{
          自我介绍: ƒ (),
          constructor: ƒ Person(姓名),
          __proto__: Object,
        }
      },
      __proto__: ƒ () // Function
    }
```

最后请出我们的小红

```js
    let 小红 = new Student('小花', 345678)
    /*
    {
      姓名: "小红",
      学号: 345678,
      __proto__: {
    
        constructor: ƒ Student(姓名, 学号),
    
        __proto__: {
          自我介绍: ƒ (),
          constructor: ƒ Person(姓名),
          __proto__: Object,
        }
    
      }
    }
    */
    
    小红.自我介绍()
    // ‘你好，我是 小红’
```

OK，一切正常。

## 总结

```js
    function Person(姓名) {
      this.姓名 = 姓名
    }
    
    Person.prototype.自我介绍 = function() {
      console.log(`你好，我是 ${this.姓名}`)
    }
    
    function Student(姓名, 学号){
      Person.call(this, 姓名) // 调用父级构造函数
      this.学号 = 学号
    }
    
    Student.prototype = Object.create(Person.prototype) // 建立原型链
    
    Student.prototype.constructor = Student // 解决 constructor 问题
    /*
    {
      name: "Student",
    
      prototype: {
    
        constructor: ƒ Student(姓名, 学号),
        // Student.prototype 里面的 constructor 就是 Student
    
        __proto__:{
          // Student.prototype 里面的 __proto__ 是 Person.prototype
          自我介绍: ƒ (),
          constructor: ƒ Person(姓名),
          __proto__: Object,
        }
    
      },
    
      __proto__: ƒ () // Function
    }
    */
    
    Student.prototype.报数 = function() {
      console.log(`我的学号是 ${this.学号}`)
    }
    
    let 小红 = new Student('小红', 345678)
    /*
    {
      姓名: "小红"
      学号: 345678
      __proto__: {
    
        constructor: ƒ Student(姓名, 学号)
        // 小红 是由 Student 构造的
        // 小红.__proto__ 跟 Student.prototype 是一样的
    
        __proto__: {
          自我介绍: ƒ ()
          constructor: ƒ Person(姓名)
          __proto__: Object
        }
    
      }
    }
    */
    
    小红.自我介绍()
    小红.报数()
    // ‘你好，我是 小红’
```

### 用 class 继承

```js
    class Person {
      constructor(姓名) {
        this.姓名 = 姓名
      }
      自我介绍() {
        console.log(`你好，我是 ${this.姓名}`)
      }
    }
    
    class Student extends Person {
      constructor(姓名, 学号) {
        super(姓名) // 这里的 姓名 两个字要与父类中的一样
        this.学号 = 学号
      }
      报数() {
        console.log(`我的学号是 ${this.学号}`)
      }
    }
    
    let 小红 = new Student('小红', 345678)
    
    小红.自我介绍()
    小红.报数()
```