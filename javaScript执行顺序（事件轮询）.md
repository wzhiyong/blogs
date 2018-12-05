@(JavaScript)
# javsScript执行顺序（事件轮询机制）
最近Vue源码看到$nextTick里面牵扯到 javsScript执行顺序有点晕所以专门整理了一下。

众所周知javaScript是单线程，所以异步操作就经常出现。要想真正弄懂javaScript的执行顺序就需要去真正理解异步任务的执行顺序，而这个过程的关键就是去理解javsScript的事件轮询机制。在学习事件轮询机制之前要先去理解任务队列。
## 任务队列
在异步任务执行完成之后，将之前安排好的任务加入到一个队列中去，之后根据事件轮询的规则去执行这些任务。这个队列就是任务队列。

例如我们在使用Promise时会使用then方法传入当前面的异步任务完成之后所执行的函数。
- 这个Promise就是我们的异步任务。
- then函数的参数就是之前安排好的任务。

事件轮询就是处理任务队列中任务的执行顺序，而任务队列分为两种macro-task 和 micro-task, 不同的异步任务会加入到不同的任务队列中区。

### macro-task （宏任务队列）
常见的宏任务：javascript(整体代码)、setTimeout、setInterval，setImmediate、I/O、UI渲染
这些异步类型会在异步完成之后加入到macro-task任务队列中去。
#### 源任务
在macro-task任务队列中还会细分几种不同的源任务队列其中setTimeout、setInterval为同一源任务。我们在将异步任务加入到macro-task队列时会判断当前有没有同源任务如果有则加入到同源任务的队列中，没有则新建一个源任务队列。
### micro-task（微任务队列）
常见的微任务：promise、MutationObserver、process.nextTick
这些异步类型会在异步完成之后加入到micro-task任务队列中去。
micro-task中没有源任务队列，只有一个队列。在我执行代码的时候发现process.nextTick永远会比promise先执行，所以我猜测在这其中还是有优先级概念存在的。
## 事件轮询机制
事件轮询就是根据一个规则顺序去执行两个任务队列中的任务。
### 规则
先说这个规则： macro-task执行一个源任务队列 -> micro-task 执行完队列中的所有任务
我们在前面说了任务队列的分类，每次轮询会从macro-task中取出一个源任务队列全部执行，然后再全部执行完 micro-task中所有的任务，这就是一个轮询。
### 优先级
两个任务队列中的任务的执行顺序也是有优先级存在的。
在我实验中：
- macro-task 队列里javascript(整体代码)优先级最高，setTimeout大概率高于setImmediate，剩下的因为无法控制时间无法做一个准确的比较。
- micro-task process.nextTick高于Promise

## demo
理论上的概念上面已经说完了，我这里举一个例子可以更好的理解。
``` javascript
    setImmediate(() => {  // 加入macro-task
        console.log(1)
    }, 0);

    setTimeout(() => {  // 加入macro-task
        console.log(2)
    }, 0);

    let timer = setInterval(() =>{  // 加入macro-task
        console.log(3);
        clearInterval(timer)
    },0);

    new Promise(resolve => {
        console.log(4); // 同步任务
        resolve()
    }).then(() => { // 加入micro-task
        console.log(5)
    });

    setTimeout(() => {   // 加入macro-task
        console.log(6);
        new Promise(resolve => {
            console.log(7); // 同步任务
            resolve() 
        }).then(() => {  // 加入micro-task
            console.log(8)
        });
    }, 0);

    new Promise(resolve => {
        console.log(9); // 同步任务
        resolve()
    }).then(() => { // 加入micro-task
        console.log(10)
    });

    process.nextTick(() => { // 加入micro-task
        console.log(11)
    });
```

答案：4、9、11、5、10、2、3、6、7、8、1。下面对每个输出做了一个简单的解析

4：同步任务主线程执行  
9：同步任务主线程执行  
11：macro-task中javascript(整体代码)任务执行完成，开始执行micro-task。其中process.nextTick优先级高先输出  
5：micro-task任务  
10：micro-task任务，此时当前micro-task任务队列已经全部执行完成一次事件轮询结束。  
2：开始第二次事件轮询，其中macro-task任务队列中当前有setTimeout和setImmediate两个源任务队列，其中setTimeout大概率先执行与setImmediate，查资料说是观察者的原因还没有仔细研究。所以开始执行setTimeout的源任务队列其中有三个任务，setTimeout、setInterval为同一源任务。源任务里的执行顺序根据加入的先后顺序执行。所以首先输出2  
3：setTimeout源任务队列任务  
6：setTimeout源任务队列任务。当前setTimeout源任务队列任务已经完成。在这个setTimeout中新建了一个Promise，所以这里又加入到micro-task中一个任务。  
7：同步任务主线程执行  
8：前面setTimeout源任务队列任务直线完成之后macro-task中的一个任务已经执行完成，此时开始执行micro-task中的任务。  
当前micro-task中一个一个刚刚在6中新建的promise所以输出，第二次轮询结束。  
1：第三次轮询开始当前macro-task中只有一个setImmediate源任务队列，执行完成之后所有都已经完成结束。  