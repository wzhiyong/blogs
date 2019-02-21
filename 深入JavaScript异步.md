@(JavaScript)
# JavaScript异步
## 什么是异步？
异步简单的理解就是一段代码逻辑的执行时间是间断的，并不是连续的。通常都是执行一部分然后过一段时间在执行一部分。
## 为什么需要异步？
因为JavaScript是单线程的，它需要异步这种机制让代码更高效的执行。类似http请求，文件操作这种耗时的操作如果不异步执行那么将十分的卡顿。
## 如何实现？
实现的方式目前主要有三种回调函数、Promise、Generator 。其中回调函数是最早的实现方式，在ES6中加入了Promise和Generator。使用哪一种个人认为是取决于异步操作的复杂程度，例如设计多嵌套的使用后两者就比较好，如果只有一层回调函数更加的简洁明了。
### 回调函数
回调函数是十分简单方便的一种异步执行方式，就是把将来要执行的函数放在异步完成时能访问到的上下文中并调用。
```javascript
function callback(){
	console.log()
}
post('interface/a',callback)
```
这里callback就是回调函数，将自己当作参数传入，让异步执行完成之后可以访问到并执行。
我们可以看出来只有一层回调的时候，回调函数其实非常方便简单只用了JavaScript最简单的原理没有借助其他API。但是当嵌套十分复杂的时候就会出现很多问题。如：
``` javascript
post('interface/a',function(){
	post('interface/b',function(){
		post('interface/c',function(){
			....
		})
	})
})
```
这种嵌套十分复杂的回掉函数非常难以维护，首先我这里只是写了调用如果再加上其中的逻辑代码这段代码就会变得十分‘庞大’，而且这种一层一层的嵌套让其上下文环境互相交融，耦合性十分强，当我们要改其中的某一层时需要考虑上几层和下几层这不是我们想要的。所以回调函数在只有一层的时候使用最佳，其他情况下不推荐使用。
### Promise
Promise是对异步执行的一种解方案，引入了Promise对象。
####使用
Promise的原理十分简单，就是状态的改变，并在变成不同状态的时候执行不同的函数。有两种状态改变路径pending->fulfilled
和pending->rejected。一个Promise对象只能改变一次状态。改变状态的方式也很简单就是执行resolve和reject函数。
``` javascript
let promise = new Promise((resolve,reject) => {
	http.post('interface/a',function(){
		resolve('success') // pending->fulfilled
	},function(){
		reject('fail') // pending->rejected。
	})
})
```
这其中的resolve和reject 是内置的参数。
当状态改变之后就通过Promise.protoype.then或Promise.protoype.catch方法来执行对应状态的函数。
``` javascript
promise.then((data)=>{
	// pending->fulfilled执行
}).catch(error =>{
	// pending->rejected执行
})
```
Promise的基础使用十分简单，这种链式处理模式也十分符合我们处理代码的逻辑思路，当处理同时需要多个异步操作时更能体现出他的有用之处。
##### 链式使用
我们经常会遇到几个异步任务连续依赖的情况，如果使用回调函数就会出现回调地狱的问题，用Promise就十分清楚明白。
``` javascript
// 回调函数
post('interface/a',function(){
	post('interface/b',function(){
		post('interface/c',function(){
			....
		})
	})
})
	// Promise
post('interface/a').then(data =>{
	return post('interface/b')
}).then(data => {
	return Post('interface/c'
}).then(data => {
	....
})
```
对比之下我们可以明显的感觉到Promise更加的简洁明了，相比回调函数的嵌套最主要解决了以下两个问题：
1. 上下文互相独立，通过参数交互，解耦合。
2. 书写清楚明白，链式结构。
##### Promise.all
在处理某个操作需要多个异步操作全部完成时在执行，Promise也有绝佳的解决方式。
``` javascript
 Promise.all([promise1,promise2,promise3]).then(data => {}).catch(e => {})
```
只有当所有的Promise全部变为fulfilled状态时就会执行then中的函数，当其中一个变为rejected状态就会执行catch函数，很好的解决了上述的情况。

再使用Promise.all时有下面几个注意点
1. Promise.all中的实例如果自己定义了then外部将获取then的返回值，如果没有返回值则是undefined。
2. Promise.all中的实例如果自己定义了catch 外部将不会捕获他的错误，出现错误时Promise.all的状态也不会变为rejected，Promise.all的then中对应的值是catch的返回值。
3. Promise.all中实例自己定义的then和catch如果出现错误，那外部将捕获他们
``` javascript
let Pro1 = new Promise((resolve) => {
    setTimeout(() => {
        resolve('Pro1')
    }, 1000);
}).then(data => {
    console.log(data) // 'Pro1'
});
let Pro2 = new Promise((resolve) => {
    setTimeout(() => {
        resolve('Pro2')
    }, 2000);
});
let Pro3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject('Pro3 Error')
    }, 3000);
}).catch(e => {
    console.log(e); // Pro3 Error
    return `${e} catch`
});

Promise.all([Pro1, Pro2, Pro3]).then(data => {
    console.log(data) // [ undefined, 'Pro2', 'Pro3 Error catch' ]
});
```
#### 异常处理
Promise的的异常处理也有些区别。在Promise中如果出现了错误，js并不会终止执行。
```  javascript
new Promise(() =>{
    console.log(x) // UnhandledPromiseRejectionWarning: ReferenceError: x is not defined
});
setTimeout(()=>{
    console.log(3) // 3
},3000);
```
这里我们在Promise中输出一个不存在值x应该通畅会报错并终止js的执行但在Promise中会打印出错误但是代码会继续执行。如果我们有写catch,错误会被catch所捕获
``` javascript
new Promise(() =>{
    throw 'err'
}).catch(e => {
	console.log(e) // 'err'
});
```
这里catch回捕获到错误，所以我们平时在使用Promise时最好要写上catch函数。
#### 总结
Promise核心就是状态的改变，并执行不同的回调函数。与回调函数相比是增加了一个全新的API，在处理多异步任务的时候十分有用，在代码的书写上（链式）也比回调函数（跳跃式）清楚明白，但是独特的错误处理和一些API需要充分理解才能使用好。
### Generator异步
### async/await