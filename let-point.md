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

### 1. let只作用于块级作用域

在ES5中作用域分为全局作用域和函数作用域，_var_ 与 _function_ 的声明均在其所属的所用域中有效。
而ES6添加了块级作用域，使得对于 _let_ 与 _const_ 声明，其在块级作用域中有效。块级作用域以 _{}_ 进行标识。

```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

需要注意的是，在 _for_ 循环中 _let_ 声明的计数器是只在本轮循环中有效的。而 _var_ 则会声明到所属的函数作用域。

```js
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

上方的代码块中，计数器 _i_ 是 _var_ 声明，因此其所用域为全局作用域。经过 _for_ 循环后，_i_ 变为10，因此最后的输出并不如我们预想的那样输出为6，而是10。而下方的代码块中，_i_ 使用了 _let_ 声明，因此 _console.log_ 的对象为每轮循环中对应的变量 _i_ ，因此输出符合我们的预期。

```js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

可能有人会有疑问，如果使用 _let_ 计数器只在本轮有效，那么每轮的 _i_ 是如何叠加的呢？这是因为JS引擎在每轮循环时会记住上一轮 _i_ 的值，并在此基础上计算本轮的 _i_ 。同时，还需注意的是设置计数器的区域是一个父作用域，循环体内部是一个子作用域。

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

### 2. let不存在hoist

对于 _var_ 和 _function_ 声明，存在着hoist（变量提升现象）。变量提升指的是在执行JS语句之前，会将 _var_ 与 _function_ 的变量在最顶部声明为 _undefined_ ，并在真正的声明语句位置对其进行赋值。

```js
// var 的情况
// => var foo = undefined;
console.log(foo); // 输出undefined
var foo = 2; // => foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

如代码块中上半部分 _var_ 变量的声明，在 _console.log_ 时会输出 _undefined_ 。而 _let_ 与 _const_ 声明则不会有hoist现象，因此若在声明前滴啊用，则会报错 _ReferenceError_ 。

需要注意的是，若在块级作用域中声明了 _let_ 或 _const_ 变量，就意味着该变量绑定了这个区域。其父作用域的同名变量并不会传入该作用域，是的声明之前对该变量的调用会抛出error。

```js
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

该变量声明前的部分区域称为暂时性死区（Temporal Dead Zone）。