# 10JavaScript的子集和扩展

由于书写的时间很早，所以扩展之类的省略。

## JavaScript子集

### 安全子集

可用来静态检查。

ADsafe：包含静态检查，使用JSLint（http://jslint.org）作为检验器。

dojox.secure：比ADsafe多了使用标准DOM API。

Caja：Google发布的安全子集。

FBJS：被Facebook采用。

Microsoft Web Sandbox：有效地重新实现了一个安全的JavaScript虚拟机。

## 解构赋值

`let [x,y] = [1,2];//等价于 let x=1,y=2`

## E4X:ECMAScript for XML

E4X将XML文档视为一个XML对象，将XML片段(在常见的父对象中包含多个XML元素)视为一个紧密相关的XML列表对象。

XML对象和DOM(文档对象模型)对象没有任何关系，理解这一点非常重要。下面会给出E4X的一个快速入门教程，而不会进行更深入的讲解。

### 入门

E4X只定义了很少的新语言语法。最显著的新语法当属将XML标签引入JavaScript语言中。可以在JavaScript代码中直接书写XML标签直接量，比如:

```
//创建一个XML对象
var pt =
<periodictable>
	<element id="1"><name>Hydrogen</name></element>
	<element id="2"><name>Helium</name></element>
	<element id="3"><name>Lithium</name></element></periodictable>;
//给这个表格添加一个新元素
pt.element +=<element id="4"><name>Beryllium</name></element>;
```

XML直接量语法中使用花括号作为转义字符，可以在XML中嵌人JavaScript表达式。例如，这里是另外一种创建XML元素的方法:

```
pt = <periodictable></periodictable>;//创建一个空表格
var elements = ["Hydrogen"，"Helium"，nLithium"];//待添加的元素
//使用数组元素创建XML标签
for(var n = 0;n < elements.length; n++) {
	pt.element += <element id={n+1}><name>{elements[n]}</name></element>;
}
```

除了使用直接量语法，我们也可以将字符串解析成XML。下面的代码为上段代码创建的节点增加了一个新元素:

```
pt.element += new XML('<element id="5"><name>Boron</name></element>');
```

当涉及XML片段的时候，使用XMLList()替换XML():

```
pt.element += new XMLList('<element id="6"><name>Carbon</name></element>' + '<element id="7"><name>Nitrogen</name></element>');
```

E4X提供了一些显而易见的语法用以访问所创建的XML文档的内容：

```
var elements = pt.element;//得到所有<element>标签组成的一个列表
var names = pt.element.name;//得到所有的<name>标签的一个列表
var n = names[o];//"Hydrogen”(氢)，name的第0个标签的内容
```

E4X同样为操作XML对象提供了语法支持，点点 (..) 运算符是“后代运算符”(descendant operator) ，可以用它替换普通的点 (.)成员访问运算符

```
//另一种得到所有<name>标签对应列表的方法

var names2 = pt..name;
```

E4X甚至定义了通配符运算:

```
//得到所有<element>标签的所有子节点
//这也是得到所有<name>标签对应列表的另外一种方法
var names3 = pt.element.*;//*可以理解为全部
```

E4X中使用字符@来区分属性名和标签名 (从XPath中借用过来的语法)。比如，可以这样来获得一个属性;

```
//"氮"的原子序数是多少
var atomicNumber = pt.element[1].@id;
```

可以使用通配符来获得属性名@*:

```
//获得所有的<element>标签的所有属性
var atomicNums = pt.element.@*;
```

E4X甚至包含了一种强大且极其简洁的语法用来对列表进行过滤，过滤条件可以是任意谓词表达式：

```
//对所有的<element>元素组成的]一个列表进行过滤
//过滤出那些id属性小于3的元素

var lightElements = pt.element.(@id < 3);

//对所有的element元素组成的列表进行过滤
//过滤出那些name以B开始的元素
//然后得到过滤后元素的<name>标签列表
var bElementNames = pt.element.(name.charAt(0) == 'B').name;
```

E4X标准可以用for/each来遍历XML标签和属性列表。for/each和for/in循环非常类似，for/in循环用以遍历对象的属性名，for/each循环用以遍历对象的属性值:

```
//输出元素周期表中的每个元素名
for each (var e in pt.element){
	console.log(e.name);
}
//输出每个元素的原子序数
for each (var n in pt.element.@*) console.log(n);
```

E4X表达式可以出现在赋值语句的左侧，可以用它来对已存在的标签和属性进行修改或添加新标签或属性:

```
//修改氢元素的<element>标签，给它添加一个新属性
//像下面这样添加一个子元素
//<element id="1" symbol="H">
//	<name>Hydrogen</name>
//	<weight>1.00794</weight>
//</element>

pt.element[0].@symbol ="H";
pt.element[0].weight = 1.00794;
```

通过标准的delete运算符也可以方便地删除属性和标签:

```
delete pt.element[0].@symbol; // 删除一个属性
delete pt..weight;//删除所有的<weight>标签
```

我们可以通过E4X所提供的语法来进行大部分的XML操作。E4X同样定义了能够调用XML对象的方法，例如，这里用到了insertChildBefore()方法:

```
pt.insertChildBefore(pt.element[1], <element id="1"><name>Deuterium</name></element>);
```

E4X中是完全支持命名空间的，它为使用XML命名空间提供了语法支持和API支持:

```
//声明默认的命名空间
default xml namespace = "http://www.w3.org/1999/xhtml";
// 这里也是一个包含一些svg标签的xhtm1文档
d = <html>
		<body>
		This is a small red square:
		<svg xmlns="http://www.w3.org/2000/svg" width="10" height="10">
			<rect x="o" y="o" width="10" height="10" fll="red"/>
		</svg>
		</body>
	</html>
	
// body元素和它的命名空间里的uri及其localName
var tagname = d.body.name();
var bodyns = tagname.uri;
var localname = tagname.localName;

//选择<svg>元素需要多做一些工作，因为<svg>不在默认的命名空间中
//因此需要为svg创建一个命名空间，并使用::运算符将命名空间添加至标签名中
var svg = new Namespace('http://www.w3.org/2000/svg');
var color = d..svg::rect.@fill;//"red"
```

