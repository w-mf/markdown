# ES6 - Promise

### 目录

- 什么是promise
- Promise应用
- Promise VS callback
- Promise VS async

---
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

---

### 语法

#### new Promise( function(resolve, reject) {...});
> 一个 Promise有以下几种状态:<br>
&emsp;pending: 初始状态。<br>
&emsp;fulfilled: 意味着操作成功完成。(resolve 函数被调用时)<br>
&emsp;rejected: 意味着操作失败。（reject  函数被调用时）<br>

&emsp;&emsp;接收一个函数参数，并且传入两个参数：resolve，reject。<br>
&emsp;&emsp;resolve和reject函数被调用时，分别将promise的状态改为fulfilled（完成）或rejected（失败）。<br>
&emsp;&emsp;函数内部通常会执行一些异步操作，一旦完成，可以调用resolve函数来将promise状态改成fulfilled，或者在发生错误时将它的状态改为rejected。
如果在函数中抛出一个错误，那么该promise 状态为rejected。函数的返回值被忽略。
![image](../images/promise1.png)




