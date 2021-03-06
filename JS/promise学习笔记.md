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

``` javascript
promise.then(
  onFulfilled?: Function,
  onRejected?: Function
) => Promise
```

.then()方法遵循一下规则：

* onFulfilled() 和 onRejected() 都是可选的。
如果提供的参数不是函数，必须忽略参数。

* onFulfilled() 会在 promise 完成后调用，把 promise 的值作为第一个参数。

* onRejected() 会在 promise 拒绝后调用，把拒绝的原因作为第一个参数。这个原因可以是任何可用的 JavaScript 值，不过因为拒绝本质上是异常，我推荐使用 Error 对象。

* onFulfilled() 和 onRejected() 只会调用一次。
.then() 必须返回一个新的 promise，promise2。
如果 onFulfilled() 或 onRejected() 返回一个值 x，并且 x 是个 promise，那么 promise2就会等待x状态发生改变，然后promise2的状态和值就会与 x 相同。

* 否则，promise2 会成为已完成状态，x 会作为参数传递给promise2。

* 如果 onFulfilled 或 onRejected 抛出一个异常 e，promise2 会被拒绝，原因是 e。
如果 onFulfilled 不是函数，且 promise1 已完成，那么 promise2 必须是已完成，值和 promise1 的相同。

* 如果 onRejected 不是函数，且 promise1 已拒绝，那么 promise2 必须是已拒绝，原因和 promise1 的相同。
* then中的函数体不是promise，所以如果想要串行传输的数据，而不引起回调地狱的，应在函数体当中返回promise。

手写promise的例子

``` javascript
function ajax(url,type){
    return new Promise((resolve,reject) => {
        var XHR = new XHRHttpRequest();
        XHR.open(type,url,true);
        XHR.send();
        XHR.onreadystatechange = () => {
            if(XHR.readyState == 4) {
                if(XHR.status == 200) {
                    //这里可以不写的，可以直接使用.catch函数来捕获其异常
                    try {
                        let response = JSON.parse(XHR.responseText);
                        resolve(response);
                    } catch(e) {
                        reject(e);
                    }
                } else {
                    reject(new Error(XHR.statusText));
                }
            }
        }
    });
}
```


# promise捕获错误
``` javascript
getJSON("/visa.json").then(handleSuccess)
.catch(function(error) {
  // 处理前一个回调函数运行时发生的错误
  console.log('出错啦！', error);
});
```
相当于

``` javascript
getJSON("/visa.json").then(handleSuccess)
.catch(function () {

});
```
两种的差别很微妙，也很重要

* 在第一个例子中getJSON的错误会被处理，但是handleSuccess会被吞掉

* catch能同时处理getJSON和handleSuccess的错误
  ``` javascript
  getJSON()
    .then(
      handleSuccess,
      handleNetworkError
    )
    .catch(handleProgrammerError)
  ;
  ```

* 在then中的promise，要主动地去catch错误，因为后续的catch是不会捕获该promise的错误的。可采用如下方式去捕获

  ``` JavaScript
  return new Promise(resolve,reject)=>{
    try{
      if(){
        resolve();
      }
      else{
        reject();
      }
    }catch(e){
      reject(new Error(e));
    }
  }
  ```




promise对象错误具有冒泡的性质，当前的错误会被下一个catch所捕捉



# 常用的promise函数
- Promise.all([p1,p2,p3])
- Promise.race([p1,p2,p3])

这两个函数都是应用在多个`promise`实例的
`Promise.all`有两种情况

1. 只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

2. 只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

3. 适用场景：数组中的promise对象**不**需要**串行**执行

4. 需要串行执行时，解决方法
  - 使用`Math.reduce`让其串行执行

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

# 如何取消一个promise

**思路** :解决这个promise，并使用‘cancelled’作为原因，如果你像区分普通错误。就在错误处理函数添加分支

常见错误

1. 标准的promise没有`.cancel()`,而且也违反了其他规则：只有**创建**promise的函数才可以完成，拒绝或取消这个promise。把`.cancel`暴露出来会破坏了封装星

2. 有些聪明的人想出利用 Promise.race() 来做取消。问题在于**取消的控制来自于创建 promise 的函数，这个函数是唯一你能做清理工作的地方**，例如清空 timeout 或者清空对数据的引用来释放内存等等。

3. 当你忘记处理拒绝状态的promise时，Chrome 会在控制台到处抛出警告消息！


所以我们可以承认的那个代表是否取消的值可以是一个promise本身，可能是这样：

``` javascript
const wait = (
  time,
  cancel = Promise.reject()
) => new Promise((resolve, reject) => {
  const timer = setTimeout(resolve, time);
  const noop = () => {};

  cancel.then(() => {
    clearTimeout(timer);
    reject(new Error('Cancelled'));
  }, noop);
});

const shouldCancel = Promise.resolve(); // Yes, cancel
// const shouldCancel = Promise.reject(); // No cancel

wait(2000, shouldCancel).then(
  () => console.log('Hello!'),
  (e) => console.log(e) // [Error: Cancelled]
);
```
