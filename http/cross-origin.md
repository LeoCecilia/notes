## document.domain
`documnet.domain`常用于同一主域，不同子域的跨域请求，比如**foo.com**和**img.foo.com**之间，只要把这两个页面都指向其主域，既可以实现跨域通信，然后通过`ifr.contentWindow`就可访问子页面的window，子页面可以通过访问`parent.window`或`parent`访问父页面的window

``` html
<!-- foo.com/a.html -->
<iframe id="ifr" src="http://img.foo.com/b.html"></iframe>
<script>
document.domain = 'foo.com';
function aa(str) {
    console.log(str);
}
window.onload = function () {
    document.querySelector('#ifr').contentWindow.bb('aaa');
}
</script>
```

``` html
<!-- img.foo.com/b.html -->

<script>
document.domain = 'foo.com';
function bb(str) {
    console.log(str);
}

parent.aa('bbb');
</script>
```

## window.name
只要不关闭浏览器，既可以实现跨域通域通信
实践一下，你在qq.com的控制台写下`window.name="asdf";`，然后在地址栏上输入baidu.com，打开控制台，输入`window.name;//asdf`

这个例子不太理解哎
``` html
<!-- foo.com/a.html -->
<iframe id="ifr" src="http://bar.com/b.html"></iframe>
<script>
function callback(data){
  console.log(data);
}
</script>
```

``` html
<!-- bar.com/b.html -->
<input id="txt" type="text">
<input type="button" value="发送" onclick="send();">


<script>
var proxyA = 'http://foo.com/aa.html';    // foo.com下代理页面
var proxyB = 'http://bar.com/bb.html';    // bar.com下代理空页面

var ifr = document.createElement('iframe');
ifr.style.display = 'none';
document.body.appendChild(ifr);

function send() {
    ifr.src = proxyB;
}

ifr.onload = function() {
    ifr.contentWindow.name = document.querySelector('#txt').value;
    ifr.src = proxyA;
}
</script>
```
``` html
<!-- foo.com/aa.html -->
top.callback(window.name);//window.top 返回最顶层的窗口对象
```

## location.hash
把传递的数据依附在url上,数据为对象的形式，然后再使用`JSON.parse()`来进行转换
``` html
<!-- foo.com/a.html-->
<iframe id="ifr" src="http://bar.com/b.html"></iframe>
<script>
function callback(data) {
    console.log(data)
}
</script>
```
``` html
<!-- bar.com/b.html -->
<script>
window.onload = function() {
    var ifr = document.createElement('iframe');
    ifr.style.display = 'none';
    var height = document.documentElement.scrollHeight;
    var data = '{"h":'+ height+', "json": {"a":1,"b":2}}';
    ifr.src = 'http://foo.com/aa.html#' + data;
    document.body.appendChild(ifr);
}
</script>
```
``` html
<!-- foo.com/aa.html -->
var data = JSON.parse(location.hash.substr(1));
top.document.getElementById('ifr').style.height = data.h + 'px';
top.callback(data);
```

## window.navigator
IE6的API

## postMessage
H5,IE8支持
`window.postMessage()` 方法被调用时，会在所有页面脚本执行完毕之后（e.g., 在该方法之后设置的事件、之前设置的`timeo`ut 事件,etc.）向目标窗口派发一个  `MessageEvent` 消息。 该`MessageEvent`消息有四个属性需要注意： `message` 属性表示该`message` 的类型； `data` 属性为 `window.postMessag`e 的第一个参数；origin 属性表示调用`window.postMessage()`方法时调用页面的当前状态； source 属性记录调用 `window.postMessage()` 方法的窗口信息。
``` javascript
// URL: http://a.com/foo
var win = window.open('http://b.com/bar');
win.postMessage('Hello, bar!', 'http://b.com');
```
``` javscript
window.addEventListener('message',function(event) {
    console.log(event.data);
});
```
