# 12Web浏览器中的JavaScript

## 客户端JavaScript

Window对象是所有客户端JavaScript特性和API的主要接入点。它表示Web浏览器的一个窗口或窗体，用标识符window来引用。

### window对象的属性

#### location属性

```js
//设置location属性，从而跳转到新的Web页面
window.location = "http://www.baidu.com/";//窗口直接跳百度
```

#### 最重要属性document

它引用Document对象（显示在窗口中的文档），Document对象有一些重要方法，比如getElementById()等，返回的Element对象又有style和className属性。

#### onload属性

Window、Document和Element对象上另一个重要的属性集合是事件处理程序相关的属性。可以绑定一个函数，会以异步方式调用，以“on”开头：

```js
var btn = document.getElementById("mybtn");
btn.onclick = function () {
    this.innerHTML = new Date().toString();
}
```

Window对象的onload处理程序是最重要的事件处理程序之一。当显示在窗口中的文档内容稳定并可以操作时会触发它。

## 在HTML里嵌入JavaScript

四种方法

### 内联，放在\<script>\</script>之间

tips：在HTML中\<script>标签中的内容被当作其他内容一样对待。如果JavaScript代码包含了"<"或"&"，这些字符就被解释成为XML标记。因此，如果要使用XHTML，最好把所有的JavaScript代码放入CDATA里：

```
<script>
	<!CDATA[
		//你的JavaScript代码
	]>
</script>
```

只做了解，XHTML已被废弃。

### \<script>标签的src指定外部文件

### HTML中的事件处理程序

很少使用，例：

`<input type="checkbox" name="options" value="giftwrap" onchange="order.options.gitfwrap = this.checked;">`

onchange会在用户选择或取消选择复选框时执行

### URL中的JavaScript

很少使用，可以用在任何使用常规URL的地方例：

```js
<a href="javascript: alert(new Date().toLocaleTimeString());">
	what time is it?
</a>
```

## JavaScript程序的执行

Web页面中所有JavaScript代码共用同一个全局Window对象，这意味着它们都可以看到相同的Document对象，可以共享相同的全局函数和变量的集合。

如果Web页面包含一个嵌入的窗体（例如\<iframe>），此时会有不同的全局对象。

### 两个阶段

#### 阶段1：载入文档和执行脚本

载入文档内容，并执行\<script>元素里的代码。

当HTML解析器遇到\<script>元素时，它默认必须先执行脚本，然后再恢复文档的解析和渲染。这对内联脚本没什么问题，但如果脚本源代码是一个由src属性指定的外部文件，这意味着脚本后面的文档部分在下载和执行脚本之前，都不会出现在浏览器中（指文档的文本内容已经载入，但未被浏览器引擎解析为DOM树，即JavaScript代码会"阻塞"页面UI的渲染）。

#### 阶段2：事件驱动

当文档载入完成，并且所有脚本执行完成后，JavaScript执行就进入它的第二阶段。这个阶段是异步的，而且由事件驱动的。

事件驱动阶段里发生的第一个事件是load事件，指示文档已经完全载入，并可以操作。JavaScript程序经常用这个事件来触发或发送消息。我们会经常看到一些定义函数的脚本程序，除了定义一个onload事件处理程序函数外不做其他操作，这个函数会在脚本事件驱动阶段开始时被load事件触发。

### 同步、异步和延迟的脚本

JavaScript第一次添加到Web浏览器时，还没有API可以用来遍历和操作文档的结构和内容。当文档还在载入时，JavaScript影响文档内容的唯一方法时快速生成内容，使用document.write()完成上述任务。

```js
<button id="mybtn">what time</button>
  <a href="javascript:alert(new Date().toLocaleTimeString());">
    what time is it?
  </a>
<script>
	function factorial(n) {
    	if (n <= 1) return n;
    	else return n * factorial(n - 1);
	}

	document.write("<table>");
	document.write("<tr><th>n</th><th>n!</th></tr>");
	for (let i = 1; i <= 10; i++) {
    	document.write("<tr><td>" + i + "</td><td>" + factorial(i) + "</td></tr>");
	}
	document.write("</table>");
	document.write("Generated at " + new Date());
</script>
```

JavaScript的执行只在默认情况下是同步和阻塞的。\<sctipt>标签可以有defer和async属性（在支持的浏览器里）改变脚本的执行方式。这些都是布尔属性，没有值，只要出现即可，（书上说这些属性只在和src属性联合使用时才有效）。

```js
//test.js封装了上述script标签里的代码
<script defer src="./test.js"></script> ///斐波那契数列不会渲染
<script async src="./test.js"></script> ///斐波那契数列不会渲染
```

defer和async属性都像在告诉浏览器链接进来的脚本不会使用document.write()，也不会生成文档内容，因此浏览器可以在下载脚本时继续解析和渲染文档。

defer是延迟执行（当前HTML页面解析完成后执行，会按延迟脚本在文档里的出现顺序执行）

async是当前js文件加载完成后执行js（意味着可能会无序）

二者都是异步加载。

### 事件驱动的JavaScript

addEventListener()：第一个参数是事件的名称，第二个参数是调用的函数，第三个参数是布尔值，false为冒泡（向上冒气泡），true为捕获。

IE8及以前的浏览器用attachEvent()，只有前两个参数，与上面相同。

### 客户端JavaScript线程模型

JavaScript语言核心不包含任何线程机制，HTML5定义了一种作为后台线程的”WebWorker“，但客户端还是严格单线程。

Web worker是一个用来执行计算密集任务而不冻结用户界面的后台线程。运行在Web worker线程里的代码不能访问文档内容，不能和主线程或其他worker共享状态，只可以和主线程和其他worker通过异步事件进行通信，所以主线程不能检测并发性，并且Web worker不能修改JavaScript程序的基础单线程执行模型。

### 具体客户端JavaScript时间线

1. Web浏览器创建Document对象，并且开始解析Web页面，解析HTML元素和它们的文本内容后添加Element对象和Text节点到文档中。在这个阶段document.readystate属性的值是“loading”。
2. 当HTML解析器遇到**没有**async和defer属性的\<script>元素时，它把这些元素添加到文档中，然后执行行内或外部脚本。这些脚本会同步执行，并且在脚本下载(如果需要)和执行时解析器会暂停。这样脚本就可以用document.write()来把文本插入到输入流中。解析器恢复时这些文本会成为文档的一部分。同步脚本经常简单定义函数和注册后面使用的注册事件处理程序，但它们可以遍历和操作文档树，因为在它们执行时已经存在了。这样，同步脚本可以看到它自己的\<script>元素和它们之前的文档内容。
3. 当解析器遇到设置了async属性的\<script>元素时，它开始下载脚本文本，并继续解析文档。脚本会在它下载完成后尽快执行，但是解析器没有停下来等它下载。异步脚本禁止使用document.write()方法。它们可以看到自己的\<script>元素和它之前的所有文档元素，并且可能或干脆不可能访问其他的文档内容。
4. 当文档完成解析，document.readyState属性变成“interactive”。
5. 所有有defer属性的脚本，会按它们在文档的里的出现顺序执行。异步脚本可能也会在这个时间执行。延迟脚本能访问完整的文档树，禁止使用document.write()方法。
6. 浏览器在Document对象上触发DOMContentLoaded事件。这标志着程序执行从同步脚本执行阶段转换到了异步事件驱动阶段。但要注意，这时可能还有异步脚本没有执行完成.
7. 这时，文档已经完全解析完成，但是浏览器可能还在等待其他内容载入，如图片。当所有这些内容完成载入时，并且所有异步脚本完成载入和执行，document.readyState属性改变为“complete”，Web浏览器触发Window对象上的load事件
8. 从此刻起，会调用异步事件，以异步响应用户输入事件、网络事件、计时器过期等。

## 兼容性和互用性

这是一个挑战。下面提供一些网站，可能会对你有用：

https://developer.mozilla.org
	Mozilla开发者中心
http://msdn.microsoft.com
	Microsoft开发者网络
http://developer.apple.com/safari

​	Apple开发者网络里的Safari开发者中心
http://en.wikipedia.org/wiki/Comparison_of_layout_engines_(HTML_5)

​	Wikipedia文章跟踪了HTML5特性和API在各个浏览器里的实现状态

http://en.wikipedia.org/wiki/Comparison_of_layout_engines_(Document_Object_Model)

​	一篇简单的文章，跟踪DOM特性的实现状态

[Can I use... Support tables for HTML5, CSS3, etc](https://caniuse.com/)	

​	这个“何时可用·.....”站点跟踪重要Web特性的实现状态，允许根据各种	标准进行过滤，并在某个特性只剩下少量已部署的浏览器不支持时推荐	使用。

http://www.quirksmode.org/dom
	根据W3C标准列出的各种浏览器的DOM兼容性表格

http://webdevout.net/browser-support
	另一个跟踪浏览器开发商对于Web标准的实现的站点

#### 处理兼容性的策略

##### 积极策略

根据上方的文档，限制选择所有浏览器都普遍支持的特性（或者很容易模拟出）

##### 消极策略

1. 使用别人的框架或库，大（知名）框架的设计者往往会把兼容性考虑得不错。

2. ~~分级浏览器支持：是由Yahoo!率先提出的一种测试技术。从某种维度对浏览器厂商/版本/操作系统变体进行分级。分级浏览器中的A级要通过所有的功能测试用例。对于C级浏览器来说则不必所有用例都通过测试。你可以在http://developer.yahoo.com/yui/articles/gbs阅读更多关于Yahoo!的分级浏览器支持情况。~~

3. 功能测试：在脚本中添加相应的代码来检测是否在浏览器中支持该功能。例：

   ```js
   if (element.addEventListener) {//在使用这个W3C方法之前首先检测它是否可用
     element.addEventListener("keydown", handler, false);
     element.addEventListener("keypress", handler, false);
   }
   else if (element.attachEvent) {//在使用该IE方法之前首先检测它
     element.attachEvent("onkeydown", handler);
     element.attachEvent("onkeypress", handler);
   }
   else {//否则，选择普遍支持的技术
     element.onkeydown = element.onkeydown = handler;
   }
   ```

## 可访问性

盲人用户使用一种叫做屏幕阅读器的“辅助性技术”将书面的文字变成语音词汇。有些屏幕阅读器是识别JavaScript的，而另一些只能在禁用JavaScript时才会工作得更好。如果你设计的站点过于依赖JavaScript来呈现数据的话，就会把那些使用读屏软件的用户拒之门外。 JavaScript的角色应当是增加信息的表现力，而不是负责信息的呈现。JavaScript可访问性的一条重要原则是，设计的代码即使在禁用JavaScript解释器的浏览器中也能正常使用 (或至少以某种形式正常使用)。

可访问性关心的另一个重要的问题是，对于那些只使用键盘但不能(或者选择不用)使用鼠标的用户来说，如果编写的JavaScript代码依赖于特定的鼠标事件，这就会将那些不使用鼠标的用户排除在外。Web浏览器允许使用键盘来遍历和激活一个Web页面中的UI元素。并且JavaScript代码也应该允许这样做。为了考虑到可访问性，应该尽可能地支持独立于设备的事件。
创建可访问的Web页面并非鸡毛蒜皮的小问题，而对于可访问性的完整讨论则超出了本书的范畴。关心可访问性的Web应用开发者应该阅读这里的文档: http://www.w3.org/WAl/intro/aria（404了已经）的WAI-ARIA (Web Accessibility Initiative-Accessible Rich InternetApplications)标准。

## 安全性

### 同源策略

脚本不能读取从不同服务器（域、端口或协议）载入的文档的内容，除非这个文档就是包含该脚本的文档。类似地，一个脚本不能在来自不同服务器的文档上注册事件监听器。这就防止脚本窃取其他页面的用户输入 (例如，组成一个密码项的键盘单击过程)。这一限制叫做同源策略 。