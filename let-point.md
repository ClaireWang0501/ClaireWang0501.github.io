---
layout: default
---

# let声明需要注意的point
> Written May 6, 2018. Tagged **Javascript**, **ES6**, **let**.

```
1.  let只作用于块级作用域（需注意for循环时let内部是如何操作的）
2.  let不存在hoist（需注意暂时性死区）
3.  同一块级作用域内let不能重复声明（需注意ios10的bug）
4.  在全局中声明let变量，不属于顶层属性
```

### let只作用于块级作用域

在ES5中作用域分为全局作用域和函数作用域，**_var_**与**function**的声明均在其所属的所用域中有效。
而ES6添加了块级作用域，使得对于_let_与const声明，其在块级作用域中有效。块级作用域以{}进行标识。

```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

需要注意的是，在for循环中let声明的计数器是只在本轮循环中有效的。而var则会声明到所属的函数作用域。

```js
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

上方的代码块中，计数器i是var声明，因此其所用域为全局作用域。经过for循环后，i变为10，因此最后的输出并不如我们预想的那样输出为6，而是10。而下方的代码块中，i使用了let声明，因此console.log的对象为每轮循环中对应的变量i，因此输出符合我们的预期。

```js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

可能有人会有疑问，如果使用let计数器只在本轮有效，那么每轮的i是如何叠加的呢？这是因为JS引擎在每轮循环时会记住上一轮i的值，并在此基础上计算本轮的i。同时，还需注意的是设置计数器的区域是一个父作用域，循环体内部是一个子作用域。

```js
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

因此在上方的代码块执行时并不会发生重复定义的error。