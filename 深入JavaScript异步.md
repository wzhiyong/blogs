@(JavaScript)
# JavaScript异步
## 什么是异步？
异步简单的理解就是一段代码逻辑的执行时间是间断的，并不是连续的。通常都是执行一部分然后过一段时间在执行一部分。
## 为什么需要异步？
因为JavaScript是单线程的，它需要异步这种机制让代码更高效的执行。类似http请求，文件操作这种耗时的操作如果不异步执行那么将十分的卡顿。
## 如何实现？
实现的方式目前主要有四种回调函数、Promise、Generator、async/await 。其中回调函数是最早的实现方式，在ES6中加入了Promise和Generator两个异步解决方案，ES7中加入了async/await，使用哪一种个人认为是取决于异步操作的复杂程度，如果都是只有一层的异步操作每一个都可取，不过回调函数更简单的使用方式可能更方便和简洁。如果需要多层嵌套的使用后几者，但是往往业务开发过程中几层的异步嵌套都是无法预测到的，所以还是不推荐使用回调函数。后面几者因为是新加入的特性兼容性是主要问题，目前运行在游览器上的代码主要使用的解决方案是Promise,在写node时使用async/await更好。
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
这种嵌套十分复杂的回掉函数非常难以维护，首先我这里只是写了调用如果再加上其中的逻辑代码这段代码就会变得十分‘庞大’，而且这种一层一层的嵌套让其上下文环境互相交融，耦合性十分强，当我们要改其中的某一层时需要考虑上几层和下几层这不是我们想要的。所以回调函数在只有一层的时候使用最佳，其他情况下不推荐使用。如果项目中有使用Promise等最好统一一种方式，因为其他方式的一层调用也很简单，而且像http方法最好统一入口，全局定义共用方法，这样在处理异常情况时更加安全和方便。
### Promise
Promise是对异步执行的一种解方案，引入了Promise对象。
#### 使用
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
Promise核心就是状态的改变，并执行不同的回调函数。与回调函数相比是增加了一个全新的API，在处理多异步任务的时候十分有用，在代码的书写上（链式）也比回调函数（跳跃式）清楚明白，但是独特的错误处理方式和一些API需要充分理解才能使用好。
### Generator异步
#### Generator函数
generator是ES6中新加入的一种特殊函数，与一般函数完全不同，提供了一种新的异步解决方案。简单来说，generator函数执行之后返回的是一个状态机，通过yield定义状态，通过next方法依次遍历状态机。在执行一次next之后如果不继续执行可以理解为这个函数暂停并封存了上下文环境，移交了控制权，等下次执行时在解封上下文获得控制权，是不是感觉和异步很像，发送一个请求等待，过一段时间在执行回调。
``` javascript
function* generator() {
  yield 'ijijin';
  yield 'hello';
}
let a = generator() // 执行返回了一个状态机
a.next() // { value: 'ijijin', done: false } 
a.next() // { value: 'hello', done: false }
a.next() // { value: 'undefined', done: true }
```
具体的generator语法可以去ECMAScript 6 入门上学习,这里只探讨他的异步部分：https://es6.ruanyifeng.com/#docs/generator
#### 异步使用
generator函数可以暂停执行并且不阻塞这满足异步最大的要求。
``` javascript
function* generator() {
  yield post('....');
  yield post('....');
}
```
上面有两个post请求我们如果每次执行完成之后再继续执行下一个，在执行的同时也不会阻塞其他代码的执行，并且这种写法看上去也很像同步的样子，是不是感觉很完美。现在只需要一个能让这个代码自动执行的方法就是每次异步完成之后自动执行下一个next()的方法。这里要与Promise结合，我这里写出一个实现方法，能实现的方法不止一种。
``` javascript
// fn 是一个generator函数
function generator(fn){
	let a = fn()// 生成状态机
	function run(data){
		if(data.done) return
		data.value.then(data => {
			a.next(data)
		})}
	run(a.next())
}

function* async(){
	let a = post('.....') // 请求A 返回一个promise
	// 处理请求A的结果a
	let b = post('.....') // 请求B 返回一个promise
	// 处理请求B的结果b
	。。。。。
}
generator(async)// 这里 请求A和请求B就会顺序执行。
```
这样写异步就不许要考虑太多的作用域，解耦等问题而且编码符合正常的同步逻辑，不会出现回调函数的回调地狱或者promise好多的.then.catch，对于后续的维护和其他的的阅读都十分友好，比较好的generator封装库是co函数库，他就是写了一个让generator自动执行的执行器。
### async/await
async/await是ES7中加入的新特性，也是可以更好处理异步，很多人都说async/await是异步的终极解决方法。
``` javascript
async function test(){
	let a = await post('...') // 这里要返回一个promise对象，不然也会转化成一个promise对象;
	.....
	let b = await post('...') 
}
```
是不是感觉和generator很像，特殊的函数标识（*/async），和特殊的运算符（yiled/await），不过对于异步函数明显后者两个名字明显让人更好的理解这是一个异步的操作。await会等待post返回的promise对象状态变为fulfilled或者reject然后才继续往下执行。async/await和上面的generator封装的使用promise自动执行的效果和原理都极为相似，他就是将generator的自动执行器封装了起来，做了更好的优化。
#### 异常处理
使用async/await时推荐使用try...catch处理异常，因为这样更好的理解和优美，当然也可以通过.catch来捕获错误，但是我们使用aync/await让代码已经看上不很不像‘异步’了，如果用上.catch就不那么完美了。
## 兼容性
当我们使用新的API时肯定要考虑API的兼容性,下面整理了一下各个API的兼容性
### promise
![enter image description here](https://testfund.10jqka.com.cn/ifundapp_app/public/wzy/testimg/dist/generator.png)

### generator
![enter image description here](https://testfund.10jqka.com.cn/ifundapp_app/public/wzy/testimg/dist/promise.png)

### async/await
![enter image description here](https://testfund.10jqka.com.cn/ifundapp_app/public/wzy/testimg/dist/async.png)
最新的一般游览器都已经兼容了这些属性，如果要兼容低版本可是使用babel编译，但是由于promsie是原生的API所以只能引用ployfill或者重新Promise，async/await和generator是可以经过babel编译兼容低版本，但是async/await是依赖Promise的所以使用时还需要考虑Promise的兼容性。

## 总结
上面的几种方式其实都能解决我们的需求，但是从健壮性、可维护性、可读性方面async/await最优，但是由于受游览器环境的兼容限制可以在游览器环境中使用Promise来解决，在node环境中要尽量去拥抱最新的最好的解决方案，回调函数虽然有他的可取之处，但是从长远角度来看，不推荐使用。
