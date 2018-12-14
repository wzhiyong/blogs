@(JavaScript)
# javascript多线程
## 单线程
我们都知道javascript是一门单线程的语言，意思就是所有代码都跑在一个线程上，每段代码都是按顺序执行的。也就是说我们前一段代码执行时间决定后一段代码的开始执行时刻，如果前一段代码十分复杂或者耗时就会拖后后面的所有代码，就降低我们整体代码的响应速度，这种情况就暴露出了单线程一些缺点，所以如果能再开一个线程做一些耗时耗力即提升了我们代码的响应速度也可以充分利用计算机的计算能力就美好了。下面就进入这个美好的东西。
## Web Woker
web woker允许我们在创建一个线程去执行一些耗时的任务，这些操作不会影响主线程也不会被主线程所影响，当任务执行完成之后我们可以将结果返回到主线程。这样就可以大大提高我们主线程的执行速度。
不过我们也要筛选放进去的任务，如果和主线程的交互比较频繁的操作就不要放进来了，可以把例如文件操作，http轮询，或者计算复杂结果简单得类似任务放进去，最后只需要抛出一个完成标识或者简单值就可以了。
### 使用
``` javascript
let woker = new Worker('//www.xxx.com/a.js');
```
这里就已经创建了一个线程，这里需要注意的是这里的a.js必须是一个网络文件且同源，不然worker是不会执行的但是也不会报错。
``` javascript
// main 
worker.postMessage() // 传输送信息给woker线程

worker.onmessage = function(e){ // 接收woker线程发送的信息
}
worker.onerror = function(e){ // 当woker线程中出现错误时会触发该事件
}
worker.terminate(); // 在主线程关闭worker
 

// woker
使用
self.onmessage = function(e){ // 接收主线程发送的信息
}

self.postMessage();  // 发送信息给主线程

self.close(); 在线程里关闭自己
```
线程之间的传输数据非常简单需要注意几个点
1. 数据的传输是深拷贝的
2. 传输一些特殊对象需要主线程需要在postMessage中传入第二个参数，是一个数组，在其中填入徐要传输的变量名。这样做的目的是移交这个变量的控制权，例如MessageChannel的port,必须要移交控制权之后才能正常传输不然会抛出错误。
``` javascript
	let messageChannel = new MessageChannel()
	worker.postMessage({event: 'messageChannel', data: messageChannel.port1}, [messageChannel.port1])
```
### 使用场景
worker使用并不复杂，但是需要我们考虑使用的场景，过度和无意义的使用会造成性能的浪费，我总结除了下面几个使用原则。
- 与主线程交互少或没有交互
- 耗时
- 生命周期长
- 需要不停执行，不能因为其他操作而阻塞

在使用完worker之后要及时的关闭，且不能滥用。
### 兼容
[Web Woker兼容性](https://caniuse.com/#feat=webworkers)
## MessageChannel
MessageChannel  可以创建一个消息通道，他会抛出两个属性port1、port2，它们是这个消息通道两端，可以互相传送消息。
```javascript
	const messageChannel = new MessageChannel();
	messageChannel.port1.postMessage() // 给port2发送消息
	messageChannel.port1.onmessage = function(e){// 接受port1发送的消息
		
	}
	messageChannel.port1.postMessage() // 给port1发送消息
	messageChannel.port1.onmessage = function(){// 接受port1发送的消息
			
	}
```
### MessageChannel和Web Woker
我们可以通过MessageChannel让两个worker之间直接通信减少与主线程之间的交互。
首先在主线程创建两个woker并且将MessageChannel两个端口发送过去。这里需要注意MessageChannel的port1和port2不能直接传输过去，需要移交控制权，不然会报下面这个错误。
![](https://testfund.10jqka.com.cn/ifundapp_app/public/wzy/assets/dist/MessageChannel.png)
``` javascript
// main
const messageChannel = new MessageChannel();
const worker1 = new Worker('//www.xxx.com/worker1.js');
const worker2 = new Worker('//www.xxx.com/worker2.js');
worker1.postMessage({event: 'messageChannel', data: messageChannel.port1}, [messageChannel.port1]);
worker2.postMessage({event: 'messageChannel', data: messageChannel.port2}, [messageChannel.port2]);
setTimeout(() => {
    worker1.postMessage({event: 'message', data: 'main to worker1'})
}, 10000);
```
在worker1中接收到port1储存起来用来以后的传输，并且发送信息给worker2
``` javascript
// worker1
self.onmessage = function (e) {
    self[`ev_${e.data.event}`](e.data.data);
};

function ev_messageChannel(data) {
    console.log(1,data)
    self.port = data;
    self.port.onmessage= function (e) {
        console.log(e.data)
    }
}

function ev_message(data) {
    console.log(data)
    say()
}

function say() {
    console.log('start worker1 to worker2');
    self.port.postMessage('arriving work2')
}
```
在worker2中接收到port2储存起来用来以后的传输，收到worker1发送来的消息在返回一条消息给woker1
``` javascript
// worker2
self.onmessage = function (e) {
    self[`ev_${e.data.event}`](e.data.data)
};

function ev_messageChannel(data) {
    console.log(2,data)
    self.port = data;
    self.port.onmessage= function (e) {
        console.log(e.data)
        say()
    };
}

function ev_message(data) {
    say()
}

function say() {
    console.log('start worker2 to worker1');
    self.port.postMessage('arriving worker1')
}
```