# JS 模块化

## 什么是模块化

将一个复杂的程序依据一定的规则封装成几个块，并进行组合合在一起。

块的内部数据/实现是私有的，只是向外部暴露一些接口（方法）与外部其他模块通信

## 模块化的进化史

全部定义在全局，污染全局环境，容易命名冲突

=》

简单对象封装，定义一个对象，把部分数据放在对象中，对象中的属性随时可以修改，不安全

=》

IIFE，立即执行函数，里面的变量不可知，解决安全问题

=》

IIFE增强模式，引入参数变量，引入依赖（引入jQuery就是立即执行函数，window上添加$属性）

### 为什么要模块化

降低复杂度，减少耦合度，按需部署（引入）

#### 模块化好处

- 避免命名冲突
- 更好的分离，按需加载
- 更高复用性
- 高可维护性

#### 提出模块化后带来的问题（由此引入模块化规范）

- 请求过多
- 依赖模糊
- 难以维护

## 模块化规范

### CommonJS(nodeJS)

#### 规范

每个JS文件都可当做一个模块；

在服务器端：模块的加载是运行时**同步**加载的（会阻塞）

在浏览器端：模块需要提前编译打包处理

#### 基本语法

暴露模块，暴露的本质都是exports对象

- module.exports = value
- exports.xxx = value

引入模块

- require()：参数第三方模块为模块名，自定义模块为模块文件路径

#### 实现

服务器端：node.js

浏览器端：Browserify（CommonJS的浏览器端的打包工具）

### AMD（Asynchronous Module Definition）

#### 规范

专门用于浏览器端，模块的加载是异步的

#### 基本语法

暴露模块：

```js
// 定义没有依赖的模块
define(function(){
	return 模块;
});

// 定义有依赖的模块
define(['module1', 'module2'], function(m1,m2){
  return 模块;
});
```

引入模块：

```js
require(['module1', 'module2'], function(m1,m2){
  // 使用m1,m2
});
```

#### 实现

require.js

### CMD(阿里内部使用)

#### 规范

专门用于浏览器端，模块的加载是异步的

模块使用时才会加载

#### 基本语法

暴露模块

```js
// 暴露没有依赖的模块
define(function(require, exports, module){
  exports.xxx = value;
  module.exports = value;
})

// 定义有依赖的模块
define(function(require, exports, module){
  // 同步不引入依赖模块
  var module2 = require('./module2');
  // 异步引入依赖模块
  require.async('./module3', function(m3){
    
  })
  
  exports.xxx = value;
})
```

#### 实现

Sea.js

### ES6

#### 规范

依赖模块需要编译打包处理，有的浏览器不支持ES6

#### 语法

导出模块：export

引入模块：import

#### 实现

babel

