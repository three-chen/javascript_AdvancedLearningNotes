[TOC]

# 13Window对象

## 计时器

### setTimeout()

单次调用，返回一个值，这个值可以传递给clearTimeout()用于取消这个函数的执行。

如果以0毫秒的超时时间来调用setTimeout()，那么指定的函数不会立刻执行。相反，会把它放到队列中，等到前面处于等待状态的事件处理程序全部执行完成后，再“立即”调用它。

### setInterval()

重复调用，返回一个值，这个值可以传递给clearInterval()用于取消后续函数的调用。

```js
var h = setInterval(f,interval);//循环调用f(),interval为毫秒间隔
//在end毫秒后停止调用，前提是end已经定义了
if (end) setTimeout(function(){ clearInterval(h); },end);
```

## 浏览器定位和导航

Window对象的location属性引用的是Location对象，它表示该窗口中当前显示的文档的URL，并定义了方法来使窗口载入新的文档。
Document对象的location属性也引用到Location对象：
`window.location === document.location // 总是返回true`

Document对象也有一个URL属性，是文档首次载入后保存该文档的URL的静态字符串如果定位到文档中的片段标识符(如#table-of-contents)，Location对象会做相应的更新，而document.URL属性却不会改变。

### 载入新的文档

#### assign()

使窗口载并显示你指定的URL中的文档。

#### replace()

类似assign()，但它在载入新文档之前会从浏览历史中把当前文档删除。如果脚本无条件地载入一个新文档，replace()方法可能是比assgin()方法更好的选择。否则，“后退”按钮会把浏览器带回到原始文档，而相同的脚本则会再次载入新文档。如果检测到用户的浏览器不支持某些特性来显示功能齐全的版本，可以用location.replace()来载入静态的HTML版本，例：

```js
//如果浏览器不支持XMLHttpRequest对象
//则将其重定向到一个不需要Aiax的静态页面
if (!XMLHttpRequest) location.replace(“staticpage.htm]l");
```

在这个例子中传入replace()的是一个相对URL。相对URL是相对于当前页面所在的目录来解析的，就像将它们用于一个超链接中。

#### reload()

让浏览器重新载入当前文档。

一种更传统的办法是直接赋值给location属性：

```js
location = "http://www.baidu.com"；
location.search = "?search=js高级";
```

## 浏览历史

Window对象的history属性引用的是该窗口的History对象。History对象是用来把窗口的浏览历史用文档和文档状态列表的形式表示。History对象的length属性表示浏览历史列表中的元素数量，但出于安全的因素，脚本不能访问已保存的URL。(如果允许，则任意脚本都可以窥探你的浏览历史。)

### back()、forward()与go()

History对象的back()和forward()方法与浏览器的“后退”和“前进”按钮一样:它们使浏览器在浏览历史中前后跳转一格。第三个方法一-g0()接受一个整数参数，可以在历史列表中向前 (正参数)或向后 (负参数)跳过任意多个页。

```js
history.go(-2);// 后退两个历史记录，相当于单击“后退”按钮两次
```

如果窗口包含多个子窗口(比如\<iframe>元素一一见14.8.2节)，子窗口的浏览历史会按时间顺序穿插在主窗口的历史中。这意味着在主窗口调用history.back()（举例）可能会导致其中一个子窗口往回跳转到前一个显示的文档，但主窗口保留当前状态不变。

## 浏览器和屏幕信息

### Navigator对象

能进行浏览器嗅探，提供运行中的浏览器的版本信息：

### Screen对象

提供有关窗口显示的大小和可用的颜色数量的信息。

## 对话框

### alert()、confirm()、prompt()

alert()向用户显示一条消息并等待用户关闭对话框。

confirm()也显示一条消息，要求用户单击“确定”或“取消”按钮，并返回一个布尔值。

prompt()同样也显示一条消息，等待用户输入字符串，并返回那个字符串。

```js
do {
  var name = prompt("what is your name?");// 得到一个字符串输入
  //得到一个布尔值
  var correct = confirm("You entered '" + name + "'.\n" + "Click Okay to proceed or Cancel to re-enter.");
} while (!correct)
alert("Hello," + name); // 输出一个纯文本消息
```

都会产生阻塞，用户体验差（alert并不一定会阻塞）。

### showModalDialog()

模态对话框，就是"显示出来后就不能点选位于其下的对话框的对话框"。

## 错误处理

Window对象的onerror属性是一个事件处理程序，当未捕获的异常传播到调用栈上时就会调用它，并把错误消息输出到浏览器的JavaScript控制台上。如果给这个属性赋一个函数，那么只要这个窗口中发生了JavaScript错误，就会调用该函数，即它成了窗口的错误处理程序。

window.onerror的第一个参数是描述错误的一条消息。第二个参数是一个字符串，它存放引发错误的JavaScript代码所在的文档的URL。第三个参数是文档中发生错误的行数。

## 多窗口和窗体

一个Web浏览器窗口可能在桌面上包含多个标签页。每一个标签页都是独立的“浏览上下文” (browsing context)，每一个上下文都有独立的Window对象，而且相互之间互不干扰。

但是窗口并不总是和其他窗口完全没关系。一个窗口或标签页中的脚本可以打开新的窗口或标签页，当一个脚本这样做时，这样多个窗口或窗口与另一个窗口的文档之间就可以互操作 （受同源策略约束）。