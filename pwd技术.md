# PWA
PWA是Progressive Web App的英文缩写， 翻译过来就是渐进式增强WEB应用， 是Google 在2016年提出的概念，2017年落地的web技术。目的就是在移动端利用提供的标准化框架，在网页应用中实现和原生应用相近的用户体验的渐进式网页应用。可以理解成“将网页书签添加到手机屏幕”,然后通过技术实现离线加载，使用体验上类似原生应用。
### 原理
PWA涉及的技术领域全部在前端范围内，主要使用了server worker、cacheStorage两个技术实现了离线缓存提升速度。
#### service worke
service worker是一种特殊的web worker，他是独立于主线程，相当于一段独自运行在后台的脚本，基于service worker可以实现请求拦截分发，消息推送，静默更新以及地理围栏等服务。
#####  使用
1. 注册service worker注册成功后页面生效，执行注册的service worker文件。
2. 执行生命周期 install，安装成功后则生效。
3. service worker生效后可以拦截页面发出的所有请求，根据需求和环境可以选择是否从缓存中处理请求。
##### 注意
1. 上下文与主线程隔离，可以通过postMessage通信。
2. 游览器自动回收和激活service worker。
3. 渐进式，不会影响到不兼容的版本。
#### cacheStorage
cacheStorage是一种特殊缓存，他缓存的是HTTP请求，缓存请求报文和响应报文。我们可以缓存一些静态文件请求然后在断网的情况下使用。理论上我们可以缓存所有请求，需要根据业务和需求选择。
#### demo
``` javascript
// index.js
navigator.serviceWorker.register('./service-worker.js').then(function (registration) {
    console.log('Registration successful, scope is:', registration.scope);
}).catch(function (error) {
    console.log('Service Worker registration failed, error:', error);
});

//service-worker.js
const CACHE_NAME = 'v1'; // 缓存库名
// 安装
self.addEventListener('install', function (event) {
    event.waitUntil(
        caches.open(CACHE_NAME).then(function (cache) { // 安装正常

        }).then(function () { // 安装失败
            return self.skipWaiting();
        })
    );
});


// 监听页面所有请求
self.addEventListener('fetch', function (event) {
    // 拦截请求
    event.respondWith(
        caches.open(CACHE_NAME).then(function (cache) {
            // 正常进行网络请求
            // 可以根据业务和需求定制这部分代码 比如请求静态文件全部走缓存等
            return fetch(event.request).then(function (networkResponse) {
                // 如果请求正常则返回请求的值
                cache.put(event.request, networkResponse.clone());
                return networkResponse;
            }).catch(function () {
                // 出现错误则从缓存中获取
                return cache.match(event.request);
            });
        })
    );
});
```
### 优缺点
#### 优点
- 轻量
- 简单
- 更新快
- 用户安装使用简单
#### 缺点
- 兼容性 service worker ios支持是从11.3开始，并且无法再iphone上实现标准化
- ios/android差异
