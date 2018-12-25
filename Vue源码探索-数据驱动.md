# Vue 数据驱动
### MVVM
把Model和View用ViewModel关联起来。ViewModel负责把Model的数据同步到View显示出来，还负责把View的修改同步回Model。
![enter image description here](https://images2015.cnblogs.com/blog/746387/201702/746387-20170223155932085-1172851114.png)
###  jQuery vs Vue
#### jQuery
``` javascript
let name = 'jack';
<div id="name"></div>
$('#name').html(name)

name = 'tom'
$('#name').html(name)

```
#### Vue
``` javascript
<div>{{name}}<div>
name = 'jack';
name = 'tom'
```
#### 对比
- 清晰
- 简单
- 页面复杂难以维护
### Vue入口
``` javascript
new Vue({
    render: h => h(App)
}).$mount('#app');
```

### Vue Class 类的定义
``` javascript
function Vue (options) {
	 this._init(options)
 }

 Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    vm._uid = uid++
	...
    initLifecycle(vm) // 初始化声明周期
    initEvents(vm) // 初始化事件
    initRender(vm) // 初始化虚拟DOM
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm) // 对 data、methods、computed、watcher 做初始化操作，数据响应式
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    ...
  }
```
### 执行过程
初始化之后执行的第一个方式$mount()方法去挂载。
``` javascript
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  ...
  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
	...
  } else {
    updateComponent = () => {
      // vm._render() 生成虚拟DOM
      // vm._update() 创建真实DOM
      vm._update(vm._render(), hydrating)
    }
  }
  
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
	 ...
  return vm
 }
```


### 虚拟DOM
``` javascript
export default class VNode {

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }
}
```
### 数据响应式
#### watcher
``` javascript
    class Watcher {
        constructor(vm, expOrFn) {
            this.vm = vm;
            this.getter = expOrFn; // 我们传进来的那个方法
            this.desps = []; // 上一个状态的订阅
            this.newDeps = []; // 最新的订阅
            this.get() 
        }

        get() {
            pushTarget(this);
            this.getter.call(this.vm)
        }

        addDep(dep) {
            this.newDeps.push(dep);
            dep.addSub(this)
        }
		
        update() {
	        queueWatcher(this)       
        }

		run(){
		   thi.get()
		}

    }
```

#### observer
``` javascript
 function observer(data) {
     if(Object.prototype.toString.call(data ) !== "[object Object]") return;
     Object.keys(data).forEach((key => defineReactive(data, key, data[key])))
 }
```

``` javascript  
    function defineReactive(data, key, val) {
        let dep = new Dep();
        observer(data);
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get() { // 依赖收集
                if (Dep.target) {
                    dep.depend()
                }
                return val
            },
            set(newVal) { // 更新派发
                if (newVal === val) return;
                val = newVal;
                dep.modify();
            },
        })
    }
```

#### dep

``` javascript
       class Dep {
        constructor() {
            this.subs = []
        }

        depend() {
            if (Dep.target) {
                Dep.target.addDep(this)
            }
        }

        addSub(watcher) {
            this.subs.push(watcher)
        }

        modify() {
            this.subs.forEach(watcher => {
                watcher.update()
            })
        }
    }

   function pushTarget(_target) {
        Dep.target = _target // 全局唯一
    }

```


### 优化
- 不需要监听的数据不写在data里,
- 减少同时watcher的数量

### demo
不需要监听的数据不写在data里,
``` html
<template>
    <section>
        <p>{{name}}</p>
        <p>{{age}}</p>
    </section>
</template>

<script>
    export default {
        name: "Component",
        data(){
            return{
                name:'jack',
                age:'19'
            }
        },
        methods:{
          initStaticData(){
              this.idCard = '630105173722913214';
              this.tid = '1639102341'
          }
        },
        created(){
            this.initStaticData()
        }
    }
</script>

<style lang="less">

</style>
```
减少同时watcher的数量
- v-if 
- 分页
- 懒加载

### 整体例子
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue data text</title>
</head>
<body>
<div id="app">
    <p v-text="name"></p>
    <p v-text="age"></p>
    <p v-text="address"></p>
    <p v-text="name"></p>
</div>
</body>
<script>
    // 注册观察者
    function observer(data) {
        if(Object.prototype.toString.call(data )!== '[object Object]' ) return
        Object.keys(data).forEach((key => defineReactive(data, key, data[key])))
    }

    // 对数据添加get,set方法
    function defineReactive(data, key, val) {
        let dep = new Dep();
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get() { // 依赖收集
                if (Dep.target) {
                    dep.depend()
                }
                return val
            },
            set(newVal) { // 更新派发
                if (newVal === val) return;
                val = newVal;
                dep.modify();
            },
        })
    }

    // 订阅者
    class Dep {
        constructor() {
            this.subs = []
        }

        depend() {
            if (Dep.target) {
                Dep.target.addDep(this)
            }
        }

        addSub(watcher) {
            this.subs.push(watcher)
        }

        modify() {
            this.subs.forEach(watcher => {
                watcher.update()
            })
        }
    }

    // 监听
    class Watcher {
        constructor(vm, expOrFn) {
            this.vm = vm;
            this.getter = expOrFn;
            this.desps = []; // 上一个状态的订阅
            this.newDeps = []; // 最新的订阅
            this.get()
        }

        get() {
            pushTarget(this);
            this.getter.call(this.vm)
        }

        addDep(dep) {
            this.newDeps.push(dep);
            dep.addSub(this)
        }

        update() {
            this.get();
        }

    }

    function pushTarget(_target) {
        Dep.target = _target // 全局唯一
    }


    // Vue
    class Vue {
        constructor(options) {
            this.options = options;
            this._data = options.data;
            this.$el = document.querySelector(this.options.id);
            Object.keys(this._data).forEach(key => this._proxy(key));// 代理数据 可以通this访问
            observer(options.data); // 监听数据变化
            let updateComponent = () => {
                this._update(this._render())
            };
            new Watcher(this, updateComponent);
        }

        _proxy(key) {
            let _this = this;
            Object.defineProperty(_this, key, {
                enumerable: true,
                configurable: true,
                set(newVal) {
                    _this._data[key] = newVal;
                },
                get() {
                    return _this._data[key]
                }
            })
        }

        _update(odlVNode, newVNode) {
            // diff 算法 odlVNode,newVNode对比
            // 生成真实DOM
            this._createElement()
        }

        _createElement() {
            this._bindText();
        }

        _render() {
            // 生成虚拟DOM
        }

        _bindText() {
            let Dom = this.$el.querySelectorAll('[v-text]');
            for (let bindDom of Dom) {
                let attr = bindDom.getAttribute('v-text');
                bindDom.innerHTML = this._data[attr];
            }
        }
    }

    let vue1 = new Vue({
        id: '#app',
        data: {
            name: 'Jack',
            age: '23',
            address: '五常大道'
        }
    });
</script>
</html>
```
https://testfund.10jqka.com.cn/ifundapp_app/public/wzy/vue-data/dist/
