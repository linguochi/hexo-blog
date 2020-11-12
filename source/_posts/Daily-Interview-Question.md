---
title: 前端面试每日一题精彩解析
author: 林国池
date: 2020-11-11 20:40:38
tags:
---

# 前置知识

## JavaScript 基础

### reduce

> arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
> Accumulator (acc) (累计器)
> Current Value (cur) (当前值)
> Current Index (idx) (当前索引)
> Source Array (src) (源数组)

reduce 为数组中的每一个元素依次执行 callback 函数，不包括数组中被删除或从未被赋值的元素, 最终返回函数累计处理的结果

# 每日一题与解答

## 第 160 题：

### 题目

输出以下代码运行结果，为什么？如果希望每隔 1s 输出一个结果，应该如何改造？注意不可改动 square 方法
[原文链接](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/389)

```javascript
const list = [1, 2, 3];
const square = num => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num * num);
    }, 1000);
  });
};

function test() {
  list.forEach(async x => {
    const res = await square(x);
    console.log(res);
  });
}
test();
```

### 解答：

> 普通的 for 等于同一个块作用域连续 await,而 forEach 的回调是一个个单独的函数，跟其他回调同时执行，互不干扰

forEach 大概可以这么理解

```javascript
Array.prototype.forEach = function(callback) {
  // this represents our array
  for (let index = 0; index < this.length; index++) {
    // We call the callback for each entry
    callback(this[index], index, this);
  }
};
```

也就是

```javascript
function test() {
  list.forEach(async x => {
    const res = await square(x);
    console.log(res);
  })(
    //forEach循环等于三个匿名函数;
    async x => {
      const res = await square(x);
      console.log(res);
    },
  )(1);
  (async x => {
    const res = await square(x);
    console.log(res);
  })(2);
  (async x => {
    const res = await square(x);
    console.log(res);
  })(3);

  // 上面的任务是同时进行
}
```

改成用普通的 for 语句，或者 of 循环后

```javascript
async function test() {
  for (let x of list) {
    const res = await square(x);
    console.log(res);
  }
}
//等价于

async function test() {
  const res = await square(1);
  console.log(res);
  const res2 = await square(2);
  console.log(res);
  const res3 = await square(3);
  console.log(res);
}
```

> 也可以使用 reduce 按顺序运行 Promise
> [mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)

```javascript
// reduce 方法的第一个参数是 async 函数，导致该函数的第一个参数是前一步操作返回的 Promise 对象，所以必须使用await等待它操作结束
// 如果没有 await _ 这一行，得到的结果和 forEach() 是一样的
list.reduce(async (_, x) => {
  await _;
  const res = await square(x);
  console.log(res);
}, undefined);
```

上面的 reduce 也可以有不用 await 的写法

```javascript
function test() {
  list.reduce(
    (pre, cur) => pre.then(() => square(cur)).then(console.log),
    Promise.resolve(),
  );
}

test();
```

> 采用 promise 链

```javascript
function test() {
  let promise = Promise.resolve();
  const start = Date.now();
  for (let i = 0; i < list.length; i++) {
    promise = promise
      .then(() => {
        const res = square(list[i]);
        return res;
      })
      .then(value => {
        console.log(`${Date.now() - start}`, value);
      });
  }
}
```

## 第 159 题

### 题目

实现 Promise.retry，成功后 resolve 结果，失败后重试，尝试超过一定次数才真正的 reject

### 用途

检测某个异步请求是否符合预期

### 解答

```javascript
Promise.retry = function(promiseFn, times = 3) {
  return new Promise(async (resolve, reject) => {
    while (times--) {
      try {
        var ret = await promiseFn();
        resolve(ret);
        break;
      } catch (error) {
        if (!times) reject(error);
      }
    }
  });
};
function getProm() {
  const n = Math.random();
  return new Promise((resolve, reject) => {
    setTimeout(() => (n > 0.9 ? resolve(n) : reject(n)), 1000);
  });
}
Promise.retry(getProm);
```
