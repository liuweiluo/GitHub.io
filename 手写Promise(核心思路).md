### 手写Promise(核心思路)

index.js
```
/*
  1. Promise 就是一个类 在执行这个类的时候 需要传递一个执行器进去 执行器会立即执行
  2. Promise 中有三种状态 分别为 成功 fulfilled 失败 rejected 等待 pending
     pending -> fulfilled
     pending -> rejected
     一旦状态确定就不可更改
  3. resolve和reject函数是用来更改状态的
     resolve: fulfilled
     reject: rejected
  4. then方法内部做的事情就判断状态 如果状态是成功 调用成功的回调函数 如果状态是失败 调用失败回调函数 then方法是被定义在原型对象中的
  5. then成功回调有一个参数 表示成功之后的值 then失败回调有一个参数 表示失败后的原因
  6. 同一个promise对象下面的then方法是可以被调用多次的
  7. then方法是可以被链式调用的, 后面then方法的回调函数拿到值的是上一个then方法的回调函数的返回值
*/

const MyPromise = require('./myPromise');

  let promise = new MyPromise(function (resolve, reject) {
    setTimeout(function () {
      resolve('成功')
    }, 2000)
  })
  
  promise.then(value=>{
    console.log(value);
    return 100;
  },reason=>{
    console.log(reason);
  })
  
  
```

myPromise.js
```
const PENDING = 'pending'; // 等待
const FULFILLED = 'fulfilled'; // 成功
const REJECTED = 'rejected'; // 失败

class MyPromise{
  constructor (executor) {
      executor(this.resolve, this.reject)
  }
  // promsie 状态 
  status = PENDING;
  // 成功之后的值
  value = undefined;
  // 失败后的原因
  reason = undefined;
  // 成功回调
  successCallback = [];
  // 失败回调
  failCallback = [];
  
  resolve = value => {
    // 如果状态不是等待 阻止程序向下执行
    if (this.status !== PENDING) return;
    // 将状态更改为成功
    this.status = FULFILLED;
    // 保存成功之后的值
    this.value = value;
    // 判断成功回调是否存在 如果存在 调用
    this.successCallback && this.successCallback(this.value);
    // while(this.successCallback.length) this.successCallback.shift()()
  }
  reject = reason => {
    // 如果状态不是等待 阻止程序向下执行
    if (this.status !== PENDING) return;
    // 将状态更改为失败
    this.status = REJECTED;
    // 保存失败后的原因
    this.reason = reason;
    // 判断失败回调是否存在 如果存在 调用
    this.failCallback && this.failCallback(this.reason);
    //while(this.failCallback.length) this.failCallback.shift()()
  }
  
  then (successCallback, failCallback) {
    let promsie2 = new MyPromise((resolve, reject) => {
      // 判断状态
      if (this.status === FULFILLED) {
            let x = successCallback(this.value);
            // 判断 x 的值是普通值还是promise对象
            // 如果是普通值 直接调用resolve 
            // 如果是promise对象 查看promsie对象返回的结果 
            // 再根据promise对象返回的结果 决定调用resolve 还是调用reject
            resolvePromise(promsie2, x, resolve, reject)
      }else if (this.status === REJECTED) {
            let x = failCallback(this.reason);
            resolvePromise(promsie2, x, resolve, reject)
      } else {
        // 等待
        // 将成功回调和失败回调存储起来
        this.successCallback = successCallback;
        this.failCallback = failCallback;
      }
    });
    return promsie2;
  }
  
  function resolvePromise (promsie2, x, resolve, reject) {
    if (x instanceof MyPromise) {
      // promise 对象
      x.then(value => resolve(value), reason => reject(reason));
      // x.then(resolve, reject);
    } else {
      // 普通值
      resolve(x);
    }
  }
}

module.exports = MyPromise;
```
