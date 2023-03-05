[TOC]

# 15脚本化CSS

具体去菜鸟或MDN看吧，这里给出一些我没见过的。

## text-shadow

text-shadow在文本下产生阴影效果。（当然也可以用定位实现，只是麻烦些）

```html
<!-- text-shadow属性自动产生阴影效果-->
<span style="text-shadow: 3px 3px 1px #888">Shadowed</span>
```

## 查询计算出的样式

用浏览器窗口对象的getComputedStyle()方法来获得一个元素的计算样式。此方法的第一个参数就是要获取其计算样式的元素，第二个参数也是必需的，通常是null或空字符串，但它也可以是命名CSS伪对象的字符串，如“:before”、”:after”、“:first-line“、“:first-letter”。

```js
var title = document.getElementById("section1title");
var titlestyles = window.getComputedStyle(element,null);
```

getComputedStyle()方法的返回值是一个CSSStyleDeclaration对象，它代表了应用在指定元素(或伪对象)上的所有样式。表示计算样式的CSSStyleDeclaration对象和表示内联样式的对象之间有一些重要的区别:

1. 计算样式的属性是只读的。
2. 计算样式的值是绝对值:类似百分比和点之类相对的单位将全部转换为绝对值。所有指定尺寸(例如外边距大小和字体大小)的属性都有一个以像素为度量单位的值。该值将是一个冠以“px”后缀的字符串，使用时仍然需要解析它，但是不用担心单位的解析或转换。其值是颜色的属性将以“rgb(#,#,#)”或“rgba(#,#,#,#)”的格式返回。
3. 不计算复合属性，它们只基于最基础的属性。例如，不要查询margin属性，应该使用marginLeft和marginTop等。
4. 计算样式的cssText属性未定义

## 脚本化样式表

### 介绍

这种方法直接修改样式省去了查询DOM元素，效率更高。

在脚本化样式表时，将会碰到两类需要使用的对象。第一类是元素对象，第二类是CSSStyleSheet对象，它表示样式表本身。document.styleSheets属性是一个只读的类数组对象，它包含CSSStyleSheet对象，表示与文档关联在一起的样式表。如果为定义或引用了样式表的\<style>或\<link>元素设置title属性值，该title作为对应CSSStyleSheet对象的title属性就可用。

### 开启和关闭样式表

\<style>、\<link>元素和CSSStyleSheet对象都定义了一个在JavaScript中可以设置和查询的disabled属性。顾名思义，如果disabled属性为true，样式表就被浏览器关闭并忽略。

以下disableStyleSheet()函数说明这一点。如果传递一个数字，函数将其当做document.styleSheets数组中的一个索引，如果传递一个字符串，函数将其当做CSS选择器并传递给document.querySelectorAll()，然后设置所有返回元素的disabled属性：

```js
function disableStylesheet(ss) {
	return function () {
		if (typeof ss === "number")
			document.styleSheets[ss].disabled = true;
		else {
			var sheets = document.querySelectorAll(ss);
			for (var i = 0; i < sheets.length; i++)
				sheets[i].disabled = true;
		}
	}
}
//页面会不显示document.styleSheets[0].cssRules的所有样式，内联样式还生效
setTimeout(disableStylesheet(0), 1000);
```

### 查询、插入与删除样式表规则

CSSStyleSheet对象有一个cssRules[]数组，它包含样式表的所有规则:
`var firstRule = document.styleSheets[o].cssRules[o];`

除了查询和修改样式表中已存在的规则以外，也能向样式表添加和从中删除规则。标准的API接口定义了insertRule()和deleteRule()方法来添加和删除规则：

```js
//第一个参数是样式，第二个参数是索引
document.styleSheets[0].insertRule("H1 { color: red; ", 0);
```

### 创建新样式表

就是新增一个\<style>标签

```js
style = document.createElement("style");
document.head.appendChild(style);
```

