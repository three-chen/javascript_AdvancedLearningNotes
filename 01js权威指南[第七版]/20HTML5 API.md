[TOC]

# 20HTML5 API

## 地理位置

```js
navigator.geolocation.getCurrentPosition(function (pos) {
    var latitude = pos.coords.latitude;//纬度
    var longitude = pos.coords.longitude;//经度
    var accuracy = pos.coords.accuracy;//米
    console.log("your position: " + latitude + ", " + longitude + ", " + accuracy);
});
```

地理位置API是异步的：getCurrentPosition()方法和watchPosition()方法需要接受一个回调函数作为第一个参数，在判断用户的位置信息(或者当位置改变信息)时，浏览器会调用该函数。可以有第二个参数，当获取地理位置信息失败的时候调用。可以有第三个参数，为配置对象，该对象的属性指定了是否需要高精度的位置信息，该位置信息的过期时间，以及允许系统在多长时间内获取位置信息。

谷歌地图API例：

```js
//需要拿到谷歌地图API的key，前提是有外网的信用卡，有点难搞
function getmap() {
    //检查是否支持地理位置API
    if (!navigator.geolocation) throw "Geolocation not supported";

    // 创建一个新的<img元素，并开始请求地理位置信息，
    // img元素显示包含当前位置的地图，然后再将返回该图片
    var img = document.createElement("img");
    navigator.geolocation.getCurrentPosition(setMapURL);
    return img;
    function setMapURL(pos) {
        var latitude = pos.coords.latitude;
        var longitude = pos.coords.longitude;
        var accuracy = pos.coords.accuracy;

        //构造一个URL，用于请求一张显示当前位置的静态Google地图
        var url = "http://maps.google.com/maps/api/staticmap" + "?center=" + latitude + "," + longitude + "&size=640x640&sensor=true";

        //设置一个大致的缩放级别
        var zoomlevel = 20;//以各种方式开始缩放
        if (accuracy > 80)//在低精度情况下进行放大
            zoomlevel -= Math.round(Math.log(accuracy / 50) / Math.LN2);
        url += "&zoom=" + zoomlevel;//将缩放级别添加到URL中

        //现在image对象中显示该地图。感谢Google
        img.src = url;
    }
}
```

所以我换了个高德地图API，需要支付宝，这个好搞，https://lbs.amap.com/api/javascript-api/guide/abc/prepare文档链接

```js
var url = 'https://webapi.amap.com/maps?v=1.4.15&key=输你的key&callback=onLoad';
var jsapi = document.createElement('script');
jsapi.charset = 'utf-8';
jsapi.src = url;
document.head.appendChild(jsapi);
function getmap() {
    //检查是否支持地理位置API
    if (!navigator.geolocation) throw "Geolocation not supported";

    navigator.geolocation.getCurrentPosition(setMapURL);
    function setMapURL(pos) {
        var latitude = pos.coords.latitude;
        var longitude = pos.coords.longitude;
        var accuracy = pos.coords.accuracy;

        console.log(longitude, latitude);

        var map = new AMap.Map('container', {
            zoom: 11,//级别
            center: [longitude, latitude],//中心点坐标
            viewMode: '3D'//使用3D视图
        });
    }
}
window.onLoad = getmap;
```

## 历史记录管理

HTML5定义了一个相对更加复杂和强健的历史记录管理方法，该方法包含historypushstate()方法和popstate事件。当一个Web应用进入一个新的状态的时候，它会调用history.pushstate()方法将该状态添加到浏览器的浏览历史记录中。该方法的第一个参数是一个对象，该对象包含用于恢复当前文档状态所需的所有信息。该对象可以是任何能够通过JSON.stringify()方法转换成相应字符串形式的对象，也可以是其他类似Date和RegExp这样特定的本地类型 (参见下面的补充内容)。该方法的第二个可选参数是一个可选的标题 (普通的文本字符串) ，浏览器可以使用它 (比如，在一个\<Back>菜单中)来标识浏览历史记录中保存的状态。该方法的第三个参数是一个可选的URL，表示当前状态的位置。相对的URL都是以文档的当前位置为参照，通常该URL只是简单地指定URL(诸如#state)这样的hash (或者“片段标识符”)部分。将一个URL和状态关联，可以允许用户将应用的内部状态作为书签添加到浏览器中，并当在URL中包含足够信息的时候，应用可以在从书签中载入的时候就恢复它的状态。

除了pushstate()方法之外，History对象还定义了replaceState()方法，该方法和pushstate()方法接受同样的参数，但是不同的是，它不是将新的状态添加到浏览历史记录中，而是用新的状态代替当前的历史状态。

当用户通过“后退”和“前进”按钮浏览保存的历史状态时，浏览器会在Window对象上触发一个popstate事件。与该事件相关联的事件对象有一个state属性，该属性包含传递给pushstate()方法的状态对象的副本(另一个结构性复制)

## 跨域消息传递

postMessage()方法接受两个参数。其中第一个参数是要传递的消息。HTML5标准提到，该参数可以是任意基本类型值或者可以复制的对象 (实际上可能还没实现)，如果想要作为消息传递对象或者数组，首先应当使用]SON.stringify()方法对其序列化。

其中第二个参数是一个字符串，指定目标窗口的源。其中包括协议、主机名以及URL(可选的)端口部分 (可以传递一个完整的URL，但是除了协议、主机名和端口号之外的任何信息都会忽略)。这是一个安全特性：由于恶意代码或普通用户都可以在窗口中浏览新的未知文档，因此postMessage()只会将消息传递给指定的窗口，而不会传递给包含非同源文档的窗口。当然，如果传递的消息不包含任何敏感信息的话，并且愿意将其传递给任何窗口，就可以直接将该参数设置成“*”通配符即可。如果要指定和当前窗口司源的话，那么也可以简单地使用“/”

如果指定的源匹配的话，那么当调用postMessage()方法的时候，在目标窗口的Window对象上就会触发一个message事件。在目标窗口中的脚本则可以定义通知message事件的处理程序函数。调用该事件处理程序的时候会传递给它一个拥有如下属性的事件对象:

1. data：作为第一个参数传递给postMessage()方法的消息内容副本
2. source：消息源的Window对象
3. origin：一个字符串，指定消息来源

通常，onmessage()事件处理程序应当首先检测其中的origin属性，忽略来自未知源的消息。

## WebWorker

有点多线程的意思

### Worker对象

要创建一个新的Worker，只须使用worker()构造函数，并将指定在Worker中运行的JavaScript脚本的URL传递给该构造函数即可，例：

`var loader = new Worker("utils/loader.js");`

一旦获取到Worker对象后，就可以通过postMessage()方法来传递参数了。传递给postMessage()方法的值会复制，最终的副本会通过message事件传递给Worker，例：

`loader.postMessage("file.txt");`

这本书有些老了，讲的也很浅，后面我就大致浏览，不做记录，《JavaScript高级程序设计》里继续深入。

