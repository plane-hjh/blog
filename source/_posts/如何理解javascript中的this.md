---
title: 如何理解javascript中的this
date: 2020-03-17 23:27:26
tags: javascript
category: javascript
---

关于this指向的文章，网上一搜一大把，可是很多文章都是复制粘贴的水文，是否真正的了解JavaScript中的this，那就未必。那么我们应该如何简单轻松的理解this呢？

要简单轻松的理解this，就要记住一句话：**this的指向并不是在函数定义的时候确定的，而是在其被调用的时候确定的**。也就是说，**this是在运行的时候基于函数的执行环境绑定的。**

this的指向可以分为5大类：

- 纯粹的函数调用
- 作为对象的属性调用
- 作为构造函数调用
- apply,call调用
- 箭头函数调用

### 一、 纯粹的函数调用

 纯粹的函数调用，也就是最普遍的函数调用方式。非严格模式下，this指向的是window。严格模式下，this指向了undefined。

```
// 非严格模式下
var a = 10;    // 全局变量相当于在window上面挂载属性a，window.a
function func() {
    "use strict";
    console.log(this.a);
}

func();    // 10

// 严格模式下
var b = 10;
function func1() {
    "use strict";
    console.log(this.b);
}

func1();    // Uncaught TypeError: Cannot read property 'b' of undefined
```

### 二、作为对象的属性调用

函数作为对象的属性调用时候，this 指向这个对象。（实际上纯粹的函数调用也可以算是作为对象的函数调用，因为全局变量是直接挂载在window上的，所以 func() 也就相当于 window.func() ）

```
var a = 20;
var obj = {
    a: 10,
    func: func
}

function func() {
    console.log(this.a);
}

obj.func()    // this指向了obj，所以this.a会打印出10
```

要注意的是，如果像下面这种情况，this 指向的是window的

```
var a = 20;
var obj = {
    a: 10,
    func: func
}

function func() {
    console.log(this.a);
}

var func1 = obj.func

func1()    // 20, this指向了window
```

所以说还得看函数调用的位置。

### 三、作为构造函数调用

通过构造函数，可以生成一个新对象（这个也就是我们经常说的实例）。这时，this就指这个新对象。

```
function Func() {
    this.a = 10;
}

var obj = new Func()
console.log(obj.a);    // 10
```

使用new调用构造函数实际上会经历四个过程：

1.  创建一个新对象
2. 将构造函数的作用域赋给新对象。（this指向了新对象）
3. 执行构造函数中的代码。（挂载属性）
4. 返回新的对象

如何手动实现new的功能？

```
function create() {
    // 创建一个新的对象
    var obj = new Object(),
    // 获得构造函数，arguments中去除第一个参数，Func指的是构造函数
    Func = [].shift.call(arguments);
    // 将构造函数的作用域赋给新对象。
    obj.__proto__ = Func.prototype;
    // 执行构造函数中的代码，使obj 可以访问到构造函数中的属性
    var res = Func.apply(obj, arguments);
    // 返回新的对象。判断构造函数的返回值是否是对象，如果是就返回。不是就返回新的对象。
    return res instanceof Object ? res : obj;
};

function Func() {
    this.a = 20;
}

// 使用new
var func1 = new Func()
                        
// 使用手写的new，即create
var func2 = create(Func)

console.log(func1, func2); // {a: 20}, {a: 20}
```

### 四、apply、call调用

apply、call都是是函数的一个方法，作用是改变函数的调用对象。它们的第一个参数就表示改变后的调用这个函数的对象。this 指的就是这第一个参数。

```
var obj = {
    a: 20
}
function func() {
    console.log(this.a);
}

func.call(obj);    // 20
func.apply(obj);    // 20
```

不同的是，apply的第二个参数是一个数组或者类数组对象，而call的是指定的参数列表。

### 五、箭头函数调用

箭头函数有两个作用，更简短的函数并且不绑定this。即箭头函数不会创建自己的this,它只会从自己的作用域链的上一层继承 this 。简单点来说，箭头函数的 this 会继承外层不为箭头函数的函数的 this

```
var a = 20;
function func() {
    var fun1 = () => {
        console.log(this.a)
    }
    return fun1
}

func()();    // 20，箭头函数fun1的this会从外层不为箭头函数的函数func继承而来。func的this指向了window，所以this.a是20
```
