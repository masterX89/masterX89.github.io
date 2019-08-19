---
title: JavaScript中的async/await浅析（一）
date: 2019-03-09 10:28:22
categories:
	- JavaScript
tags:
	- JavaScript
	- 异步
---

![hexo-pages001](/images/hexo-pages001.jpg)

async/await正式纳入[ES7](https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_Next_support_in_Mozilla#ECMAScript_2017)标准之中，给JavaScript的异步编程带来了许多的便利。首先介绍一下async/await的使用场景（大部分情况是由于promise链式带来的不便利）：

- 多个promise的then链，后续的步骤需要之前的**每一个**步骤的结果
- 在循环中需要异步获取数据，循环外异步设置状态

# 什么是promise

从开头的场景就可以知道，使用async/await大部分情况是由于Promise在解决某些需求的时候力不从心了，那么我们就需要先了解Promise，这里是MDN的[Promise定义](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)和[使用Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises)，在理解Promise可以胜任的场景之后，再引入async/await的优点。

> **Promise** 对象用于表示一个异步操作的最终状态（完成或失败），以及其返回的值。
>
> Promise 本质上是一个绑定了回调的对象，而不是将回调传进函数内部。

<!-- more -->

首先我们设计一个业务场景，需求中task1从服务器异步获取数据，然后再传递给task2完成任务。代码如下：

```javascript
function task1() {
    setTimeout(() => console.log('task1 take too longtime'), 1000);
}

function task2(){
    console.log('this is task2');
}

task1();
task2();
```

然而输出结果却如下：

```
this is task2
task1 take too longtime
```

如果在实际场景中，task2的参数就会为undefined，原因是由于task1是异步函数调用，那么我们如果想要这两个task顺序执行应该怎么办呢？在这里引入Promise的[约定](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises#%E7%BA%A6%E5%AE%9A)：

> - 在 JavaScript 事件队列的[当前运行完成](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop#%E6%89%A7%E8%A1%8C%E8%87%B3%E5%AE%8C%E6%88%90)之前，回调函数永远不会被调用。
> - 通过 .then 形式添加的回调函数，甚至都在异步操作完成之后才被添加的函数，都会被调用。
> - 通过多次调用 .then，可以添加多个回调函数，它们会按照插入顺序并且独立运行。

从上述的约定可以看出，Promise最好的应用场景就是链式调用。

## 链式调用

还是上一个业务场景，这次我们用Promise的then链来解决这个异步函数调用的问题：

```javascript
function task1() {
    return new Promise(resolve =>{
        setTimeout(() => {
            console.log('task1 take too longtime');
            resolve();
        }, 1000);
    })
}

function task2(){
    console.log('this is task2');
}

task1().then(() => {
    task2();
});	
```

最终结果如下：

```
task1 take too longtime
this is task2
```

同时Promise可不是仅仅.then一次，所谓的链式调用即是可以形成一个then链，理论上可以一直then来将异步函数顺序地执行下去，如下面的业务场景共有三个任务，需要顺序执行，分别消耗100，200，300ms的时间：

```javascript
function task1(time) {
    console.log(`task1 take ${time} ms`);
    return new Promise(resolve => {
        setTimeout(() => resolve(), time);
    });
}

function task2(time) {
    console.log(`task2 take ${time} ms`);
    return new Promise(resolve => {
        setTimeout(() => resolve(), time);
    });
}

function task3(time) {
    console.log(`task3 take ${time} ms`);
    return new Promise(resolve => {
        setTimeout(() => resolve(), time);
    });
}

function finalTask(){
    console.time('finalTask');
    const time1 = 100, time2 = 200, time3 = 300;//三个任务分别模拟100，200，300ms
    task1(time1)
        .then(()=>task2(time2))
        .then(()=>task3(time3))
        .then(()=>{
            console.timeEnd('finalTask');
        })
}

finalTask();
```

最后的结果如下：

```
task1 take 100 ms
task2 take 200 ms
task3 take 300 ms
finalTask: 626.755ms
```

可以看到三个任务是顺序执行的，并且消耗的时间与计时器的时间相差无几。

## 链式调用传参

设计场景中一个常见的需求就是连续执行两个或者多个异步操作，并且每一个后来的操作都在前面的操作执行成功之后，带着上一步操作所返回的结果开始执行。下面演示promise链式传递参数：

```javascript
function task1() {
    return new Promise(resolve =>{
        setTimeout(() => {
            console.log('task1 take too longtime');
            resolve('this is task1 data');
        }, 1000);
    })
}

function task2(data){
    console.log('this is task2');
    console.log(data);
}

task1().then((data) => {
    task2(data);
});	
```

最后结果如下：

```
task1 take too longtime
this is task2
this is task1 data
```

可以看到任务1和2还是顺序执行的，并且在任务2中，拿到了任务1传递来的参数，这个参数通过任务1中的resolve函数传递给下一个异步函数。每一个 Promise 代表了链式中另一个异步过程的完成。

## Catch的后续链式调用

但是在很多场景中，可能存在需要catch error的情况，比如服务器宕机了，或者甚至服务器传来不正确的json数据（笔者就遇到了这种情况，json格式错误，但是前端可以通过字符串操作来纠正并解析数据，然而每次异步获取数据都会走catch）。在这种场景下，首先要明确一点，catch之后是可以继续链式调用promise的，同时catch中可以对数据处理并传递给下一个函数，代码如下：

```javascript
new Promise((resolve, reject) => {
    console.log('Initial');
    resolve();
})
.then(() => {
    throw new Error('Something failed');
    console.log('Do this');
})
.catch(() => {
    console.log('Do that');
    return 'error text';
})
.then((data) => {
    console.log(data);
    console.log('Do this whatever happened before');
});
```

最后的执行结果如下：

```
Initial
Do that
error text
Do this whatever happened before
```

对这个结果进行分析，可以看出异步函数是链式顺序执行的，其中“Do this”方法中由于抛出一个Error，所以没有输出“Do this”。接着catch方法中正确执行，并且假设“error text”为处理后的结果，是可以顺利传递给下一个方法的。