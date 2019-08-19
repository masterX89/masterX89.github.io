---
title: JavaScript中的async/await浅析（二）
date: 2019-03-09 10:28:22
categories:
	- JavaScript
tags:
	- JavaScript
	- 异步
---

[![996.icu](https://img.shields.io/badge/link-996.icu-red.svg)](https://996.icu)

在上一篇的文章中已经简单介绍了promise的使用方法和then链的情况，但是还有一些状态下promise会力不从心：

- 在循环中需要异步获取数据，循环外获取数据
- 多个promise的then链，后续的步骤需要之前的**每一个**步骤的结果

# promise的不足

我在实际业务操作的时候，初期一直遇到一个困惑，我已经学会了如何使用promise链，但是却无法解决在一个循环中异步获取每一个item，并且要保证他们的**遍历次序**，最后在循环外setState。以及我无法在链式中的后续步骤获取之前**每一步**的操作结果。接着我将举例说明这些情况产生的场景，并提出我当前的解决方案。

<!-- more -->

## 循环异步获取数据

在这里假设有一个list数组，每一个item是这个任务的id，名称和执行时间。需要对该数组遍历，每一个遍历操作是异步函数，我把每一个item的执行时间刻意设置成不同时长，期待数组遍历时以id递增的次序读取每一个item。

```javascript
let content = [
{id:1,name:'task1',time:200},
{id:2,name:'task2',time:300},
{id:3,name:'task3',time:100}];
function getData(item){
    return new Promise(resolve => {
      setTimeout(()=>{
         console.log(item);
         resolve();
     },item.time);
  })
}
content.map((item)=>{
    getData(item);
});
console.log('I wanna result');
```

下面看一下测试效果：

```
I wanna result
{ id: 3, name: 'task3', time: 100 }
{ id: 1, name: 'task1', time: 200 }
{ id: 2, name: 'task2', time: 300 }
```

可以看到结果很不幸，读取到的item并不是以id递增的次序顺序输出的，而是以异步函数执行所需的时间打印出来的，并且期待的最终结果最先执行。那么我是否可以使用promise么？答案是可以的，但是我之后再另一篇文章中对此情况进行说明，在这里我们先解释使用async/await解决该场景。

在这个需求场景中，之前我一直使用的promise链无法发挥其作用了。原因很简单，我需要在循环中使用then来处理这个问题，但是我不知道应该加在哪里。并且最终结果也需要在循环结束的时候，加在then的最后。在阐述完promise链的第二个不足后，我再一起提出解决方案。

## 获取每一步结果

在这个需求场景中，我们设计一个问题，我们需要执行三个任务，这三个任务是异步函数，但是在第三个任务执行的时候，需要用到前两个函数的执行结果。拿到前一个执行结果很简单，但是如果想要获取到的一个函数的执行结果，就需要将第一个函数的执行结果添加到第二个函数传递给第三个函数的结果中。在这里我直接字符串连接，最后结果可以通过截取逗号来得到每一个结果。除了字符串连接还可以将每一步结果push到数组中，然后把数据数组在then链中传递。

```javascript
function task1() {
    return new Promise(resolve => {
        setTimeout(() => resolve('result1'), 100);
    });
}

function task2(data) {
    return new Promise(resolve => {
        setTimeout(() => resolve(`${data},result2`), 200);
    });
}

function task3(data) {
    return new Promise(resolve => {
        setTimeout(() => resolve(`${data},result3`), 300);
    });
}

task1()
.then((data)=>task2(data))
.then((data)=>task3(data))
.then((data)=>{
    console.log(data);
})
```

下面是最后的结果，这是第三个任务中的异步函数获取到的最后结果，可以通过逗号将字符串切割获取到每一步的结果。

```
result1,result2,result3
```

# async/await介绍

在这里可以先看一下[阮一峰老师的介绍](http://www.ruanyifeng.com/blog/2015/05/async.html)，然后在下面我给出我的一些理解。

> 异步I/O不就是读取一个文件吗，干嘛要搞得这么复杂？**异步编程的最高境界，就是根本不用关心它是不是异步。**

我们从最开始的问题入手，我们遇到了什么问题？一个变量初始化—>异步读取服务器数据—>修改变量。但是在这个过程中，由于异步方法的存在，使得我们往往先执行了修改变量的操作，后从服务器获取到了数据。那么，我们是不是可以用一种方法，让我们可以不用去关心一个方法是不是异步函数，但是可以很轻松的去异步编程呢？async/await就提供了这样一个解决方案。

## 什么是async/await

1. **async**用于将一个函数申明为异步函数
2. **await**用于等待一个异步函数的执行完成，即通过await来调用一个async函数
3. 且根据语法规定：await只能出现在async函数中

## 什么是async

先通过一个测试程序来输出async申明的函数：

```javascript
async function getData(){
    return 'what is async';
}
console.log(getData());
```

最后控制台输出如下：

```
Promise { 'what is async' }
```

通过这个例子就可以明白，原来用async申明的函数，最后也是返回一个promise啊！那么我们依旧可以使用then链来让异步函数按自己想要的顺序执行，如下：

```javascript
async function getData(){
    return 'what is async';
}
getData().then((data)=>{console.log(data)});
```

我们通过then链把这个异步函数的结果输出出来，可以看到async申明的函数，本质上就是一个异步函数，也可以使用then来获取到结果

```
what is async
```

## 什么是await

我们只有将async和await组合使用，才可以看出async/await比promise更加优秀的地方。首先先看一下await如何使用，如下是样例程序：

```javascript
async function getData(){
    return 'what is async';
}
async function test(){
    let temp = await getData();
    console.log(temp);
}
test();
```

最后结果和在then里获取promise结果一致：

```
what is async
```

但是！可以看出来`let temp = await getData();`这一条语句，在使用的时候，已经和平时编程习惯相差无几了。那么我们来看一看，如何解决文章开头promise的问题。

# async/await解决方案

## 循环异步获取数据

这是文章开头的问题，这里重述一遍。这里假设有一个list数组，每一个item是这个任务的id，名称和执行时间。需要对该数组遍历，每一个遍历操作是异步函数，我把每一个item的执行时间刻意设置成不同时长，期待数组遍历时以id递增的次序读取每一个item。

但是我们这次，使用async/await来重写这个场景，不使用then，预期结果为：三个任务task按照id递增顺序输出，三个任务执行完成后，最终结果才输出。

```javascript
let content = [
{id:1,name:'task1',time:200},
{id:2,name:'task2',time:300},
{id:3,name:'task3',time:100}];
function getData(item){
    return new Promise(resolve => {
      setTimeout(()=>{
         console.log(item);
         resolve();
     },item.time);
  })
}
async function test(){
    for (let item of content) {
        let temp = await getData(item);
    }
    console.log('I wanna result');
}
test();
```

可以看出，使用async和await不仅可以写出类似于适应平时编程习惯的写法，而且还可以满足场景需求。

```
{ id: 1, name: 'task1', time: 200 }
{ id: 2, name: 'task2', time: 300 }
{ id: 3, name: 'task3', time: 100 }
I wanna result
```

但是这个场景中有一个问题存在，由于**forEach**和**map**这两种写法，会要求在遍历中使用一个**函数**，而await必须要存在于async申明的函数中。所以await在这两种循环中是无法胜任的，请尽量使用没有内部函数的循环。

## 获取每一步结果

重述一遍文章开头的场景：在这个需求场景中，我们设计一个问题，我们需要执行三个任务，这三个任务是异步函数，但是在第三个任务执行的时候，需要用到前两个函数的执行结果。这里我们改写用async/await的写法：

```javascript
function task1() {
    return new Promise(resolve => {
        setTimeout(() => resolve('result1'), 100);
    });
}

function task2() {
    return new Promise(resolve => {
        setTimeout(() => resolve(`result2`), 200);
    });
}

function task3() {
    return new Promise(resolve => {
        setTimeout(() => resolve(`result3`), 300);
    });
}

async function test(){
    let task1Data = await task1();
    let task2Data = await task2();
    let task3Data = await task3();
    console.log('task1Data: ',task1Data);
    console.log('task2Data: ',task2Data);
    console.log('task3Data: ',task3Data);
}

test();
```

从结果可以看到，三个任务是依次执行的，而且保存在了各自的变量申明之中，在后续的步骤中，可以在任意位置来调用读取异步函数的结果：

```
task1Data:  result1
task2Data:  result2
task3Data:  result3
```

# await结合then

（记于3月27日新的心得）

await和then链也是可以结合使用的，例如当我需要从异步函数获取到的对象中，再进行解析获取到特定属性。

```javascript
let data = {content:"test",id:1};
function get(){
    return new Promise(resolve=>{
        setTimeout(()=>resolve(data),5000);
    })
}
async function test(){
    let card = await get().then(data=>{return data.content});
    console.log(card);
}
test();
```

结果是：

```
测试结果
```

# 参考文献

1. [边城](https://segmentfault.com/u/jamesfancy)老师的[理解 JavaScript 的 async/await](https://segmentfault.com/a/1190000007535316)
2. [阮一峰](http://www.ruanyifeng.com/)老师的[async 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/async.html)

