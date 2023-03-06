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

   使用 const 声明可以让浏览器运行时强制保持变量不变，也可以让静态代码分析工具提前发现不合法的赋值操作。因此，很多开发者认为应该优先使用 const 来声明变量，只在提前知道未来会有修改时，再使用 let。这样可以让开发者更有信心地推断某些变量的值永远不会变，同时也能迅速发现因 意外赋值导致的非预期行为。
   
   tips：由于 const 声明暗示变量的值是单一类型且不可修改，JavaScript 运行时编译器可以将其所有实例 都替换成实际的值，而不会通过查询表进行变量查找。谷歌的 V8 引擎就执行这种优化。

## Symbol类型

Symbol（符号）是 ECMAScript 6 新增的数据类型。符号是原始值，且符号实例是唯一、不可变的。 符号的用途是确保对象属性使用唯一标识符，不会发生属性冲突的危险。

### 符号的基本用法

符号需要使用 Symbol()函数初始化。因为符号本身是原始类型，所以 typeof 操作符对符号返回 symbol。

```js
let sym = Symbol();
console.log(typeof sym); // symbol 
```

调用 Symbol()函数时，也可以传入一个字符串参数作为对符号的描述（description），将来可以通 过这个字符串来调试代码。但是，这个字符串参数与符号定义或标识完全无关：

```js
let genericSymbol = Symbol();
let otherGenericSymbol = Symbol(); 
let fooSymbol = Symbol('foo');
let otherFooSymbol = Symbol('foo'); 
console.log(genericSymbol == otherGenericSymbol); // false
console.log(fooSymbol == otherFooSymbol); // false
```

符号没有字面量语法，这也是它们发挥作用的关键。按照规范，你只要创建 Symbol()实例并将其 用作对象的新属性，就可以保证它不会覆盖已有的对象属性，无论是符号属性还是字符串属性。

```js
let genericSymbol = Symbol(); 
console.log(genericSymbol); // Symbol() 
let fooSymbol = Symbol('foo'); 
console.log(fooSymbol); // Symbol(foo);
```

### 使用全局符号注册表

如果运行时的不同部分需要共享和重用符号实例，那么可以用一个字符串作为键，在全局符号注册 表中创建并重用符号。 为此，需要使用 Symbol.for()方法：

```js
let fooGlobalSymbol = Symbol.for('foo'); 
console.log(typeof fooGlobalSymbol); // symbol
```

Symbol.for()对每个字符串键都执行幂等操作。第一次使用某个字符串调用时，它会检查全局运 行时注册表，发现不存在对应的符号，于是就会生成一个新符号实例并添加到注册表中。后续使用相同 字符串的调用同样会检查注册表，发现存在与该字符串对应的符号，然后就会返回该符号实例。

```js
let fooGlobalSymbol = Symbol.for('foo'); // 创建新符号
let otherFooGlobalSymbol = Symbol.for('foo'); // 重用已有符号
console.log(fooGlobalSymbol === otherFooGlobalSymbol); // true
```

即使采用相同的符号描述，在全局注册表中定义的符号跟使用 Symbol()定义的符号也并不等同：

```js
let localSymbol = Symbol('foo'); 
let globalSymbol = Symbol.for('foo'); 
console.log(localSymbol === globalSymbol); // false
```

以使用 Symbol.keyFor()来查询全局注册表，这个方法接收符号，返回该全局符号对应的字 符串键。如果查询的不是全局符号，则返回 undefined。

```js
// 创建全局符号 let s = Symbol.for('foo'); 
console.log(Symbol.keyFor(s)); // foo 

// 创建普通符号 let s2 = Symbol('bar'); 
console.log(Symbol.keyFor(s2)); // undefined
```

如果传给 Symbol.keyFor()的不是符号，则该方法抛出 TypeError：

```js
Symbol.keyFor(123); // TypeError: 123 is not a symbol
```

### 使用符号作为属性

凡是可以使用字符串或数值作为属性的地方，都可以使用符号。这就包括了对象字面量属性和 Object.defineProperty()/Object.defineProperties()定义的属性。对象字面量只能在计算属 性语法中使用符号作为属性。

```js
let s1 = Symbol('foo'), 
 s2 = Symbol('bar'), 
 s3 = Symbol('baz'), 
 s4 = Symbol('qux'); 
let o = { 
 [s1]: 'foo val' 
}; 
// 这样也可以：o[s1] = 'foo val'; 
console.log(o); // {Symbol(foo): foo val} 
Object.defineProperty(o, s2, {value: 'bar val'}); 
console.log(o); // {Symbol(foo): foo val, Symbol(bar): bar val} 
Object.defineProperties(o, { 
 [s3]: {value: 'baz val'}, 
 [s4]: {value: 'qux val'} 
}); 
console.log(o); 
// {Symbol(foo): foo val, Symbol(bar): bar val, 
// Symbol(baz): baz val, Symbol(qux): qux val} 
```

类似于 Object.getOwnPropertyNames()返回对象实例的常规属性数组，Object.getOwnPropertySymbols()返回对象实例的符号属性数组。这两个方法的返回值彼此互斥。Object.getOwnPropertyDescriptors()会返回同时包含常规和符号属性描述符的对象。Reflect.ownKeys()会返回两种类型的键：

```js
let s1 = Symbol('foo'), 
 s2 = Symbol('bar'); 
let o = { 
 [s1]: 'foo val', 
 [s2]: 'bar val', 
 baz: 'baz val', 
 qux: 'qux val' 
}; 
console.log(Object.getOwnPropertySymbols(o)); 
// [Symbol(foo), Symbol(bar)] 
console.log(Object.getOwnPropertyNames(o)); 
// ["baz", "qux"] 
console.log(Object.getOwnPropertyDescriptors(o)); 
// {baz: {...}, qux: {...}, Symbol(foo): {...}, Symbol(bar): {...}} 
console.log(Reflect.ownKeys(o)); 
// ["baz", "qux", Symbol(foo), Symbol(bar)] 
```

### 常用内置符号

ECMAScript 6 也引入了一批常用内置符号（well-known symbol），用于暴露语言内部行为，开发者 可以直接访问、重写或模拟这些行为。这些内置符号都以 Symbol 工厂函数字符串属性的形式存在。 这些内置符号最重要的用途之一是重新定义它们，从而改变原生结构的行为。比如，我们知道 for-of 循环会在相关对象上使用 Symbol.iterator 属性，那么就可以通过在自定义对象上重新定义 Symbol.iterator 的值，来改变 for-of 在迭代该对象时的行为。 这些内置符号也没有什么特别之处，它们就是全局函数 Symbol 的普通字符串属性，指向一个符号的实例。所有内置符号属性都是不可写、不可枚举、不可配置的。

### Symbol.asyncIterator（异步遍历器）

根据 ECMAScript 规范，这个符号作为一个属性表示“一个方法，该方法返回对象默认的 AsyncIterator。 由 for-await-of 语句使用”。换句话说，这个符号表示实现异步迭代器 API 的函数。 for-await-of 循环会利用这个函数执行异步迭代操作。循环时，它们会调用以 Symbol.asyncIterator 为键的函数，并期望这个函数会返回一个实现迭代器 API 的对象。很多时候，返回的对象是实现该 API 的 AsyncGenerator：

```js
class Foo { 
 async *[Symbol.asyncIterator]() {} 
} 
let f = new Foo(); 
console.log(f[Symbol.asyncIterator]()); 
// AsyncGenerator {<suspended>} 
```

技术上，这个由 Symbol.asyncIterator 函数生成的对象应该通过其 next()方法陆续返回 Promise 实例。可以通过显式地调用 next()方法返回，也可以隐式地通过异步生成器函数返回：

```js
class Emitter { 
 constructor(max) { 
 this.max = max; 
 this.asyncIdx = 0; 
 } 
 async *[Symbol.asyncIterator]() { 
 while(this.asyncIdx < this.max) { 
 yield new Promise((resolve) => resolve(this.asyncIdx++)); 
 } 
 } 
} 
async function asyncCount() { 
 let emitter = new Emitter(5); 
 for await(const x of emitter) { 
 console.log(x); 
 } 
} 
asyncCount(); 
// 0
// 1
// 2
// 3
// 4
```

## for-of

for-of 语句是一种严格的迭代语句，用于遍历可迭代对象的元素，语法如下：

`for (property of expression) statement`，具体例子：

```js
for (const el of [2,4,6,8]) { 
 document.write(el); 
}
```

for-of 循环会按照可迭代对象的 next()方法产生值的顺序迭代元素。
