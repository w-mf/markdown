# ES6 - Promise


### 什么是promise

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)：Promise 对象用于一个异步操作的最终完成（或失败）及其结果值的表示。(简单点说就是处理异步请求。我们经常会做些承诺，如果我赢了你就嫁给我，如果输了我就嫁给你之类的诺言。这就是promise的中文含义：诺言，一个成功，一个失败。)

- ES6 原生提供了 Promise 对象。
- Promise最大的好处是在异步执行的流程中，把执行代码和处理结果的代码清晰地分离了
- 有了Promise，串行的异步任务，不需要写一层一层的嵌套代码。如：

```
function test1(resolve, reject) {
    var isSuccess = true;
    setTimeout(function() {
        if (isSuccess) {
          resolve('success');
        } else {
          reject('error');
        }
    }, 1000);
}

var p1 = new Promise(test1);

p1.then( (res) => {
    console.log(1,res);
    return res+ ' 1';
}).then( (res) => {
    console.log(2, res);
    return new Promise( function(resolve, reject) {
        setTimeout(function() {
          resolve(' 2.1 in success');
        }, 500)
    });
}).then((res) => {
    console.log(3, res);
}).catch((res) => {
    console.log('catch', res)
});

```
![image](/images/promise2.png)

#### 原理

&emsp;&emsp;Promise对象代表一个未完成、但预计将来会完成的操作。它有以下三种状态：<br>

- pending：初始值。<br>
- fulfilled：代表操作成功。<br>
- rejected：代表操作失败。<br>

&emsp;&emsp;Promise只有两种状态改变的方式，既可以从pending转变为fulfilled或从pending转变为rejected。一旦状态改变，就会一直保持这个状态，不会再发生变化，promise.then绑定的函数就会被调用。

>Promise一旦新建就会「立即执行」，无法取消。


### 语法

#### new Promise( function(resolve, reject) {...});


&emsp;&emsp;接收一个函数参数，并且传入两个参数：resolve，reject。<br>
- resolve函数的作用：在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；<br> 
- reject函数的作用：在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。<br>

&emsp;&emsp;resolve和reject函数被调用时，分别将promise的状态改为fulfilled（完成）或rejected（失败）。<br>
&emsp;&emsp;函数内部通常会执行一些异步操作，一旦完成，可以调用resolve函数来将promise状态改成fulfilled，或者在发生错误时将它的状态改为rejected。<br>
&emsp;&emsp;如果在函数中抛出一个错误，那么该promise状态为rejected。函数的返回值被忽略。

![image](/images/promise1.png)

&emsp;&emsp;Promise实例生成以后，可以用then方法指定resolved状态和reject状态的回调函数。<br>

### API
#### then()
```
语法：Promise.prototype.then(onFulfilled, onRejected)
```

&emsp;&emsp;then方法会返回一个新的Promise实例,且返回值将作为参数传入这个新Promise的resolve函数。<br>
&emsp;&emsp;它有两个参数，分别为Promise从pending变为fulfilled和rejected时的回调函数（第二个参数非必选）。这两个函数都接受Promise对象传出的值作为参数。<br>
&emsp;&emsp;简单来说，then就是定义resolve和reject函数的，其resolve参数相当于：
```
var p2 = new Promise( function(resolve, reject) {
  console.log(1);
  resolve(2);
  console.log(3);
});
p2.then( (res)=>{//Promise中的resolve()执行改函数
    console.log('then',res);
} );
```
而新建Promise中的'resolve(data)'，则相当于执行resolveFun函数。

### catch()

```
语法：Promise.prototype.catch(onRejected)
```
该方法是.then(undefined, onRejected)的别名，用于指定发生错误时的回调函数。
```
var p3 = new Promise( function(resolve, reject) {
  var state = false;
  if(state){
      resolve('success');
  }else{
      reject('error');
  }
});
p3.then( (res)=>{
    console.log('success',res);
}).catch( (res)=>{//等价于.then(undefined,(res)=>{})
    console.log('error1',res);
})
//.then(undefined,(res)=>{
//    console.log('error2',res);
//});
```
promise对象的错误，会一直向后传递，直到被捕获。
> 上文提到过，promise状态一旦改变就会凝固，不会再改变。因此promise一旦fulfilled了，再抛错，也不会变为rejected，就不会被catch了。

#### all()

```
语法：Promise.all(arr)
```
该方法用于将多个Promise实例，包装成一个新的Promise实例。
```
var p = Promise.all([p1, p2, p3]);
```
&emsp;&emsp;Promise.all方法接受一个数组作参数，数组中的对象（p1、p2、p3）均为promise实例（如果不是一个promise，该项会被用Promise.resolve转换为一个promise)。它的状态由这三个promise实例决定。<br>

&emsp;&emsp;当p1,p2,p3状态都变为fulfilled，p的状态才会变为fulfilled，并将三个promise返回的结果，按参数的顺序（而不是 resolved的顺序）存入数组，传给p的回调函数

```
graph LR
p1_fulfilled-->p_fulfilled
p2_fulfilled-->p_fulfilled
p3_fulfilled-->p_fulfilled
```
```
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 1000, "1s success");
});
var p2 = new Promise(function (resolve, reject) {
    resolve('0s success');
});
var p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 3000, "3s success");
}); 

Promise.all([p1, p2, p3]).then(function(values) { 
  console.log(values); 
});
```
&emsp;&emsp;当p1, p2, p3其中之一状态变为rejected，p的状态也会变为rejected，并把第一个被reject的promise的返回值，传给p的回调函数
```
graph LR
p1_fulfilled-->p_rejected
p2_rejected-->p_rejected
p3_fulfilled-->p_rejected
```
```
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, reject, 1000, "1s error");
});
var p2 = new Promise(function (resolve, reject) {
    reject('0s error');
});
var p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 3000, "3s success");
}); 

Promise.all([p1, p2, p3]).then(function(values) { 
    console.log(values); 
}).catch((error)=>{
    console.log(error);
});
```
&emsp;&emsp;这多个 promise 是同时开始、并行执行的，而不是顺序执行。
```
function timer(ms) {
    return new Promise(function (resolve) {
        setTimeout(function () {
            resolve(ms);
        }, ms);
    });
}
var startDate = Date.now();
Promise.all([
    timerPromisefy(10),
    timerPromisefy(11),
    timerPromisefy(12),
    timerPromisefy(13)
]).then(function (values) {
    console.log(Date.now() - startDate + 'ms');
    console.log(values);
});
```

### race()
```
语法：Promise.race(arr)
```
&emsp;&emsp;该方法同样是将多个Promise实例，包装成一个新的Promise实例。
```
var p = Promise.race([p1, p2, p3]);
```
&emsp;&emsp;Promise.race方法同样接受一个数组作参数。当p1, p2, p3中有一个实例的状态发生改变（变为fulfilled或rejected），p的状态就跟着改变。并把第一个改变状态的promise的返回值，传给p的回调函数。

```
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 1000, "1s success");
});
var p2 = new Promise(function (resolve, reject) {
    resolve('0s success');
});
var p3 = new Promise((resolve, reject) => {
  setTimeout((ms)=>{
      //console.log('3s');
      resolve(ms);
  }, 3000, "3s success");
}); 

Promise.race([p1, p2, p3]).then(function(res) { 
  console.log(res);
});
```
&emsp;&emsp;在第一个promise对象变为resolve后，并不会取消其他promise对象的执行。

### resolve()
```
语法:
Promise.resolve(value);
Promise.resolve(promise);
Promise.resolve(thenable);
```
&emsp;&emsp;Promise.resolve(value)方法返回一个以给定值解析后的Promise对象。但如果这个值是个thenable（即带有then方法），返回的promise会“跟随”这个thenable的对象，采用它的最终状态（指resolved/rejected/pending/settled）；否则以该值为成功状态返回promise对象。

&emsp;&emsp;它可以看做new Promise()的快捷方式。

```
Promise.resolve('success');
/*******等同于*******/
new Promise((resolve) => {
    resolve('Success');
});
```
```
Promise.resolve('success').then((res)=>{
    console.log('success',res);
});
Promise.resolve([1,2,3]).then((res)=>{
    console.log('success',res);
});


var p4 = new Promise((resolve,reject)=>{
    var state = true;
    if(state){
        resolve('p4 resolve');
    }else{
        reject('p4 reject');
    }
}); 
Promise.resolve(p4).then((res)=>{
    console.log('success',res);
});


var p5 = Promise.resolve({
    then: function (resolve, reject) {
        resolve("这是一个带有then方法的对象");
    }
});
console.log(p5);
p5.then(function(value){
    console.log(value);
});

```

### reject()
 ```
 语法：Promise.reject(reason)
 ```
 这个方法和上述的Promise.resolve()类似，使Promise对象立即进入rejected状态，并将错误对象传递给then指定的onRejected回调函数。


promise.then(onFulfilled, onRejected)
在onFulfilled中发生异常的话，在onRejected中是捕获不到这个异常的。
promise.then(onFulfilled).catch(onRejected)
.then中产生的异常能在.catch中捕获

### 需注意的问题
- promise.then(onFulfilled, onRejected)
在onFulfilled中发生异常的话，在onRejected中是捕获不到这个异常的。<br>
promise.then(onFulfilled).catch(onRejected)
.then中产生的异常能在.catch中捕获。<br>

- 如果在then中抛错，而没有对错误进行处理（即catch），那么会一直保持reject状态，直到catch了错误。<br>

- 每次调用then都会返回一个新创建的promise对象，而then内部只是返回的数据。<br>
```
var p1 = new Promise(function (resolve) {
    resolve(100);
});
p1.then(function (value) {
    return value * 2;
});
p1.then(function (value) {
    return value * 4;
});
p1.then(function (value) {
    console.log(value); //==>100
});
    ↓↓
    ↓↓
var p2 = new Promise(function (resolve) {
    resolve(100);
});
p2.then(function (value) {
    return value * 2;
}).then(function (value) {
    return value * 4;
}).then(function (value) {
    console.log(value);//==>800
});

/******************/
function test(data) {
    var promise = Promise.resolve(data);
    promise.then(function(value) {
        return value + 1;
    });
    return promise;
}
test(10).then(function(value) {
   console.log(value);          //想要得到11，实际输出10
});
    ↓↓
    ↓↓
function test2(data) {
    var promise = Promise.resolve(data);
    return promise.then(function(value) {
        return value + 1;
    });
}
test2(10).then(function(value) {
   console.log(value);          //==>11
});
```

- promise状态变为resove或reject，就凝固了，不会再改变
```
console.log(1);
new Promise(function (resolve, reject){
    reject();
    setTimeout(function (){
        console.log('times');
        resolve();           
    }, 0);
}).then(()=>{
    console.log(2);
}).catch(()=>{
    console.log(3);
});
console.log(4);
```
## ES7 - async/await

- ES7支持，Node.js 7.6 支持
- async 和 await 基于 promise 的。使用 async 的函数将会始终返回一个 promise 对象。
- 在使用 await 的时候我们暂停了函数，而非整段代码。
- async 和 await 是非阻塞的。
- async/await使得异步代码看起来像同步代码
```
async function logger() {
  // 暂停直到获取到返回数据
  let data = await fetch('http://sampleapi.com/posts')
  console.log(data)
}


//Promise
const makeRequest = () =>
getJSON()
.then(data => {
  console.log(data)
return "done"
})
makeRequest()

//Async/Await
const makeRequest = async () => {
  console.log(await getJSON())
}
makeRequest()
```

- https://www.cnblogs.com/fundebug/p/6667725.html （为什么async/await）更好
- http://www.css88.com/archives/7980
- http://blog.csdn.net/sinat_17775997/article/details/60609498