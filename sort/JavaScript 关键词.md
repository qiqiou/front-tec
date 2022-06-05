# JS基础

## 1. 闭包

### 1.1 谈谈对闭包的理解，闭包的用途，闭包的缺点

1. 是什么
2. 使用场景（私有变量、生命周期）
3. 缺点（内存泄露、改变值）

## 2. this指向

### 2.1 call模拟实现

原理：将{}变为{ fn }，调用fn后，再删除fn属性

### 2.2 apply模拟实现

原理同call

### 2.3 bind模拟实现

原理：首先判断类型是否为function，在返回一个函数，若是new 调用，替换成调用父类new，否则使用apply绑定this指向

### 2.4 new 模拟实现

原理：

1. 创建空对象
2. 修改原型链
3. 修改this指向
4. 执行并返回结果

### 2.5 function与箭头函数的区别

- 普通函数this：严格和非严格模式下的this指向，修改this方法

- 箭头函数this：this指向，arguments、prototype、构造函数、new、yield

### 2.6 call、apply、bind 的区别

关键字：

1. 第一个参数含义及默认指向
2. 第二个参数格式及是否支持多次传入
3. 返回结果

### 2.7 谈谈this对象的理解

提示：运行时生成、内部调用，一旦确定this指向就不可修改；

绑定规则：默认（window）、隐式（对象）、new（实例对象）、显式（apply等）

## 3. 原型对象

### 3.1 实例属性和原型属性的关系

关键字：属性来源、私有共有属性、重写属性、如何判断属性来源

### 3.2 prototype和\_proto\_的关系

关键字：[[prototype]]指针，通过\_proto\_访问构造函数的原型

### 3.3 JS继承的实现方式

> https://www.cnblogs.com/humin/p/4556820.html

#### 原型链继承： 将父类的实例作为子类的原型

- 缺点：新增属性有顺序、不能多继承、父类属性共享、父类构造函数不能传参

#### 构造继承：复制父类的实例属性给子类

- 优点：支持多继承、支持构造函数传参、父类属性不共享
- 缺点：非父类实例，父类原型属性不继承，函数不复用父类实例副本影响性能

#### 实例继承：为父类实例添加新特性，作为子类实例返回

- 优点：不限制调用方法
- 缺点：非子类实例，不支持多继承

#### 拷贝继承：遍历父类实例属性，拷贝至子类原型上

- 优点：支持多继承
- 缺点：内存高、父类不可枚举方法不可继承、非父类实例

#### 组合继承：构造函数继承+原型链继承

- 优点：实例原型属性都可继承、既是实例对象也是原型对象、属性不共享、可传承、函数可复用：
- 缺点：调用了两次父类构造函数

#### 寄生组合继承：通过，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

### 3.4 实现一个instanceof

遍历原型链

### △3.5 JavaScript原型，原型链 ? 有什么特点？

## 4. 作用域链和执行上下文

### 4.1 说说你对作用域链的理解

关键词：作用域是什么，有哪些作用域，分别举例；词法作用域；变量查找过程的作用域链

### 4.2 输出结果是什么

```js
function a(){
    this.b = 3
}
var b = 9;
a(); // a中的this.b指向全局

//说出输出什么
console.log(b)  // 3
```

```js
window.name = 'bytedance';

function A() {
    this.name = '123';
}

A.prototype.getA = function () {
    console.log(this.name + 1);
}
// 等同于
/* A = {
*  prototype: {
*    constructor: function(){
*      this.name = '123';
*    },
*    getA: function() {
*      console.log(this.name + 1);
*    }
*  }
*}
*/

var a = new A(); // this指向A
var funcA = a.getA;// this指向全局

// 等同于
/*
* funcA = {
*    console.log(this.name + 1);
*	}
*/

//输出什么
funcA(); // bytedance1
a.getA();// 1231
```

### 4.3 JavaScript中执行上下文和执行栈是什么

关键词：执行上下文分类：全局、函数（调用时生成）、eval；执行上下文的生命周期：创建（确定this、变量提升）、执行、回收；执行栈：后进先出

## 5. 变量提升

```js
var a=33;
function test(){
  console.log(a)
  var a='eee'
}
// 等同于
/* function test(){
*  	var a = undefined; // 变量提升
*  	console.log(a)
*  	a = 'eee'
*	}
*/
test() // undefined
```

```js
var temp = 1;
function test() {
  temp = 10;
  return;
  function temp() {}
}

// 等同于
/* function test() {
*  	function temp() {} // temp为test作用域内的变量，不会改变全局的temp
*  	temp = 10;
*  	return;
*}
*/
test();
console.log(temp);// 1
```

### 5.1 JavaScript文件在HTML文件里的执行顺序

关键字：

1. 加载js代码的方式：内联、外置；
2. 加载js的顺序：head中、body中，影响页面渲染；
3. 如何不阻塞页面：defer(并行下载，顺序执行，DOMContentLoaded之前执行)、async（下载后就执行）、动态加载js
4. script代码块的执行顺序：代码块直接变量、方法共享；变量和声明式函数提示；前面的代码块报错不影响后面代码块执行

## 6. 异步

### 6.0  **JavaScript运行机制是什么？**

关键词：单线程语言，事件循环机制分为同步任务和异步任务，同步任务直接进入执行栈，异步任务进入任务队列，同步任务执行完后再检查任务队列是否有任务执行；任务队列分为微任务和宏任务；settimeout、setinterval、promise、ajax、dom事件都会进入任务队列

### 6.1 **JS 异步解决方案的发展历程以及优缺点**

关键词：

1. 回调函数（回调地狱、try catch不能捕捉错误，不能return）
2. promise（解决回调地狱、需要回调函数捕捉错误）
3. Generator生成器（同步写法实现异步调用）
4. await/async

### 6.2 **你了解过Promise吗**

关键词：解决回调地域、promise的状态（pending、fulfilled、reject，不可更改）、promise的缺点（不能捕捉内部异常，无法终止操作）

### 6.3 **async/await 如何通过同步的方式实现异步**

关键词：generator的语法糖 -> 迭代器iterator -> 单链表 -> 变量过程、next指向->next方法返回的对象（done、value）->async function等同于function*，await等同于yield

### 6.4  **setTimeout、Promise、Async/Await 的区别**

setTimeout放在宏任务队列；

promise new直接执行，promise.then放在微任务队列；

async内遇到await函数，执行完await函数后，后面的代码放在微任务队列等待执行

### 6.5 **用 setTimeout 实现 setInterval，阐述实现的效果与 setInterval 的差异**

settimeout内执行一个方法后，在调用自身函数；

### 6.6 **promise实现ajax**

new XMLHttpRequest()、onreadystatechange、readystate===4、status===200、open()、send()

### 6.6 ajax原理是什么？如何实现

通过XMLHttpRequest对象发起请求，返回数据

### 6.7 async function A() { return 1 }，当我们调用 A 的时候，返回结果是什么

返回Promise.resolve(1)

### 6.8 实现一个 sleep 函数,比如 sleep(1000) 意味着等待1000毫秒

思想：new promise内调用setTimeout

1. Promise

2. Generator

3. async

4. setTimeout


### 6.9 介绍下 Promise.all 使用、原理实现及错误处理

promise.all的参数，返回结果

原理实现：遍历参数promise实例，依次调用保存返回结果到数组中，遍历完后返回结果数组；

### 6.10 函数实现`arr`的串行调用

- 使用async
- 利用`Promise.resolve()`
- 利用`reduce`函数的性质

### 6.11 设计并实现 Promise.race()

### △6.12  手写实现Promise

### 6.13 ajax的用途，ajax请求的五种状态

readystate的5个值

### 6.14 setTimeout(fn,0)的作用

任务队列尾部添加事件，等主任务和任务队列的事件全部执行完，在执行该事件

### 6.15 async 及 await 的特点，它们的优点和缺点分别是什么？await 原理是什么

async函数一定会返回一个Promise；async、await需要配套使用

解决回调地狱问题；

Generator语法糖

### 6.16 setTimeout、setInterval、requestAnimationFrame 各有什么特点？

### △6.17 async函数对 Generator 函数的改进

### △6.18 设计并实现promise.finally

finally() 方法返回一个Promise。在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。这为在Promise是否成功完成后都需要执行的代码提供了一种方式。

```js
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```



## 7. 事件循环

### 7.1 说说你对事件循环的理解

单线程语言、同步任务、异步任务、微任务、宏任务、promise、setTimeout、await

js是单线程语言、同一时间只能做一件事；

执行任务分为同步任务和异步任务，同步任务在主线程顺序执行，异步任务进入任务队列排队执行

主线程任务执行完后，再执行任务队列中的任务；

异步任务分为宏任务和微任务，微任务：promise、mutationObserver、nextTick等；宏任务：setTimeout、setInterval、dom事件等

async方法会返回一个promise对象，await之前的代码会在主线程顺序执行，await的代码会进入任务队列等待执行

## 8. 基础

### 8.1 **undefined和null的区别**

typeof取值、Number取值、定义不一样、使用场景举例（null：原型链终点，undefined：变量提升）

### 8.2 **JavaScript中怎么判断相等**

==(先转化类型再比较)、===（先比较类型）、Object.is()（NaN 等于NaN，+0 不等于-0）

### 8.3 **JS的数据类型有哪些**

六个基本数据类型和一个引用数据类型

### 8.3 显示转换和隐式转换

显示转换：

Number()：存在非数字的字符串全部转换为NaN，Number(null) 为0，Numbe(undefined)为NaN，Number('')为0

parseInt()：遇到非数字字符就停止转换

String()、Boolean()

隐式转换：

比较运算符：先转换为Boolean值再比较

算数运算符➕：存在一个字符串，就是字符串拼接

其他算数运算符：一般都是先转化为数值再运算

### 8.4 **symbol的应用场景**

定义常量、私有属性、key

### 8.5 自执行函数（IIFE）的作用是什么

放在 IIFE 里面的变量，并不会影响到其他外层的变量，也不会被外层的变量影响。

### 8.7 实现双向数据绑定

1. 获取input和show的dom
2. Object.defineProperty中get方法取input的value值，set方法设置input和show的值
3. 绑定input的keyup监听事件，设置show为2中定义的对象的属性值

### 8.8 写一个jsonp的实现

1. document.createElement('script')
2. setAttribute('src', url);
3. document.getElementsByTagName('head')[0].appendChild()

### 8.9 函数去抖

1. timer && clearTimeout(timer)
2. timer = setTimeout
3. fn.Apply(this,arguments)
4. timer = null



### 8.10 函数节流

1. timer && return;
2. timer = setTimeout
3. fn.apply(this, arguments);
4. timer = null

### 8.11 模拟Object.create

### △8.12 深拷贝浅拷贝的区别？如何实现一个深拷贝

基本类型数据保存在在栈内存中

引用类型数据保存在堆内存中，引用数据类型的变量是一个指向堆内存中实际对象的引用，存在栈中

#### 浅拷贝

浅拷贝，指的是创建新的数据，这个数据有着原始数据属性值的一份精确拷贝

如果属性是基本类型，拷贝的就是基本类型的值。如果属性是引用类型，拷贝的就是内存地址

在`JavaScript`中，存在浅拷贝的现象有：

- `Object.assign`
- `Array.prototype.slice()`, `Array.prototype.concat()`
- 使用拓展运算符实现的复制

#### 深拷贝

深拷贝开辟一个新的栈，两个对象属完成相同，但是对应两个不同的地址，修改一个对象的属性，不会改变另一个对象的属性

常见的深拷贝方式有：

- _.cloneDeep()
- jQuery.extend()
- JSON.stringify()
- 手写循环递归

#### 实现深拷贝

```js
var isObject = obj => {
    return (typeof obj === 'object' || typeof obj === 'function') && obj != null;
}
const deepClone = (obj, hash = new Map()) => {
    if (!isObject(obj))
        return obj;
    if (hash.has(obj))
        return hash.get(obj);
    var target = Array.isArray(obj) ? [] : {};
    hash.set(obj, target);
    reflect.ownKeys(obj).foreach(key => {
        target[key] = isObject(obj[key]) ? deepClone(obj[key], hash) : obj[key];
    })
    return target;
}

// 复杂版本
const clone = parent => {
  // 判断类型
  const isType = (obj, type) => {
    if (typeof obj !== "object") return false;
    const typeString = Object.prototype.toString.call(obj);
    let flag;
    switch (type) {
      case "Array":
        flag = typeString === "[object Array]";
        break;
      case "Date":
        flag = typeString === "[object Date]";
        break;
      case "RegExp":
        flag = typeString === "[object RegExp]";
        break;
      default:
        flag = false;
    }
    return flag;
  };

  // 处理正则
  const getRegExp = re => {
    var flags = "";
    if (re.global) flags += "g";
    if (re.ignoreCase) flags += "i";
    if (re.multiline) flags += "m";
    return flags;
  };
  // 维护两个储存循环引用的数组
  const parents = [];
  const children = [];

  const _clone = parent => {
    if (parent === null) return null;
    if (typeof parent !== "object") return parent;

    let child, proto;

    if (isType(parent, "Array")) {
      // 对数组做特殊处理
      child = [];
    } else if (isType(parent, "RegExp")) {
      // 对正则对象做特殊处理
      child = new RegExp(parent.source, getRegExp(parent));
      if (parent.lastIndex) child.lastIndex = parent.lastIndex;
    } else if (isType(parent, "Date")) {
      // 对Date对象做特殊处理
      child = new Date(parent.getTime());
    } else {
      // 处理对象原型
      proto = Object.getPrototypeOf(parent);
      // 利用Object.create切断原型链
      child = Object.create(proto);
    }

    // 处理循环引用
    const index = parents.indexOf(parent);

    if (index != -1) {
      // 如果父数组存在本对象,说明之前已经被引用过,直接返回此对象
      return children[index];
    }
    parents.push(parent);
    children.push(child);

    for (let i in parent) {
      // 递归
      child[i] = _clone(parent[i]);
    }

    return child;
  };
  return _clone(parent);
};
```

### 8.13  使用迭代的方式实现 flatten 函数

方法一：递归遍历数组

方法二：toString方法

### △8.14 实现 (5).add(3).minus(2) 功能

```js
Number.prototype.add = function(n) {
  return this.valueOf() + n;
};
Number.prototype.minus = function(n) {
  return this.valueOf() - n;
};
```

### 8.16 有哪些方式可以判断是否是数组，请分别介绍它们之间的区别和优劣

1. Object.prototype.toString.call()：[Object Type]
2. Instanceof： 原型链判断，[] instanceof Object 为true
3. Array.isArray()：可以判断iframe数组
4. typeof

### △8.17  为什么普通 `for` 循环的性能远远高于 `forEach` 的性能，请解释其中的原因。

### ※8.18 使用 JavaScript Proxy 实现简单的数据绑定

```js
<body>
  hello,world
  <input type="text" id="model">
  <p id="word"></p>
</body>
<script>
  const model = document.getElementById("model")
  const word = document.getElementById("word")
  var obj= {};

  const newObj = new Proxy(obj, {
      get: function(target, key, receiver) {
        console.log(`getting ${key}!`);
        return Reflect.get(target, key, receiver);
      },
      set: function(target, key, value, receiver) {
        console.log('setting',target, key, value, receiver);
        if (key === "text") {
          model.value = value;
          word.innerHTML = value;
        }
        return Reflect.set(target, key, value, receiver);
      }
    });

  model.addEventListener("keyup",function(e){
    newObj.text = e.target.value
  })
</script>
```

### ※8.19 JavaScript的变量名在定义时需要遵守哪些规则

1、必须以字母或者 $，_开头。
2、变量不能为js的关键字。
3、变量对大小写敏感，所以要注意大小写

### 8.20 JavaScript的内置对象有哪些。

1.pop
2.push
3.shift
4.unshift
5.slice
6.splice
7.concat
8.join

### 8.21 运算符优先级

### 8.22 谈谈js的垃圾回收机制

JavaScript拥有自动的垃圾回收机制，当一个值，在内存中失去引用时，垃圾回收机制会根据特殊的算法找到它，并将其回收，释放内存。

- **标记清除算法：**

1. 标记阶段，垃圾回收器会从根对象开始遍历。每一个可以从根对象访问到的对象都会被添加一个标识，于是这个对象就被标识为可到达对象。
2. 清除阶段，垃圾回收器会对堆内存从头到尾进行线性遍历，如果发现有对象没有被标识为可到达对象，那么就将此对象占用的内存回收，并且将原来标记为可到达对象的标识清除，以便进行下一次垃圾回收操作。
3. 缺点：垃圾收集后有可能会造成大量的 **内存碎片**。

- **引用计数算法：**

1. 引用计数的含义是跟踪记录每个值被引用的次数，如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。
2. 缺点： 循环引用没法回收。

### 8.23 引起JavaScript内存泄漏的操作有哪些，如何防止内存泄漏？

- 引起内存泄露的操作：

  - 全局变量引起的内存泄漏
  - 闭包引起的内存泄漏
  - dom清空或删除时，事件未清除导致的内存泄漏
  - 子元素存在引用引起的内存泄漏
  - 被遗忘的计时器

- 防止内存泄露的方法：

  - 及时清除引用。

  - 使用WeakSet和WeakMap，它们对于值的引用都是不计入垃圾回收机制的，所以名字里面才会有一个"Weak"，表示这是弱引用。

    ```js
    const wm = new WeakMap();
    const element = document.getElementById('example');
    wm.set(element, 'some information');
    wm.get(element) // "some information"
    ```

### 8.24 如何阻止事件冒泡和默认事件？

- 标准的DOM对象中可以使用事件对象的stopPropagation()方法来阻止事件冒泡，但在IE8以下中IE的事件对象通过设置事件对象的cancelBubble属性为true来阻止冒泡；
- 默认事件的话通过事件对象的preventDefault()方法来阻止，而IE通过设置事件对象的returnValue属性为false来阻止默认事件。

### 8.25 如何实现图片懒加载？

### 8.26 Js中forEach和map方法的区别

- forEach返回值是undefined,不可链式调用
- map返回一个新数组，原数组不会改变。
- 没有办法终止或者跳出forEach循环，除非抛出异常，如果想执行一个数组是否满足什么条件，可以用Array.every()或者Array.some();

### 8.27 script标签如何异步加载？

```js
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

**defer与async的区别是：**

1. defer要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行；async一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。一句话，defer是“渲染完再执行”，async是“下载完就执行”。
2. 另外，如果有多个defer脚本，会按照它们在页面出现的顺序加载，而多个async脚本是不能保证加载顺序的。

### 8.28 事件捕获和事件冒泡

- **DOM2级事件** 规定事件流包括三个阶段，事件捕获阶段、处于目标阶段和事件冒泡阶段。首先发生的事件捕获，为截获事件提供了机会。然后是实际的目标接收了事件。最后一个阶段是冒泡阶段，可以在这个阶段对事件做出响应。
- **事件冒泡**，即事件开始时由最具体的元素接收，然后逐级向上传播到较为不具体的节点window对象。
- **事件捕获** 的思想是不太具体的节点应该更早的接收到事件，而在最具体的节点应该最后接收到事件。
- 在支持addEventListener()的浏览器中，可以调用事件对象的 **stopPropagation()** 方法以阻止事件的继续传播。如果在同一对象上定义了其他处理程序，剩下的处理程序将依旧被调用，但调用 **stopPropagation()** 之后任何其他对象上的事件处理程序将不会被调用。不仅可以阻止事件在冒泡阶段的传播，还能阻止事件在捕获阶段的传播。IE9之前的IE不支持stopPropagation()方法，而是设置事件对象cancelBubble属性为true来实现阻止事件进一步传播。
- **e.preventDefault()** 可以阻止事件的默认行为发生，默认行为是指：点击a标签就转跳到其他页面、拖拽一个图片到浏览器会自动打开、点击表单的提交按钮会提交表单。

### 8.29 事件代理(事件委托)及其好处，如何知道是哪个节点的事件

- 事件委托就是利用 **事件冒泡** ，只指定一个事件处理程序，就可以管理某一类型的所有事件。

- 事件有两个属性 **e.target** 和 **e.currentTarget** ，target是真正发生事件的DOM元素，而currentTarget是当前事件发生在哪个DOM元素上。目标阶段也就是 target == currentTarget的时候。

  ```js
  window.onload = function(){
  　　var oUl = document.getElementById("ul1");
  　　oUl.onclick = function(ev){
  　　　　var ev = ev || window.event;
  　　　　var target = ev.target || ev.srcElement;
  　　　　if(target.nodeName.toLowerCase() == 'li'){
  　　　　　　　  alert(target.innerHTML);
  　　　　}
  　　}
  }
  ```

### 8.30 函数柯里化理解

- 只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

- 用途：1.延迟计算；2.参数复用；3.动态生成函数

  ```js
  const curry = (fn, currArgs=[]) => {
      return function() {
          let args = Array.from(arguments);
          [].push.apply(args,currArgs);
          if (args.length < fn.length) {
              return curry.call(this,fn,...args);
          }
          return fn.apply(this,args);
      }
  }
  ```

### 8.31 JS中substr与substring的区别？

- js中substr和substring都是截取字符串中子串，非常相近，可以有一个或两个参数。
- substr(start [，length]) 第一个字符的索引是0，start必选 length可选
- substring(start [, end]) 第一个字符的索引是0，start必选 end可选

### 8.32 使用 sort() 对数组 [3, 15, 8, 29, 102, 22] 进行排序，输出结果？

- sort 函数，可以接收一个函数，返回值是比较两个数的相对顺序的值
- 根据MDN上对Array.sort()的解释，默认的排序方法会将数组元素转换为字符串，然后比较字符串中字符的UTF-16编码顺序来进行排序。所以'102' 会排在 '15' 前面。结果是[102, 15, 22, 29, 3, 8]

### 8.33 map、filter和reduce的区别

- map 作用是生成一个新数组，遍历原数组，将每个元素拿出来做一些变换然后放入到新的数组中。map 的回调函数接受三个参数，分别是当前索引元素，索引，原数组
- filter 的作用也是生成一个新数组，在遍历数组的时候将返回值为 true 的元素放入新数组，我们可以利用这个函数删除一些不需要的元素
- reduce 可以将数组中的元素通过回调函数最终转换为一个值。对于 reduce 来说，它接受两个参数，分别是回调函数和初始值。回调函数接受四个参数，分别为累计值、当前元素、当前索引、原数组

### 8.34 Javscript数组的常用方法有哪些

数组基本操作可以归纳为 增、删、改、查，需要留意的是哪些方法会对原数组产生影响，哪些方法不会

下面对数组常用的操作方法做一个归纳

#### 增

下面前三种是对原数组产生影响的增添方法，第四种则不会对原数组产生影响

- push()：接收任意数量的参数，并将它们添加到数组末尾，返回数组的最新长度
- unshift()：在数组开头添加任意多个值，然后返回新的数组长度
- splice()：传入三个参数，分别是开始位置、0（要删除的元素数量）、插入的元素，返回空数组
- concat()：首先会创建一个当前数组的副本，然后再把它的参数添加到副本末尾，最后返回这个新构建的数组，不会影响原始数组

#### 删

下面三种都会影响原数组，最后一项不影响原数组：

- pop()：删除数组的最后一项，同时减少数组的` length` 值，返回被删除的项
- shift()：用于删除数组的第一项，同时减少数组的` length` 值，返回被删除的项
- splice()：传入两个参数，分别是开始位置，删除元素的数量，返回包含删除元素的数组
- slice()：用于创建一个包含原有数组中一个或多个元素的新数组，不会影响原始数组

#### 改

即修改原来数组的内容，常用`splice`

#### 查

即查找元素，返回元素坐标或者元素值

- indexOf()：返回要查找的元素在数组中的位置，如果没找到则返回-1
- includes()：返回要查找的元素在数组中的位置，找到返回`true`，否则`false`
- find()：返回第一个匹配的元素

#### 排序

数组有两个方法可以用来对元素重新排序：

- reverse()：将数组元素方向排列
- sort()：接受一个比较函数，用于判断哪个值应该排在前面

#### 转换方法

join() 方法接收一个参数，即字符串分隔符，返回包含所有项的字符串

#### 迭代方法

常用来迭代数组的方法（都不改变原数组）有如下：

- some()：如果有一项函数返回 true ，则这个方法返回 true
- every()：如果对每一项函数都返回 true ，则这个方法返回 true
- forEach()：数组每一项都运行传入的函数，没有返回值
- filter()：对数组每一项都运行传入的函数，函数返回 `true` 的项会组成数组之后返回
- map()：对数组每一项都运行传入的函数，返回由每次函数调用的结果构成的数组

### 8.35 Javascript字符串的常用方法有哪些

我们也可将字符串常用的操作方法归纳为增、删、改、查，需要知道字符串的特点是一旦创建了，就不可变

#### 增

- concat：用于将一个或多个字符串拼接成一个新字符串

#### 删

- slice()
- substr()
- substring()

#### 改

- trim()、trimLeft()、trimRight()：删除前、后或前后所有空格符，再返回新的字符串
- repeat()：接收一个整数参数，表示要将字符串复制多少次，然后返回拼接所有副本后的结果
- padStart()、padEnd()：复制字符串，如果小于指定长度，则在相应一边填充字符，直至满足长度条件
- toLowerCase()、 toUpperCase()：大小写转化

#### 查

- chatAt()：返回给定索引位置的字符，由传给方法的整数参数指定
- indexOf()：从字符串开头去搜索传入的字符串，并返回位置（如果没找到，则返回 -1 ）
- startWith()：字符串中是否以传入的字符串开头
- includes()：从字符串中搜索传入的字符串，并返回一个表示是否包含的布尔值

#### 转换

- split：把字符串按照指定的分割符，拆分成数组中的每一项

#### 模板匹配方法

- match()：接收一个参数，可以是一个正则表达式字符串，也可以是一个`RegExp`对象，返回数组
- search()：接收一个参数，可以是一个正则表达式字符串，也可以是一个`RegExp`对象，找到则返回匹配索引，否则返回 -1
- replace()：接收两个参数，第一个参数为匹配的内容，第二个参数为替换的元素（可用函数）

### 8.36 typeof 与 instanceof 区别

#### typeof

`typeof` 操作符返回一个字符串，表示未经计算的操作数的类型

使用方法如下：

```
typeof operand
typeof(operand)
```

`operand`表示对象或原始值的表达式，其类型将被返回

举个例子

```
typeof 1 // 'number'
typeof '1' // 'string'
typeof undefined // 'undefined'
typeof true // 'boolean'
typeof Symbol() // 'symbol'
typeof null // 'object'
typeof [] // 'object'
typeof {} // 'object'
typeof console // 'object'
typeof console.log // 'function'
```

从上面例子，前6个都是基础数据类型。虽然`typeof null`为`object`，但这只是` JavaScript` 存在的一个悠久 `Bug`，不代表`null `就是引用数据类型，并且`null `本身也不是对象

所以，`null `在 `typeof `之后返回的是有问题的结果，不能作为判断`null`的方法。如果你需要在 `if` 语句中判断是否为 `null`，直接通过`===null`来判断就好

同时，可以发现引用类型数据，用`typeof`来判断的话，除了`function`会被识别出来之外，其余的都输出`object`

如果我们想要判断一个变量是否存在，可以使用`typeof`：(不能使用`if(a)`， 若`a`未声明，则报错)

```
if(typeof a != 'undefined'){
    //变量存在
}
```

#### instanceof

`instanceof` 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上

使用如下：

```
object instanceof constructor
```

`object`为实例对象，`constructor`为构造函数

构造函数通过`new`可以实例对象，`instanceof `能判断这个对象是否是之前那个构造函数生成的对象

```
// 定义构建函数
let Car = function() {}
let benz = new Car()
benz instanceof Car // true
let car = new String('xxx')
car instanceof String // true
let str = 'xxx'
str instanceof String // false
```

关于`instanceof`的实现原理，可以参考下面：

```
function myInstanceof(left, right) {
    // 这里先用typeof来判断基础数据类型，如果是，直接返回false
    if(typeof left !== 'object' || left === null) return false;
    // getProtypeOf是Object对象自带的API，能够拿到参数的原型对象
    let proto = Object.getPrototypeOf(left);
    while(true) {                  
        if(proto === null) return false;
        if(proto === right.prototype) return true;//找到相同原型对象，返回true
        proto = Object.getPrototypeof(proto);
    }
}
```

也就是顺着原型链去找，直到找到相同的原型对象，返回`true`，否则为`false`

#### 区别

`typeof`与`instanceof`都是判断数据类型的方法，区别如下：

- `typeof`会返回一个变量的基本类型，`instanceof`返回的是一个布尔值
- `instanceof` 可以准确地判断复杂引用数据类型，但是不能正确判断基础数据类型
- 而` typeof` 也存在弊端，它虽然可以判断基础数据类型（`null` 除外），但是引用数据类型中，除了` function` 类型以外，其他的也无法判断

可以看到，上述两种方法都有弊端，并不能满足所有场景的需求

如果需要通用检测数据类型，可以采用`Object.prototype.toString`，调用该方法，统一返回格式`“[object Xxx]” `的字符串

#### 8.37 Javascript本地存储的方式有哪些？区别及应用场景？

#### 区别

关于`cookie`、`sessionStorage`、`localStorage`三者的区别主要如下：

- 存储大小：` cookie`数据大小不能超过`4k`，`sessionStorage`和`localStorage `虽然也有存储大小的限制，但比`cookie`大得多，可以达到5M或更大
- 有效时间：`localStorage `存储持久数据，浏览器关闭后数据不丢失除非主动删除数据； `sessionStorage `数据在当前浏览器窗口关闭后自动删除；`cookie`设置的`cookie`过期时间之前一直有效，即使窗口或浏览器关闭
- 数据与服务器之间的交互方式，` cookie`的数据会自动的传递到服务器，服务器端也可以写`cookie`到客户端； `sessionStorage`和`localStorage`不会自动把数据发给服务器，仅在本地保存

#### 应用场景

在了解了上述的前端的缓存方式后，我们可以看看针对不对场景的使用选择：

- 标记用户与跟踪用户行为的情况，推荐使用`cookie`
- 适合长期保存在本地的数据（令牌），推荐使用`localStorage`
- 敏感账号一次性登录，推荐使用`sessionStorage`
- 存储大量数据的情况、在线文档（富文本编辑器）保存编辑历史的情况，推荐使用`indexedDB`

#### cookie

`Cookie`，类型为「小型文本文件」，指某些网站为了辨别用户身份而储存在用户本地终端上的数据。是为了解决 `HTTP `无状态导致的问题

作为一段一般不超过 4KB 的小型文本数据，它由一个名称（Name）、一个值（Value）和其它几个用于控制 `cookie `有效期、安全性、使用范围的可选属性组成

但是`cookie`在每次请求中都会被发送，如果不使用 `HTTPS `并对其加密，其保存的信息很容易被窃取，导致安全风险。举个例子，在一些使用 `cookie `保持登录态的网站上，如果 `cookie `被窃取，他人很容易利用你的 `cookie `来假扮成你登录网站

关于`cookie`常用的属性如下：

- Expires 用于设置 Cookie 的过期时间

```
Expires=Wed, 21 Oct 2015 07:28:00 GMT
```

- Max-Age 用于设置在 Cookie 失效之前需要经过的秒数（优先级比`Expires`高）

```
Max-Age=604800
```

- `Domain `指定了 `Cookie` 可以送达的主机名
- `Path `指定了一个 `URL `路径，这个路径必须出现在要请求的资源的路径中才可以发送 `Cookie` 首部

```
Path=/docs   # /docs/Web/ 下的资源会带 Cookie 首部
```

- 标记为 `Secure `的 `Cookie `只应通过被`HTTPS`协议加密过的请求发送给服务端

通过上述，我们可以看到`cookie`又开始的作用并不是为了缓存而设计出来，只是借用了`cookie`的特性实现缓存

关于`cookie`的使用如下：

```
document.cookie = '名字=值';
```

关于`cookie`的修改，首先要确定`domain`和`path`属性都是相同的才可以，其中有一个不同得时候都会创建出一个新的`cookie`

```
Set-Cookie:name=aa; domain=aa.net; path=/  # 服务端设置
document.cookie =name=bb; domain=aa.net; path=/  # 客户端设置
```

最后`cookie`的删除，最常用的方法就是给`cookie`设置一个过期的事件，这样`cookie`过期后会被浏览器删除

#### sessionStorage

`sessionStorage `和 `localStorage `使用方法基本一致，唯一不同的是生命周期，一旦页面（会话）关闭，`sessionStorage` 将会删除数据

#### localStorage

特点：

- 生命周期：持久化的本地存储，除非主动删除数据，否则数据是永远不会过期的
- 存储的信息在同一域中是共享的
- 当本页操作（新增、修改、删除）了`localStorage`的时候，本页面不会触发`storage`事件,但是别的页面会触发`storage`事件。
- 大小：5M（跟浏览器厂商有关系）
- `localStorage`本质上是对字符串的读取，如果存储内容多的话会消耗内存空间，会导致页面变卡
- 受同源策略的限制

下面再看看关于`localStorage`的使用

设置

```
localStorage.setItem('username','cfangxu');
```

获取

```
localStorage.getItem('username')
```

获取键名

```
localStorage.key(0) //获取第一个键名
```

删除

```
localStorage.removeItem('username')
```

一次性清除所有存储

```
localStorage.clear()
```

`localStorage` 也不是完美的，它有两个缺点：

- 无法像`Cookie`一样设置过期时间
- 只能存入字符串，无法直接存对象

```js
localStorage.setItem('key', {name: 'value'});
console.log(localStorage.getItem('key')); // '[object, Object]'
```

#### indexedDB

`indexedDB `是一种低级API，用于客户端存储大量结构化数据(包括, 文件/ blobs)。该API使用索引来实现对该数据的高性能搜索

虽然 `Web Storage `对于存储较少量的数据很有用，但对于存储更大量的结构化数据来说，这种方法不太有用。`IndexedDB`提供了一个解决方案

##### 优点：

- 储存量理论上没有上限
- 所有操作都是异步的，相比 `LocalStorage` 同步操作性能更高，尤其是数据量较大时
- 原生支持储存`JS`的对象
- 是个正经的数据库，意味着数据库能干的事它都能干

##### 缺点：

- 操作非常繁琐
- 本身有一定门槛

关于`indexedDB`的使用基本使用步骤如下：

- 打开数据库并且开始一个事务
- 创建一个 `object store`
- 构建一个请求来执行一些数据库操作，像增加或提取数据等。
- 通过监听正确类型的 `DOM` 事件以等待操作完成。
- 在操作结果上进行一些操作（可以在 `request `对象中找到）

关于使用`indexdb`的使用会比较繁琐，大家可以通过使用`Godb.js`库进行缓存，最大化的降低操作难度

#### 8.38 说说 Javascript 数字精度丢失的问题，如何解决

#### 总结：

计算机存储双精度浮点数需要先把十进制数转换为二进制的科学记数法的形式，然后计算机以自己的规则{符号位+(指数位+指数偏移量的二进制)+小数部分}存储二进制的科学记数法

因为存储时有位数限制（64位），并且某些十进制的浮点数在转换为二进制数时会出现无限循环，会造成二进制的舍入操作(0舍1入)，当再转换为十进制时就造成了计算误差

#### 解决方案：

理论上用有限的空间来存储无限的小数是不可能保证精确的，但我们可以处理一下得到我们期望的结果

当你拿到 `1.4000000000000001` 这样的数据要展示时，建议使用 `toPrecision` 凑整并 `parseFloat` 转成数字后再显示，如下：

```
parseFloat(1.4000000000000001.toPrecision(12)) === 1.4  // True
```

封装成方法就是：

```
function strip(num, precision = 12) {
  return +parseFloat(num.toPrecision(precision));
}
```

对于运算类操作，如 `+-*/`，就不能使用 `toPrecision` 了。正确的做法是把小数转成整数后再运算。以加法为例：

```
/**
 * 精确加法
 */
function add(num1, num2) {
  const num1Digits = (num1.toString().split('.')[1] || '').length;
  const num2Digits = (num2.toString().split('.')[1] || '').length;
  const baseNum = Math.pow(10, Math.max(num1Digits, num2Digits));
  return (num1 * baseNum + num2 * baseNum) / baseNum;
}
```

最后还可以使用第三方库，如`Math.js`、`BigDecimal.js`

#### 一个经典的面试题

```
0.1 + 0.2 === 0.3 // false
```

为什么是`false`呢?

先看下面这个比喻

比如一个数 1÷3=0.33333333......

3会一直无限循环，数学可以表示，但是计算机要存储，方便下次取出来再使用，但0.333333...... 这个数无限循环，再大的内存它也存不下，所以不能存储一个相对于数学来说的值，只能存储一个近似值，当计算机存储后再取出时就会出现精度丢失问题

我们知道，在`javascript`语言中，0.1 和 0.2 都转化成二进制后再进行运算

```
// 0.1 和 0.2 都转化成二进制后再进行运算
0.00011001100110011001100110011001100110011001100110011010 +
0.0011001100110011001100110011001100110011001100110011010 =
0.0100110011001100110011001100110011001100110011001100111

// 转成十进制正好是 0.30000000000000004
```

所以输出`false`

再来一个问题，那么为什么`x=0.1`得到`0.1`？

主要是存储二进制时小数点的偏移量最大为52位，最多可以表达的位数是`2^53=9007199254740992`，对应科学计数尾数是 `9.007199254740992`，这也是 JS 最多能表示的精度

它的长度是 16，所以可以使用 `toPrecision(16)` 来做精度运算，超过的精度会自动做凑整处理

```
.10000000000000000555.toPrecision(16)
// 返回 0.1000000000000000，去掉末尾的零后正好为 0.1
```

但看到的 `0.1` 实际上并不是 `0.1`。不信你可用更高的精度试试：

```
0.1.toPrecision(21) = 0.100000000000000005551
```

如果整数大于 `9007199254740992` 会出现什么情况呢？

由于指数位最大值是1023，所以最大可以表示的整数是 `2^1024 - 1`，这就是能表示的最大整数。但你并不能这样计算这个数字，因为从 `2^1024` 开始就变成了 `Infinity`

```
> Math.pow(2, 1023)
8.98846567431158e+307

> Math.pow(2, 1024)
Infinity
```

那么对于 `(2^53, 2^63)` 之间的数会出现什么情况呢？

- `(2^53, 2^54)` 之间的数会两个选一个，只能精确表示偶数
- `(2^54, 2^55)` 之间的数会四个选一个，只能精确表示4个倍数
- ... 依次跳过更多2的倍数

要想解决大数的问题你可以引用第三方库 `bignumber.js`，原理是把所有数字当作字符串，重新实现了计算逻辑，缺点是性能比原生差很多

## 9. ES5、ES6

### 9.1 **ES6里新增的定义变量方式有哪些** 

新增let、const、import、class声明变量的方法

### 9.2 **let const var的区别**

1. var：函数级作用域，存在变量提升，声明前可以调用，允许重新赋值
2. const：块级作用域，不存在变量提升，声明前不可以调用，存在暂时性死区，不允许重新赋值
3. let：块级作用域，不存在变量提升，声明前不可以调用，存在暂时性死区，允许重新赋值

### 9.3 **const定义的变量哪种情况下能修改**

1. 声明复合对象时，数据地址不可修改，对象成员变量可以修改。

2. 若要冻结数据，可以使用freeze。Object.freeze()只支持浅冻结

   ```js
   {
       const obj1 = {
           x:10,
       };
       obj1.y = 20;		//更改成员变量
       const obj2 = Object.freeze({});
       obj2.x = 10;		//常规模式下，该行代码无效，严格模式下会报错
       console.log(obj2.x);//undefined
   }
   ```

### 9.4  <font color=red>ES5/ES6的继承除了写法以外还有什么区别？es5/es6如何继承</font>

>  https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/20
>
>  https://juejin.cn/post/6865562085443502093

- 原型链继承的缺点：

  - 继承的子类原型上的方法创建一定要写在原型链继承之后，不然会被父类实例覆盖。

  - **引用类型**的原型属性被所有实例共享，任何一个实例修改该原型属性值，其他实例该属性值也会被修改

    ```js
    function Parent(){
      this.colors = ['parent'];
    }
    function Children(){}
    
    Children.prototype = new Parent();
    var c1 = new Children();
    var c2 = new Children();
    
    c1.colors.push('children');
    console.log(c1.colors, c2.colors); //[ 'parent', 'children' ] [ 'parent', 'children' ]
    
    ```

- 如何解决原型链继承的缺点：

  

### 9.5 ES6 代码转成 ES5 代码的实现思路是什么

> 参考Babel 的工作原理：
>
> 1. 解析(parse)：将代码字符串解析成抽象语法树——@babel/parser 将源代码解析成 AST。
> 2. 转换(transform)：对抽象语法树进行转换操作。——@babel/generator 将 AST 解码生 JS 代码
> 3. 生成(generate): 根据变换后的抽象语法树再生成代码字符串。——@babel/plugin-transform-? 完成代码转换

1. 将代码字符串解析成抽象语法树，即所谓的 AST
2. 对 AST 进行处理，在这个阶段可以对 ES6 代码进行相应转换，即转成 ES5 代码
3. 根据处理后的 AST 再生成代码字符串

### 9.6 什么是AST？

对于 JavaScript来说，通过**词法分析 -> 语法分析 -> 语法树**，就可以开始解释执行

> 词法分析和语法分析不是完全独立的，而是交错进行的；通常情况下，每取得一个词法记号，就将其送入语法分析器进行分析。
>
> 参考文档：https://zhaomenghuan.js.org/blog/js-ast-principle-reveals.html

- **词法分析**：一段 JS 代码 `var name = 'Hello Flutter'`会被分解为词法单元

  ```js
  [
    {
      "type": "Keyword",
      "value": "var"
    },
    {
      "type": "Identifier",
      "value": "name"
    },
    {
      "type": "Punctuator",
      "value": "="
    },
    {
      "type": "String",
      "value": "'Hello Flutter'"
    },
    {
      "type": "Punctuator",
      "value": ";"
    }
  ]
  ```

- **语法分析**：把词法分析所产生的记号生成语法树，可以使用在线工具生成 AST https://astexplorer.net/

- 代码生成：AST 转换成字符串形式的代码

### 9.7 讲讲set结构

es6方法,Set本身是一个 **构造函数**，它类似于数组，但是成员值都是唯一的。

```js
const set = new Set([1,2,3,4,4])
console.log([...set] )// [1,2,3,4]
console.log(Array.from(new Set([2,3,3,5,6]))); //[2,3,5,6]
```

### 9.8 去除重复数组，去除字符串里面的重复字符

```js
// 去除数组的重复成员
[...new Set(array)]
//去除字符串里面的重复字符
[...new Set('ababbc')].join('')
```

### 9.9 如何拦截变量属性访问

使用proxy。**new Proxy()** 表示生成一个 **Proxy** 实例，**target** 参数表示所要拦截的目标对象，**handler** 参数也是一个对象，用来定制拦截行为。

```js
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});
let obj = Object.create(proxy);
obj.time // 35
```

### 9.10 忽略enumerable为false的属性的四个操作？

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象。描述对象的enumerable属性，称为“可枚举性”，如果该属性为false，就表示某些操作会忽略当前属性。

- for...in循环：只遍历对象自身的和继承的可枚举的属性。
- Object.keys()：返回对象自身的所有可枚举的属性的键名。
- JSON.stringify()：只串行化对象自身的可枚举的属性。
- Object.assign()： 忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性。

### 9.11 属性的五种遍历方法？

- for...in循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。
- Object.keys返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。
- Object.getOwnPropertyNames返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
- Object.getOwnPropertySymbols返回一个数组，包含对象自身的所有 Symbol 属性的键名。
- Reflect.ownKeys返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

### 9.12 ES6中新增的Set、Map两种数据结构怎么理解? WeakSet 与 Set 的区别

`Set`是一种叫做集合的数据结构，`Map`是一种叫做字典的数据结构

- 什么是集合？什么又是字典？

  - 集合：是由一堆无序的、相关联的，且不重复的内存结构【数学中称为元素】组成的组合

  - 字典：是一些元素的集合。每个元素有一个称作key 的域，不同元素的key 各不相同

- 共同点：集合、字典都可以存储不重复的值

- 不同点：集合是以[值，值]的形式存储元素，字典是以[键，值]的形式存储

- Set方法

  - 增删改查的方法

    - add()
    - delete()
    - has()
    - clear()

  - 遍历的方法

    - keys()：返回键名的遍历器
    - values()：返回键值的遍历器
    - entries()：返回键值对的遍历器
    - forEach()：使用回调函数遍历每个成员

  - 应用：

    - 扩展运算符和` Set` 结构相结合实现数组或字符串去重

      ```js
      // 数组
      let arr = [3, 5, 2, 2, 5, 5];
      let unique = [...new Set(arr)]; // [3, 5, 2]
      
      // 字符串
      let str = "352255";
      let unique = [...new Set(str)].join(""); // ""
      ```

    - 实现并集、交集、和差集

      ```js
      let a = new Set([1, 2, 3]);
      let b = new Set([4, 3, 2]);
      
      // 并集
      let union = new Set([...a, ...b]);
      // Set {1, 2, 3, 4}
      
      // 交集
      let intersect = new Set([...a].filter(x => b.has(x)));
      // set {2, 3}
      
      // （a 相对于 b 的）差集
      let difference = new Set([...a].filter(x => !b.has(x)));
      // Set {1}
      ```

- Map方法

  - 增删改查的方法
    - size 属性
    - set()
    - get()
    - has()
    - delete()
    - clear()
  - 遍历
    - keys()：返回键名的遍历器
    - values()：返回键值的遍历器
    - entries()：返回所有成员的遍历器
    - forEach()：遍历 Map 的所有成员

- WeakSet 和WeakMap

  - 在`API`中`WeakSet`与`Set`有两个区别：

    - 没有遍历操作的`API`

    - 没有`size`属性

    - `WeackSet`只能成员只能是引用类型，而不能是其他类型的值

      ```js
      let ws=new WeakSet();
      
      // 成员不是引用类型
      let weakSet=new WeakSet([2,3]);
      console.log(weakSet) // 报错
      
      // 成员为引用类型
      let obj1={name:1}
      let obj2={name:1}
      let ws=new WeakSet([obj1,obj2]); 
      console.log(ws) //WeakSet {{…}, {…}}
      ```

    - `WeakSet `里面的引用只要在外部消失，它在 `WeakSet `里面的引用就会自动消失，即垃圾回收机制不考虑 WeakSet 对该对象的应用

  - 在`API`中`WeakMap`与`Map`有两个区别：

    - 没有遍历操作的`API`

    - 没有`clear`清空方法

    - `WeakMap`只接受对象作为键名（`null`除外），不接受其他类型的值作为键名

      ```js
      const map = new WeakMap();
      map.set(1, 2)
      // TypeError: 1 is not an object!
      map.set(Symbol(), 2)
      // TypeError: Invalid value used as weak map key
      map.set(null, 2)
      // TypeError: Invalid value used as weak map key
      ```

    - `WeakMap`的键名所指向的对象，一旦不再需要，里面的键名对象和所对应的键值对会自动消失，不用手动删除引用

### 9.13 ES6中数组新增了哪些扩展

- 扩展运算符`...`：

  - 主要用于函数调用的时候，将一个数组变为参数序列

    ```js
    function push(array, ...items) {
      array.push(...items);
    }
    
    function add(x, y) {
      return x + y;
    }
    
    const numbers = [4, 38];
    add(...numbers) // 42
    ```

  - 通过扩展运算符实现的是浅拷贝，修改了引用指向的值，会同步反映到新数组

    ```js
    const arr1 = ['a', 'b',[1,2]];
    const arr2 = ['c'];
    const arr3  = [...arr1,...arr2]
    arr1[2][0] = 9999 // 修改arr1里面数组成员值
    console.log(arr[3]) // 影响到arr3,['a','b',[9999,2],'c']
    ```

  - 扩展运算符可以与解构赋值结合起来，用于生成数组，如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错

    ```js
    const [first, ...rest] = [1, 2, 3, 4, 5];
    first // 1
    rest  // [2, 3, 4, 5]
    
    const [first, ...rest] = [];
    first // undefined
    rest  // []
    
    const [first, ...rest] = ["foo"];
    first  // "foo"
    rest   // []
    
    const [...butLast, last] = [1, 2, 3, 4, 5];
    // 报错
    
    const [first, ...middle, last] = [1, 2, 3, 4, 5];
    // 报错
    ```

  - 可以将字符串转为真正的数组

    ```js
    [...'hello']
    // [ "h", "e", "l", "l", "o" ]
    ```

- 构造函数新增的方法

  - Array.from()：

    - 将两类对象转为真正的数组：类似数组的对象和可遍历`（iterable）`的对象（包括 `ES6` 新增的数据结构 `Set` 和 `Map`）

      ```js
      let arrayLike = {
          '0': 'a',
          '1': 'b',
          '2': 'c',
          length: 3
      };
      let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
      ```

    - 还可以接受第二个参数，用来对每个元素进行处理，将处理后的值放入返回的数组

      ```js
      Array.from([1, 2, 3], (x) => x * x)
      // [1, 4, 9]
      ```

  - Array.of()：

    - 用于将一组值，转换为数组

      ```js
      Array.of(3, 11, 8) // [3,11,8]
      ```

    - 没有参数的时候，返回一个空数组

      当参数只有一个的时候，实际上是指定数组的长度

      参数个数不少于 2 个时，`Array()`才会返回由参数组成的新数组

      ```js
      Array() // []
      Array(3) // [, , ,]
      Array(3, 11, 8) // [3, 11, 8]
      ```

- 实例对象新增的方法

  - copyWithin()：将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组
  - find()、findIndex()：用于找出第一个符合条件的数组成员和下标
  - fill()：使用给定值，填充一个数组
  - entries()，keys()，values()：`keys()`是对键名的遍历、`values()`是对键值的遍历，`entries()`是对键值对的遍历
  - includes()：用于判断数组是否包含给定的值
  - flat()，flatMap()：将数组扁平化处理，返回一个新数组，对原数据没有影响

- 数组的空位：

  - 数组的空位指，数组的某一个位置没有任何值
  - ES6 则是明确将空位转为`undefined`，包括`Array.from`、扩展运算符、`copyWithin()`、`fill()`、`entries()`、`keys()`、`values()`、`find()`和`findIndex()`

- 排序稳定性

  - 将`sort()`默认设置为稳定的排序算法

### 9.14 ES6中对象新增了哪些扩展

- 属性的简写

  - 当对象键名与对应值名相等的时候，可以进行简写

    ```js
    const baz = {foo:foo}
    // 等同于
    const baz = {foo}
    ```

  - 方法也能够进行简写

    ```js
    const o = {
      method() {
        return "Hello!";
      }
    };
    
    // 等同于
    
    const o = {
      method: function() {
        return "Hello!";
      }
    }
    ```

  - 在函数内作为返回值

    ```js
    function getPoint() {
      const x = 1;
      const y = 10;
      return {x, y};
    }
    
    getPoint()
    // {x:1, y:10}
    ```

- 属性名表达式

  - ES6 允许字面量定义对象时，将表达式放在括号内

    ```js
    let lastWord = 'last word';
    
    const a = {
      'first word': 'hello',
      [lastWord]: 'world'
    };
    
    a['first word'] // "hello"
    a[lastWord] // "world"
    a['last word'] // "world"
    ```

  - 表达式还可以用于定义方法名

    ```js
    let obj = {
      ['h' + 'ello']() {
        return 'hi';
      }
    };
    
    obj.hello() // hi
    ```

  - 属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串`[object Object]`

    ```js
    const keyA = {a: 1};
    const keyB = {b: 2};
    
    const myObject = {
      [keyA]: 'valueA',
      [keyB]: 'valueB'
    };
    
    myObject // Object {[object Object]: "valueB"}
    ```

  - 属性名表达式与简洁表示法，不能同时使用，会报错

    ```js
    // 报错
    const foo = 'bar';
    const bar = 'abc';
    const baz = { [foo] };
    
    // 正确
    const foo = 'bar';
    const baz = { [foo]: 'abc'};
    ```

- super关键字

  `this`关键字总是指向函数所在的当前对象，ES6 又新增了另一个类似的关键字`super`，指向当前对象的原型对象

  ```js
  const proto = {
    foo: 'hello'
  };
  
  const obj = {
    foo: 'world',
    find() {
      return super.foo;
    }
  };
  
  Object.setPrototypeOf(obj, proto); // 为obj设置原型对象
  obj.find() // "hello"
  ```

- 扩展运算符的应用

  - 在解构赋值中，未被读取的可遍历的属性，分配到指定的对象上面，注意：解构赋值必须是最后一个参数，否则会报错

    ```js
    let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
    x // 1
    y // 2
    z // { a: 3, b: 4 }
    ```

  - 解构赋值是浅拷贝

    ```js
    let obj = { a: { b: 1 } };
    let { ...x } = obj;
    obj.a.b = 2; // 修改obj里面a属性中键值
    x.a.b // 2，影响到了结构出来x的值
    ```

- 属性的遍历

  ES6 一共有 5 种方法可以遍历对象的属性。

  - for...in：循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）
  - Object.keys(obj)：返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名
  - Object.getOwnPropertyNames(obj)：回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名
  - Object.getOwnPropertySymbols(obj)：返回一个数组，包含对象自身的所有 Symbol 属性的键名
  - Reflect.ownKeys(obj)：返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举

  上述遍历，都遵守同样的属性遍历的次序规则：

  - 首先遍历所有数值键，按照数值升序排列
  - 其次遍历所有字符串键，按照加入时间升序排列
  - 最后遍历所有 Symbol 键，按照加入时间升序排

- 对象新增的方法
  - Object.is()：严格判断两个值是否相等，与严格比较运算符（===）的行为基本一致，不同之处只有两个：一是`+0`不等于`-0`，二是`NaN`等于自身
  - Object.assign()：方法用于对象的合并，将源对象`source`的所有可枚举属性，复制到目标对象`target
  - Object.getOwnPropertyDescriptors()：返回指定对象所有自身属性（非继承属性）的描述对象
  - Object.setPrototypeOf()，Object.getPrototypeOf()：用来设置一个对象的原型对象，读取一个对象的原型对象
  - Object.keys()，Object.values()，Object.entries()：返回自身的（不含继承的）所有可遍历（enumerable）属性的键名的数组、键对应值的数组、键值对的数组
  - Object.fromEntries()：用于将一个键值对数组转为对象

### 9.15 ES6中函数新增了哪些扩展? 

- 参数

  - `ES6`允许为函数的参数设置默认值

    ```js
    function log(x, y = 'World') {
      console.log(x, y);
    }
    
    console.log('Hello') // Hello World
    console.log('Hello', 'China') // Hello China
    console.log('Hello', '') // Hello
    ```

  - 参数默认值可以与解构赋值的默认值结合起来使用

    ```js
    function foo({x, y = 5}) {
      console.log(x, y);
    }
    
    foo({}) // undefined 5
    foo({x: 1}) // 1 5
    foo({x: 1, y: 2}) // 1 2
    foo() // TypeError: Cannot read property 'x' of undefined
    ```

  - 参数默认值应该是函数的尾参数，如果不是非尾部的参数设置默认值，实际上这个参数是没发省略的

    ```js
    function f(x = 1, y) {
      return [x, y];
    }
    
    f() // [1, undefined]
    f(2) // [2, undefined]
    f(, 1) // 报错
    f(undefined, 1) // [1, 1]
    ```

- 属性

  - 函数的length长度：`length`将返回没有指定默认值的参数个数

    ```js
    (function (a) {}).length // 1
    (function (a = 5) {}).length // 0
    (function (a, b, c = 5) {}).length // 2
    // rest 参数也不会计入length属性
    (function(...args) {}).length // 0
    // 如果设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数了
    (function (a = 0, b, c) {}).length // 0
    (function (a, b = 1, c) {}).length // 1
    ```

  - name属性：返回该函数的函数名

    ```js
    var f = function () {};
    
    // ES5
    f.name // ""
    // ES6
    f.name // "f"
    
    // 如果将一个具名函数赋值给一个变量，则 name属性都返回这个具名函数原本的名字
    const bar = function baz() {};
    bar.name // "baz"
    
    // Function构造函数返回的函数实例，name属性的值为anonymous
    (new Function).name // "anonymous"
    
    // bind返回的函数，name属性值会加上bound前缀
    function foo() {};
    foo.bind({}).name // "bound foo"
    
    (function(){}).bind({}).name // "bound "
    ```

- 作用域

  一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域

  等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的

  下面例子中，`y=x`会形成一个单独作用域，`x`没有被定义，所以指向全局变量`x`

  ```js
  let x = 1;
  
  function f(y = x) { 
    // 等同于 let y = x  
    let x = 2; 
    console.log(y);
  }
  
  f() // 1
  ```

- 严格模式

  只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错

  ```js
  // 报错
  function doSomething(a, b = a) {
    'use strict';
    // code
  }
  
  // 报错
  const doSomething = function ({a, b}) {
    'use strict';
    // code
  };
  
  // 报错
  const doSomething = (...a) => {
    'use strict';
    // code
  };
  
  const obj = {
    // 报错
    doSomething({a, b}) {
      'use strict';
      // code
    }
  };
  ```

- 箭头函数：使用“箭头”（`=>`）定义函数

  - 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象
  - 不可以当作构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误
  - 不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用 `rest` 参数代替
  - 不可以使用`yield`命令，因此箭头函数不能用作 Generator 函数

### 9.17 怎么理解ES6中 Generator的

- 执行 `Generator` 函数会返回一个遍历器对象，可以依次遍历 `Generator` 函数内部的每一个状态

- `yield`表达式可以暂停函数执行，`next`方法用于恢复函数执行，这使得`Generator`函数非常适合将异步任务同步化

- 形式上，`Generator `函数是一个普通函数，但是有两个特征：

  - `function`关键字与函数名之间有一个星号

  - 函数体内部使用`yield`表达式，定义不同的内部状态，`done`用来判断是否存在下个状态，`value`对应状态值

    ```js
    function* helloWorldGenerator() {
      yield 'hello';
      yield 'world';
      return 'ending';
    }
    
    hw.next()
    // { value: 'hello', done: false }
    hw.next()
    // { value: 'world', done: false }
    hw.next()
    // { value: 'ending', done: true }
    hw.next()
    // { value: undefined, done: true }
    ```

- 通过`next`方法才会遍历到下一个内部状态，其运行逻辑如下：

  - 遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值。

  - 下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式

  - 如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值。

  - 如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`

- 因为`Generator `函数返回`Iterator`对象，因此我们还可以通过`for...of`进行遍历

  ```js
  function* foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
    return 6;
  }
  
  for (let v of foo()) {
    console.log(v);
  }
  // 1 2 3 4 5
  ```



## 10. 设计模式

### 10.1 $emit和$on是什么设计模式

发布订阅模式

### 10.2 说说其他设计模式

## 11 前端模块化

### 11.1 对JS模块化的理解,说说es6和CommonJS的差异

> https://juejin.cn/post/6844903576309858318
> https://github.com/febobo/web-interview/issues/43

- 如果没有模块化，我们代码会怎样？

  - 变量和方法不容易维护，容易污染全局作用域

  - 加载资源的方式通过script标签从上到下。

  - 依赖的环境主观逻辑偏重，代码较多就会比较复杂。

  - 大型项目资源难以维护，特别是多人合作的情况下，资源的引入会让人奔溃

  因此，需要一种将`JavaScript`程序模块化的机制，如

  - CommonJs (典型代表：node.js早期)

  - AMD (典型代表：require.js)

  - CMD (典型代表：sea.js)

  #### AMD

  `Asynchronous ModuleDefinition`（AMD），异步模块定义，采用异步方式加载模块。所有依赖模块的语句，都定义在一个回调函数中，等到模块加载完成之后，这个回调函数才会运行

  ```js
  /** main.js 入口文件/主模块 **/
  // 首先用config()指定各模块路径和引用名
  require.config({
    baseUrl: "js/lib",
    paths: {
      "jquery": "jquery.min",  //实际路径为js/lib/jquery.min.js
      "underscore": "underscore.min",
    }
  });
  // 执行基本操作
  require(["jquery","underscore"],function($,_){
    // some code here
  });
  ```

  #### CommonJS

  `CommonJS` 是一套 `Javascript` 模块规范，用于服务端

  其有如下特点：

  - 所有代码都运行在模块作用域，不会污染全局作用域
  - 模块是同步加载的，即只有加载完成，才能执行后面的操作
  - 模块在首次执行后就会缓存，再次加载只返回缓存结果，如果想要再次执行，可清除缓存
  - `require`返回的值是被输出的值的拷贝，模块内部的变化也不会影响这个值

  ```js
  // a.js
  module.exports={ foo , bar}
  
  // b.js
  const { foo,bar } = require('./a.js')
  ```

  既然存在了`AMD`以及`CommonJs`机制，`ES6`的`Module`又有什么不一样？

  ES6 在语言标准的层面上，实现了`Module`，即模块功能，完全可以取代 `CommonJS `和 `AMD `规范，成为浏览器和服务器通用的模块解决方案

  `CommonJS` 和` AMD` 模块，都只能在运行时确定这些东西。比如，`CommonJS `模块就是对象，输入时必须查找对象属性

  ```js
  // CommonJS模块
  let { stat, exists, readfile } = require('fs');
  
  // 等同于
  let _fs = require('fs');
  let stat = _fs.stat;
  let exists = _fs.exists;
  let readfile = _fs.readfile;
  ```

  `ES6`设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量

  ```js
  // ES6模块
  import { stat, exists, readFile } from 'fs';
  ```

  上述代码，只加载3个方法，其他方法不加载，即 `ES6` 可以在编译时就完成模块加载

  由于编译加载，使得静态分析成为可能。包括现在流行的`typeScript`也是依靠静态分析实现功能

- **Es6 module使用**

  模块功能主要由两个命令构成：

  - `export`：用于规定模块的对外接口
  - `import`：用于输入其他模块提供的功能

  #### export

  一个模块就是一个独立的文件，该文件内部的所有变量，外部无法获取。如果你希望外部能够读取模块内部的某个变量，就必须使用`export`关键字输出该变量

  ```js
  // profile.js
  export var firstName = 'Michael';
  export var lastName = 'Jackson';
  export var year = 1958;
  
  或 
  // 建议使用下面写法，这样能瞬间确定输出了哪些变量
  var firstName = 'Michael';
  var lastName = 'Jackson';
  var year = 1958;
  
  export { firstName, lastName, year };
  ```

  输出函数或类

  ```js
  export function multiply(x, y) {
    return x * y;
  };
  ```

  通过`as`可以进行输出变量的重命名

  ```js
  function v1() { ... }
  function v2() { ... }
  
  export {
    v1 as streamV1,
    v2 as streamV2,
    v2 as streamLatestVersion
  };
  ```

  #### import

  使用`export`命令定义了模块的对外接口以后，其他 JS 文件就可以通过`import`命令加载这个模块

  ```js
  // main.js
  import { firstName, lastName, year } from './profile.js';
  
  function setName(element) {
    element.textContent = firstName + ' ' + lastName;
  }
  ```

  同样如果想要输入变量起别名，通过`as`关键字

  ```js
  import { lastName as surname } from './profile.js';
  ```

  当加载整个模块的时候，需要用到星号`*`

  ```js
  // circle.js
  export function area(radius) {
    return Math.PI * radius * radius;
  }
  
  export function circumference(radius) {
    return 2 * Math.PI * radius;
  }
  
  // main.js
  import * as circle from './circle';
  console.log(circle)   // {area:area,circumference:circumference}
  ```

  输入的变量都是只读的，不允许修改，但是如果是对象，允许修改属性

  ```
  import {a} from './xxx.js'
  
  a.foo = 'hello'; // 合法操作
  a = {}; // Syntax Error : 'a' is read-only;
  ```

  不过建议即使能修改，但我们不建议。因为修改之后，我们很难差错

  `import`后面我们常接着`from`关键字，`from`指定模块文件的位置，可以是相对路径，也可以是绝对路径

  ```
  import { a } from './a';
  ```

  如果只有一个模块名，需要有配置文件，告诉引擎模块的位置

  ```
  import { myMethod } from 'util';
  ```

  在编译阶段，`import`会提升到整个模块的头部，首先执行

  ```
  foo();
  
  import { foo } from 'my_module';
  ```

  多次重复执行同样的导入，只会执行一次

  ```
  import 'lodash';
  import 'lodash';
  ```

  上面的情况，大家都能看到用户在导入模块的时候，需要知道加载的变量名和函数，否则无法加载

  如果不需要知道变量名或函数就完成加载，就要用到`export default`命令，为模块指定默认输出

  ```
  // export-default.js
  export default function () {
      console.log('foo');
  }
  ```

  加载该模块的时候，`import`命令可以为该函数指定任意名字

  ```
  // import-default.js
  import customName from './export-default';
  customName(); // 'foo'
  ```

#### 动态加载

允许您仅在需要时动态加载模块，而不必预先加载所有模块，这存在明显的性能优势

这个新功能允许您将`import()`作为函数调用，将其作为参数传递给模块的路径。 它返回一个 `promise`，它用一个模块对象来实现，让你可以访问该对象的导出

```
import('/modules/myModule.mjs')
  .then((module) => {
    // Do something with the module.
  });
```

#### 复合写法

如果在一个模块之中，先输入后输出同一个模块，`import`语句可以与`export`语句写在一起

```
export { foo, bar } from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };
```

同理能够搭配`as`、`*`搭配使用

#### 使用场景

如今，`ES6`模块化已经深入我们日常项目开发中，像`vue`、`react`项目搭建项目，组件化开发处处可见，其也是依赖模块化实现

`vue`组件

```
<template>
  <div class="App">
      组件化开发 ---- 模块化
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
</script>
```

`react`组件

```
function App() {
  return (
    <div className="App">
		组件化开发 ---- 模块化
    </div>
  );
}

export default App;
```

包括完成一些复杂应用的时候，我们也可以拆分成各个模块

#### **es6和CommonJS的差异**

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。CommonJS 模块就是对象；即在输入时是先加载整个模块，生成一个对象，然后再从这个对象上面读取方法，这种加载称为“运行时加载”。ES6 模块不是对象，而是通过 export 命令显式指定输出的代码，import时采用静态命令的形式。即在import时可以指定加载某个输出值，而不是加载整个模块，这种加载称为“编译时加载”。
- CommonJS 是同步导入，因为用于服务端，文件都在本地，同步导入即使卡住主线程影响也不大。而es6模块是异步导入，因为用于浏览器，需要下载文件，如果也采用同步导入会对渲染有很大影响





## 12 js事件循环机制

- 程序开始执行之后，主程序则开始执行 **同步任务**，碰到 **异步任务** 就把它放到任务队列中,等到同步任务全部执行完毕之后，js引擎便去查看任务队列有没有可以执行的异步任务，将异步任务转为同步任务，开始执行，执行完同步任务之后继续查看任务队列，这个过程是一直 **循环** 的，因此这个过程就是所谓的 **事件循环**，其中任务队列也被称为事件队列。通过一个任务队列，单线程的js实现了异步任务的执行，给人感觉js好像是多线程的。