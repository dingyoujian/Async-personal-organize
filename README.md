## 前端菜鸡的自我修养

```js

异步函数一些知识点总结笔记

```

### Callback Hell
>callback里调用很多层callback，这样写狠辣鸡

```js
function fn1(data, callback) {
    //...... callback里再callback 就会产生callback地狱
}

```

### Promise对象

>Promise对象基本含义

1.Promise对象代表一个异步操作，有三种状态：`Pending`（进行中）、`Resolved`（已完成，又称 Fulfilled）和`Rejected`（已失败）。
2.一旦状态改变，就不会再变，任何时候都可以得到这个结果
3.一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。当处于Pending状态时，无法得知目前进展到哪一个阶段

>基本用法

```js
//异步加载图片
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    var image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}

//封装ajax
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if (this.readyState !== 4) {
        return;
      }
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
#### Promise.prototype.then()

为Promise实例添加状态改变时的回调函数。 resolve(something)

```js
var p = new Promise(fn1);

p.then(fn2).then(fn3)

```

#### Promise.prototype.catch()

用于指定发生错误时的回调函数。 reject('error')
```js

var p = new Promise(fn1);

p.then(fn2).catch(fnError);

```

#### Promise.all()

将多个Promise实例，包装成一个新的Promise实例

```js

var p = Promise.all([p1, p2, p3]);

```

1.只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数
2.只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数

#### Promise.race()

将多个Promise实例，包装成一个新的Promise实例

```js

var p = Promise.race([p1, p2, p3]);
//定义超时
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);
p.then(response => console.log(response));
p.catch(error => console.log(error));
```
1.只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数

#### Promise.resolve()

直接调用resolve方法返回一个Promise对象
-如果参数是Promise实例则直接返回
-如果参数是带有then参数的对象，则返回一个Promise对象且立即执行then方法
-如果参数是字符串等，非以上两者，则返回一个Promise对象且输出这个参数
-如果不带有参数，则直接返回一个Promise对象

#### Promise.reject()

直接调用reject方法返回一个Promise对象，不管参数是啥，都返回这个参数

#### Promise.done()

表示回调链的尾端执行的函数，能保证抛出任何可能出现的错误,可以不提供参数

```js

fn(XX).then(fn1).catch(fn2).then(fn3).done();

```

#### Promise.finally()

表示不管Rejected还是Resolved都执行这个函数

```js

fn(XX).then(fn1).finally(fn2);

```

#### Promise.try()

用Promise.try包装一下可以catch到同步错误

```js

Promise.try(database.users.get({id: userId}))
  .then(...)
  .catch(...)

```
