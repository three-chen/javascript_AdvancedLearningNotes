# 03let_const和Symbol类型和for-of语句

## let

let 声明的范围是块作用域， 而 var 声明的范围是函数作用域。

let 不允许同一个块作用域中出现冗余声明，这样会导致报错：

```js
let age; 
let age; // SyntaxError；标识符 age 已经声明过了
```

### 暂时性死区

let 与 var 的另一个重要的区别，就是 let 声明的变量不会在作用域中被提升。

在解析代码时，JavaScript 引擎也会注意出现在块后面的 let 声明，只不过在此之前不能以任何方 式来引用未声明的变量。在 let 声明之前的执行瞬间被称为“暂时性死区”（temporal dead zone），在此 阶段引用任何后面才声明的变量都会抛出 ReferenceError。

### 全局声明

与 var 关键字不同，使用 let 在全局作用域中声明的变量不会成为 window 对象的属性（var 声 明的变量则会）

```js
var name = 'Matt'; 
console.log(window.name); // 'Matt' 
let age = 26; 
console.log(window.age); // undefined
```

不过，let 声明仍然是在全局作用域中发生的，相应变量会在页面的生命周期内存续。因此，为了 避免 SyntaxError，必须确保页面不会重复声明同一个变量。

## const

const 的行为与 let 基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量，且尝试修改 const 声明的变量会导致运行时错误，相当于常量。

## 声明风格及最佳实践

1. 不使用 var

   有了 let 和 const，大多数开发者会发现自己不再需要 var 了。限制自己只使用 let 和 const 有助于提升代码质量，因为变量有了明确的作用域、声明位置，以及不变的值。

2. const 优先，let 次之

   使用 const 声明可以让浏览器运行时强制保持变量不变，也可以让静态代码分析工具提前发现不 合法的赋值操作。因此，很多开发者认为应该优先使用 const 来声明变量，只在提前知道未来会有修改时，再使用 let。这样可以让开发者更有信心地推断某些变量的值永远不会变，同时也能迅速发现因 意外赋值导致的非预期行为。

## Symbol类型

### 符号的基本用法

符号需要使用 Symbol()函数初始化。因为符号本身是原始类型，所以 typeof 操作符对符号返回 symbol。

```js
let sym = Symbol();
console.log(typeof sym); // symbol 
```

