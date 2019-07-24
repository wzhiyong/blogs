# JavaScript Decorators
## decorators 是什么
decorators是在ES7中的概念，可以把它当成一个容器作用在目标函数和类上，然后对它做一系列操作返回一个新的函数和类。
``` javascript
@something
function doing(){
	...
}
```
我们上面使用装饰器作用在了doing函数上，上面的形式其实可以理解成下面这样
``` javascript
let doing = function (){
	...
}
doing = something(doing )
```
这里我们把doing函数传入了something函数然后返回一新的doing函数，这个新的doing函数经过something的处理已经添加了一些新的行为。

## 使用 decorators

### 环境
因为decorators是ES7的语法我们需要使用babel插件：
1. 安装 @babel/plugin-proposal-decorators
``` javascript
yarn add @babel/plugin-proposal-decorators
```
2. 配置
``` javascript
// package.json
"babel": {
  "plugins": [
    [
	 "@babel/plugin-proposal-decorators", 
	 {
	   "legacy": true
	 }
    ]
  ]
}
```
这样就可以在项目中使用装饰器了

### 定义

#### 作用在函数
定义作用在函数上的装饰器，使用的是ES5的Object.defineProperty 方法，通过控制对象属性描述符来处理函数。
``` javascript
	Object.defineProperty(obj, prop, descriptor)
```
Object.defineProperty方法接受三个参数：
- 目标对象
- 属性名
- 属性的描述符

装饰器就是让我们通过操作上述的变量来处理当前属性。
``` javascript
	function something (obj,prop,descriptor){
		let _value = descriptor.value;
		descriptor.value = function () {
			... // 操作一
			_value()
			...// 操作二
		}
	}
	
	@something
	function doing(){
		...
	}
```
我们在doing函数上加了装饰器something，并且在原来函数之前增加了操作一，之后增加了操作二，形成了一个新的函数。我们也可以修改其他属性描述符如writabled修改它的可读性，等等。

#### 作用在类
作用在类上比较简单就是传入了类本身，我们通过修改类的prototype对象来处理类。
``` javascript
import Immutable from 'immutable'

function shouldComponentDecorators(target) {
   const compareEqual = function (objA, objB) {
        if (objA === objB || Immutable.is(objA, objB)) {
            return true
        }

        if (typeof objA !== 'object' || objA === null ||
            typeof objB !== 'object' || objB === null) {
            return false;
        }

        const keysA = Object.keys(objA);
        const keysB = Object.keys(objB);

        if (keysA.length !== keysB.length) {
            return false;
        }

        for (let i = 0; i < keysA.length; i++) {
            if (keysB.indexOf(keysA[i]) === -1) {
                return false;
            }
            if (!Immutable.is(objA[keysA[i]], objB[keysA[i]])) {
                return false
            }
        }
        return true
    };
	target.prototype.shouldComponentUpdate = function (nextProps, nextState, nextContext) {
		return !compareEqual(this.state, nextState) || !compareEqual(this.props, nextProps)
	}
}
```
上面就实现了一个给react组件添加shouldComponentUpdate生命周期方法的装饰器，在使用是只需要在创建react类前面增加
@shouldComponentDecorators就可给它增加十分方便。
### 增加参数
我们可能需要同一个装饰器不同情况下有所差异或者一些定制化值，可以通过传递参数实现
``` javascript
@something(true)

// 函数
function something (value){
 return function(target, key, descriptor) {
	descriptor.value = function () {
		if(value) 	... // 操作一
		else ... // 操作二
		_value()
	}
  }
}

// 类
function something (value){
  return function(target) {
	target.prototype.value= value
  }
}
```

## 场景
decorators 能使用的场景很多如封装常用操作，封装重复代码，权限控制，路由等等。
目前有个 core-decorators （https://github.com/jayphelps/core-decorators）库已经定义类很多有用的装饰器。
我们在日常开发中可以总结提取出自己的装饰器库，可以有效的提升开发体验。

## 总结
ES7新加入的装饰器特性可以让我们简化代码，增加复用，增加代码的可维护性。
