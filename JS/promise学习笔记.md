# 什么是promise
promise与异步编程有关，异步编程一般是之js的事件机制，setTimeout，setInterval以及回调函数。然后基于js的工作原理可知，JS是单线程工作，所以使用回调的时候，很容易造成回调地狱的状况。

* 这就会导致代码的可读性很差，如果请求的数据有100多种，那真的是地狱。
* 并行逻辑必须串行执行（但其实数据可以不用串行传输的）

而promise就是为了解决这个问题而生

# promise的语法

``` javascript
new Promise(function(resolve, reject) {
    //待处理的异步逻辑
    //处理结束后，调用resolve或reject方法
})
```

# promise的状态
* pending(初始化状态)
* rejected(异步任务失败)
* fulfilled(异步任务成功)

`pending`状态的promise可能以`fulfilled` 返回一个值，也可能因某种原因被`rejected`，当上文中的任一种情况发生，then函数绑定的handler就会被调用，并且提供`reject`,`resolve`的回调函数

一图胜千言

![图片](../img/promises.png)

# promise机制
``` javascript
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();
    function handler() {
      if (this.status === 200) { 
              resolve(this.response); 
          } else { 
              reject(new Error(this.statusText)); 
          }
    };
  });
  return promise;
};
getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```
上面代码中，resolve方法和reject方法调用时，都带有参数。它们的参数会被传递给回调函数。reject方法的参数通常是Error对象的实例，而resolve方法的参数除了正常的值以外，还可能是另一个Promise实例，比如像下面这样。

``` javascript
var p1 = new Promise(function(resolve, reject){
  // ... some code
});
var p2 = new Promise(function(resolve, reject){
  // ... some code
  resolve(p1);
});
```

上面代码中，p1和p2都是Promise的实例，但是p2的resolve方法将p1作为参数，这时p1的状态就会传递给p2。如果调用的时候，p1的状态是pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是fulfilled或者rejected，那么p2的回调函数将会立刻执行。

# promise的链式操作
``` javascript
getJSON("/visa.json").then(function(json) {
  return json.name;
}).then(function(name) {
  // proceed
});
```
* 上面的代码中前一个回调函数的返回结果会作为参数传递给后一个函数
* 如果前一个回调函数的返回结果为promise实例，则后一个回调函数要等待该实例执行完成后才能被调用

# promise捕获错误
``` javascript
getJSON("/visa.json").then(function(result) {
  // some code
}).catch(function(error) {
  // 处理前一个回调函数运行时发生的错误
  console.log('出错啦！', error);
});
```
相当于

``` javascript
getJSON("/visa.json").then(function(result) {
  // some code
},function(error) {
  // 处理前一个回调函数运行时发生的错误
  console.log('出错啦！', error);
});
```
promise对象错误具有冒泡的性质，当前的错误会被下一个catch所捕捉

# 常用的promise函数
- Promise.all([p1,p2,p3])
- Promise.race([p1,p2,p3])

这两个函数都是应用在多个`promise`实例的
`Promise.all`有两种情况

1. 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

2. 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

`var p = Promise.race`则是只要p1,p2,p3任一个状态发生变化，p也会跟着变化，而且即便其他实例的状态也跟着改变，p的状态也不再跟着改变

# Promise.resolve()和Promise.reject()函数
这两个函数可将参数转换为promise实例，并且resolve()对应的是fulfilled状态，reject对应的是fulfilled状态

``` javascript
var p = Promise.reject('出错啦');
p.then(null, function (error){
  console.log(error)
});
// 出错了
```






