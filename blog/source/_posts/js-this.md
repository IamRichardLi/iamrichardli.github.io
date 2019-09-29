---
title: js-this
date: 2019-09-26 11:11:37
categories:
- JS
tags:
- 执行上下文
---
浅谈一下个人对JS的this的理解。

## 定义
先来看一下MDN上对this的[定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
>A property of an execution context (global, function or eval) that, in non–strict mode, is always a reference to an object and in strict mode can be any value.

翻译一下：this是执行上下文的一个属性，在非严格模式中始终指向一个对象，而在严格模式中this可以是任何值。

## 严格模式与非严格模式下 this的差异
在非严格模式下，若未通过`apply/call/bind`等方式改变了调用，`this`会默认指向全局对象（浏览器下指向`window`，node指向`global`）。
```
function f1() {
  return this;
}

// In a browser:
f1() === window; // true

// In Node:
f1() === global; // true
```

在严格模式下，若`this`未被执行上下文定义，那么他会始终保持为`undefined`。
```
function f2() {
  'use strict'; // see strict mode
  return this;
}

f2() === undefined; // true
```

## 经典面试题
一个经典的面试题，以下代码在浏览器执行后会打印出什么？
```
var a = 5;
var obj = {
    a : 10,
    foo: function() {
        console.log(this.a);
    }
};
var bar = obj.foo;
obj.foo();
bar();
```
首先要明确一点，`执行上下文`**是在方法执行(或者说被调用)的时候才产生的**，所以`this`也只有在方法执行时才能确定值。也就是说在声明`foo`方法的时候，`this`的值还是无法确定的。

当`obj.foo()`执行时，这个时候可以确定是`obj`调用了`foo`方法，所以`this`指向了`obj`。`foo`方法中的访问的`this.a`此时实际上就是`obj.a`，打印出的结果是`10`。

我们再往下看，当`bar()`执行时，`bar`并没有明确的调用对象，因此这里`this`使用了默认绑定，在非严格模式下指向了全局对象。(关于默认绑定后面的章节会讲)

除了用默认绑定来解释以外，我们其实还可以从另一个角度来看这个问题，已知此时`bar`就相当于`obj.foo`，`this`应该指向的是调用了`bar`方法的那个对象，可是这个时候又是谁调用了`bar`方法呢？

假如我们能以某种方式将`bar()`改写成`x.bar()`，这样的话不就和上一步的`obj.foo()`一样了吗？我们也就能得到`foo`是由`x`调用的，`this`指向的就是`x`。所以这个问题又变成了`x`是否存在？

这个神秘的`x`其实就是全局对象呀！在浏览器环境下，`x`其实就是window。因为这里的`bar`其实是一个全局变量，所以可以通过`window.bar`访问到。因此在`bar()`执行时，`this`指向了全局对象(window)，打印的值就是`window.a`，也就是`5`。

## Array中的this
再看一个Array中的this，以下代码在浏览器执行后会打印出什么？
```
function fn() {
    console.log(this);
}
var arr = [fn, 1, 2];
arr[0]();
```
是不是第一眼看到这段代码感觉有点懵逼？WTF？`this`不是只能指向一个Object的吗？来个Array是什么鬼？

乍一看Array好像跟Object没什么关系，其实抛开map、reduce、filter等等这些数组的操作方法，我们常用的对数组的访问方式无非也就是`arr[index]`和`arr.length`吧。

对于上面的这个数组`arr`来说，`arr[0]`是`fn`，`arr[1]`是`1`，`arr[2]`是`2`，`arr.length`是`3`。那我们是不是可以构造出一个类数组的对象，将数组下标作为`key`，数组元素作为对应的`value`，并且提供一个`length`属性代表`key`的总数，就像下面这样：
```
function fn() {
    console.log(this);
}
var arrLike = {
    '0': fn,
    '1': 1,
    '2': 2,
    'length': 3
};
arrLike['0']();
```
所以啊，Array是一个特殊的Object。如果用`arrLike`这个类数组的对象代入，是不是感觉清晰了一点？

已知`arrLike['0']`是`fn`，相当于以`arrLike.0()`这种形式调用了`foo`方法(只是为了方便说明，实际上不能以这种方式执行)，所以`this`指向了`arrLike`，打印出的结果就是`arrLike`本身，而对于再往上的例子来说，打印出的结果就是`arr`本身。

## this的默认绑定
如果函数没有明确的调用对象，也就是说函数是独立调用的情况下，this会使用默认绑定，非严格模式下`this`指向全局对象，严格模式下，`this`绑定到`undefined`，严格模式不允许`this`指向全局对象。

在函数中以函数作为参数传递，例如`setTimeOut`和`setInterval`等，这些函数中传递的函数中的`this`指向，在非严格模式指向的是全局对象。


