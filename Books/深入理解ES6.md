# 深入理解 ES6 读书笔记

## 第 1 章 块级作用域绑定

### 不能在声明变量前访问

就算用 typeof 这样安全的操作符也不行。在声明前访问块级绑定会导致错误，因为绑定还在临时死区 CTDZ ）中。

```js
typeof foo; // Error: Cannot access 'foo' before initialization
let foo = 1;
```

这样可以

```js
typeof foo; // undefined
{
  let foo = 1;
}
```

### 循环中的函数

使用 var 声明的变量在循环后也能访问

```js
var funcs = [];
var funcs1 = [];
var obj = {
  a: 1,
  b: 1,
  c: 1
};
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

在 es6 之前要解决这个问题可以通过立即执行函数

```js
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

es6 只需要用 let、const 声明变量即可解决这个问题

### es6 之后 var 关键字的使用场景

如果希望在全局对象下定义变量，仍然可以使用 var 这种情况常见于在浏览器中跨 frame 或跨 window 访问代码

## 第 3 章 函数

### 函数参数默认值

当指定的函数参数不传，或者传入 `undefined` 时，都会使用函数参数的默认值

```js
const foo = (a, b = null) => {
  console.log({ a, b });
};

foo(1); // { a: 1, b: null }
foo(1, 2); // { a: 1, b: 2 }
foo(1, undefined); // { a: 1, b: null }
```

### 默认参数值对 arguments 对象的影响

切记，当使用默认参数值时， arguments 对象的行为与以往不同在 ECMAScript5 非严格模式下，函数命名参数的变化会体现在 arguments 对象中，以下这段代码解释了这种运行机制：

```js
function foo(a, b) {
  console.log(a === arguments[0]); // true
  console.log(b === arguments[1]); // true
  a = 'c';
  b = 'd';
  console.log(a === arguments[0]); // true
  console.log(b === arguments[1]); // true
}

foo('a', 'b');
```

在非严格模式下，命名参数的变化会同步更新到 arguments 对象中，所以形参 a，b 被赋予新值时，arguments 的值也同步更新了

在 ES5 严格模式下，无论参数如何变化，arguments 对象不再随之改变

```js
function foo(a, b) {
  'use strict';
  console.log(a === arguments[0]); // true
  console.log(b === arguments[1]); // true
  a = 'c';
  b = 'd';
  console.log(a === arguments[0]); // false
  console.log(b === arguments[1]); // false
}

foo('a', 'b');
```

在 ECMAScript6 中，如果一个函数使用了默认参数值，则无论是否显式定义了严格模式， arguments 对象的行为都将与 ECMAScript5 严格模式下保持一致。默认参数值的存在使得 arguments 对象保持与命名参数分离（arguments 对象只会存储你显式传入参数的值，不存储默认参数），这个微妙的细节将影响你使用 arguments 对象的方式，请看以下这段代码

```js
function foo(a, b = 'b') {
  console.log(arguments); // ['a']
  console.log(a === arguments[0]); // true
  console.log(b === arguments[1]); // false
  a = 'c';
  b = 'd';
  console.log(a === arguments[0]); // false
  console.log(b === arguments[1]); // false
}

foo('a');
```

### 默认参数表达式

可以通过函数执行来得到默认参数的值

```js
function getValue() {
  return 'b';
}

function foo(a, b = getValue()) {
  console.log({ a, b });
}

foo('a'); // { a: 'a', b: 'b' }
```

在这段代码中，如果不传入最后一个参数，则会调用 getValue 方法来得到正确的默认值。切记：函数声明的时候不会调用 getValue 方法获取默认值，只有当调用 foo 函数并且不传第二个参数的时候，才会调用 getValue 方法

正因为默认参数是在函数调用时求值，所以可以使用先定义的参数作为后定义参数的默认值，就像这样

```js
function add(a, b = a) {
  console.log(a + b);
}

add(1); // 2
```

同时也可以将第一个参数传入一个函数来获取第二个参数的默认值

```js
function getValue(value) {
  return value + 9;
}

function add(a, b = getValue(a)) {
  console.log(a + b);
}

add(1, 1); // 2
add(1); // 11
```

临时死区 TDZ：在引用参数默认值的时候，只允许引用前面参数的值，即先定义的参数不能访问后定义的参数。

上面代码相当于

```js
function add() {
  let a;
  let b = getValue(a);
}
```

### 不定参数

在 ES5，要实现下面 pick 函数功能，只能通过 arguments 对象

```js
function pick(obj) {
  const result = Object.create(null);
  for (let i = 1; i < arguments.length; i++) {
    result[arguments[i]] = obj[arguments[i]];
  }

  return result;
}

const object = {
  name: 'zhangsan',
  age: 22,
  language: 'chinaese'
};

console.log(pick(object, 'name', 'age')); // { name: 'zhangsan', age: 22 }
```

在函数的命名参数前添加三个点（...）就表示这是一个不定参数，该参数为一个数组，包含着自它之后传入的所有参数，通过这个数组名即可逐 访问里面的参数。举个例子 使用不定参数重写 pick 函数

```js
function pick(obj, ...keys) {
  const result = Object.create(null);
  for (let i = 0; i < keys.length; i++) {
    result[keys[i]] = obj[keys[i]];
  }

  return result;
}

const object = {
  name: 'zhangsan',
  age: 22,
  language: 'chinaese'
};

console.log(pick(object, 'name', 'age')); // { name: 'zhangsan', age: 22 }
```

### 增强的 Function 构造函数

Function 构造函数是 JavaScript 语法中很少被用到的 部分，通常我们用它来动态创建新的函数。

```js
const add = new Function('first', 'second', 'return first + second');

console.log(add(1, 2)); // 3
```

ECMAScript6 增强了 Function 造函数的功能，支持在创建函数时定义默认参数和不定参数。

```js
const add = new Function('first', 'second = 2', 'return first + second');

console.log(add(1)); // 3

const foo = new Function('...args', 'return args[0]');

console.log(foo(1, 2, 3, 4)); // 1
```

### 展开运算符

ES5 获取数组 max

```js
const values = [25, 50, 75, 100];

console.log(Math.max.apply(null, values));
```

ES6 展开运算符获取数组 max

```js
const values = [25, 50, 75, 100];

console.log(Math.max(...values)); // 100
// 等价于
console.log(Math.max(25, 50, 75, 100)); // 100
```

限制最小值

```javascript
const values = [25, 50, 75, 100];

console.log(Math.min(...values, 0)); // 0
```

