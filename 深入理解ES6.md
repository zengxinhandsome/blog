# 深入理解 ES6 读书笔记

## 第 1 章 块级作用域绑定

### 不能在声明变量前访问

就算用 typeof 这样安全的操作符也不行。在声明前访问块级绑定会导致错误，因为绑定还在临时死区 CTDZ ）中。

```js
typeof foo  // Error: Cannot access 'foo' before initialization
let foo = 1;
```

这样可以

```js
typeof foo  // undefined
{
  let foo = 1;
}
```

### 循环中的函数

使用var声明的变量在循环后也能访问

```js
var funcs = [];
var funcs1 = [];
var  obj = {
  a:1,
  b:1,
  c:1
}
for (var i = 0; i < 10; i++) {
  funcs.push(function () {
    console.log(i);
  });
}

for (var key in obj) {
  funcs.push(function () {
    console.log(key);
  });
}


funcs.forEach(function (func) {
  func(); // 输出十次 10，因为执行的时候循环已经结束了，此时 i 的值是 10
});


funcs1.forEach(function (func) {
  func(); // 输出三次 c，因为执行的时候循环已经结束了，此时 key 的值是 c
});
```

在es6之前要解决这个问题可以通过立即执行函数

```
var funcs = [];

for (var i = 0; i < 10; i++) {
  funcs.push(
    (function (j) {
      return function () {
        console.log(j);
      };
    })(i)
  );
}

funcs.forEach(function (func) {
  func(); // 0-9
});
```

es6只需要用let、const声明变量即可解决这个问题

### es6 之后 var关键字的使用场景

如果希望在全局对象下定义变量，仍然可以使用 var 这种情况常见于在浏览器中跨 frame 或跨 window 访问代码
