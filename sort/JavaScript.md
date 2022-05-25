# JS基础

## 1. 闭包

```js
var result = []
var a = 3
var total = 0
function foo(a) {
  var i = 0
  for (; i < 3; i++) {
    result[i] = function () {
      total += i * a
      console.log(total)
    }
  }
}
foo(1)
//下面都输出什么
result[0]()
result[1]()
result[2]()
```

## 2. this指向

### 2.1 call模拟实现

```js
// 此处不能使用es6 箭头函数，没有this和arguments
Function.prototype.myCall = function(context = window){
  context.fn = this;
  const args = [...arguments].slice(1);
  const res = context.fn(...args);
  delete context.fn;
  return res;
}
```

### 2.2 apply模拟实现

```js
Function.prototype.myApply = function(context = window){
  context.fn = this;
  const res = arguments[1] ? context.fn(...arguments[1]) : context.fn();
  delete context.fn;
  return res;
}
```

### 2.3 <font color=red>bind模拟实现</font>

> https://www.cnblogs.com/echolun/p/12178655.html

```js
Function.prototype.bind_ = function (obj) {
    var args = Array.prototype.slice.call(arguments, 1);
    var fn = this;
    //创建中介函数
    var fn_ = function () {};
    var bound = function () {
        var params = Array.prototype.slice.call(arguments);
        //通过constructor判断调用方式，为true this指向实例，否则为obj
        fn.apply(this.constructor === fn ? this : obj, args.concat(params));
        console.log(this);
    };
    fn_.prototype = fn.prototype;
    bound.prototype = new fn_();
    return bound;
};

Function.prototype.myBind = (context = window) => {
    if (typeof this != 'function') {
        throw new TypeError('error');
    }
    const _this = this;
    let args = [...arguments].slice(1);
    return function F() {
        if (this instanceof F){
            return new _this(...args, ...arguments);
        }

        return _this.apply(context, args.concat(arguments));
    }
}
```

### 2.4 new 模拟实现

```js
const _new(fn, ...args){
  const obj = Object.create(fn.prototype);
  const ret = fn.apply(obj, args);
  return ret instanceof Object ? ret : obj;
}
```

### 2.5 function与箭头函数的区别

1. function可以显式命名，箭头函数是隐式命名
2. 箭头函数没有prototype属性，不能使用new关键词，不能写成构造函数
3. 箭头函数不会创建新的作用域，不会改变this指向

## 3. 原型对象

### 3.1 实例属性和原型属性的关系

> 参考：https://www.cnblogs.com/cirry/p/13303153.html

```js
function Person() {}
// var Person = {
//   prototype: {
//     // Person
//     constructor: function(){}
//   }
// }
Person.prototype.name = "ccc";
Person.prototype.age = 18;
Person.prototype.sayName = function() {
  console.log(this.name);
};

// var Person = {
//   prototype: {
//     name: "ccc",
//     age: 18,
//     // Person
//     constructor: function(){},
//     sayName: function(){
//       console.log(this.name);
//     }
//   }
// }

var person1 = new Person();
var person2 = new Person();

// var person1 = {
//   prototype: {
//     name: "ccc",
//     age: 18,
//     // Person
//     constructor: function(){},
//     sayName: function(){
//       console.log(this.name);
//     }
//   }
// }

// var person2 = {
//   prototype: {
//     name: "ccc",
//     age: 18,
//     // Person
//     constructor: function(){},
//     sayName: function(){
//       console.log(this.name);
//     }
//   }
// }

person1.name = 'www'      // 在person1中添加一个name属性
// var person1 = {
//   name: 'www',
//   prototype: {
//     name: "ccc",
//     age: 18,
//     // Person
//     constructor: function(){},
//     sayName: function(){
//       console.log(this.name);
//     }
//   }
// }

person1.sayName()      // --> 'www'————'来自实例'
person2.sayName()      // --> 'ccc'————'来自原型'

console.log(person1.hasOwnProperty('name'))      // --> true
console.log(person2.hasOwnProperty('name'))      // --> false

delete person1.name      // --> 删除person1中新添加的name属性

// var person1 = {
//   prototype: {
//     name: "ccc",
//     age: 18,
//     // Person
//     constructor: function(){},
//     sayName: function(){
//       console.log(this.name);
//     }
//   }
// }
person1.sayName()      // -->'ccc'————'来自原型'
```

### 3.2 prototype和\_proto\_的关系

> **JavaScript** 中任意对象都有一个内置属性 **[[Prototype]]** ，支持通过`__proto__`来访问

```js
function Parent() {
  this.name = "parent";
}
Parent.prototype.getName = function () {
  return this.name;
};

// Parent = {
//   name: "parent",
//   prototype: {
//     // Parent
//     constructor: function(){
//       this.name = "parent";
//     },
//     getName: function () {
//       return this.name;
//     }
//   },
// }

function Children(name) {
  this.cName = "children";
}

// Children = {
//   cName: "children",
//   prototype: {
//       // Children
//       constructor: function(){
//           this.cName = "children";
//       }
//   }
// }

// 通过原型链实现继承Children.prototype.__proto__ === Parent.prototype
Children.prototype = new Parent();
// Children = {
//   cName: "children",
//   prototype: {
//     // Children
//     constructor: function(){
//         this.cName = "children";
//     },
//     __proto__: {
//       name: "parent",
//       // Parent
//       constructor: function(){
//           this.name = "parent";
//       },
//       getName: function () {
//           return this.name;
//       }
//     }
//   }
// }


//重写方法一定要写在继承之后不然会被覆盖
Children.prototype.getName = function () {
  return this.cName;
};

// Children = {
//   cName: "children",
//   prototype: {
//     // Children
//     constructor: function(){
//         this.cName = "children";
//     },
//     getName: function () {
//       return this.cName;
//     },
//     __proto__: {
//       name: "parent",
//       // Parent
//       constructor: function(){
//           this.name = "parent";
//       },
//       getName: function () {
//           return this.name;
//       }
//     }
//   }
// }

var c = new Children();
console.log(c.getName()); //children
```

### 3.3 JS继承的实现方式

> https://www.cnblogs.com/humin/p/4556820.html

- 原型链继承： 将父类的实例作为子类的原型

  - 缺点：
    - 要想为子类新增属性和方法，必须要在`new Animal()`这样的语句之后执行，不能放到构造器中
    - 无法实现多继承
    - 来自原型对象的所有属性被所有实例共享
    - 创建子类实例时，无法向父类构造函数传参

  ```js
  function Animal(name){
    this.name = name || 'animal';
    this.sleep = function(){
      console.log(this.name + ' is sleeping');
    }
  }
  
  Animal.prototype.eat = function(food){
     console.log(this.name + ' eat ' + food);
  }
  
  // const Animal = {
  //   prototype: {
  //     constructor: function(name){
  //       this.name = name || 'animal';
  //     },
  //     sleep: function(){
  //       console.log(this.name + 'is sleeping');
  //     },
  //     eat: function(food){
  //       console.log(this.name + ' eat ' + food);
  //     }
  //   }
  // }
  
  
  function Cat(){}
  
  Cat.prototype = new Animal();
  Cat.prototype.name = 'cat';
  
  // const Cat = {
  //   prototype: {
  //     constructor: function(){},
  //     name: 'cat',
  //     _proto_: {
  //       constructor: function(name){
  //         this.name = name || 'animal';
  //       },
  //       sleep: function(){
  //         console.log(this.name + 'is sleeping');
  //       },
  //       eat: function(food){
  //         console.log(this.name + ' eat ' + food);
  //       }
  //     }
  //   }
  // }
  
  var cat = new Cat();
  console.log(cat.name); // cat
  console.log(cat.eat('fish')); // cat eat fish
  console.log(cat.sleep()); // cat is sleeping
  console.log(cat instanceof Animal); //true 
  console.log(cat instanceof Cat); //true 
  ```

- 构造继承：使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类

  - 特点：
    - 解决了子类实例共享父类引用属性的问题
    - 创建子类实例时，可以向父类传递参数
    - 可以实现多继承（call多个父类对象）
  - 缺点：
    - 实例并不是父类的实例，只是子类的实例
    - 只能继承父类的实例属性和方法，不能继承原型属性/方法
    - 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

  ```js
  function Cat(name){
    Animal.call(this);
    this.name = 'cat';
  }
  
  var cat = new Cat();
  console.log(cat.name); // cat
  // console.log(cat.eat('fish')); // 会报错，Uncaught TypeError: cat.eat is not a function
  console.log(cat.sleep()); // cat is sleeping
  console.log(cat instanceof Animal); //false 
  console.log(cat instanceof Cat); //true 
  ```

- 实例继承：为父类实例添加新特性，作为子类实例返回

  - 特点：
    - 不限制调用方式，不管是`new 子类()`还是`子类()`,返回的对象具有相同的效果
  - 缺点：
    - 实例是父类的实例，不是子类的实例
    - 不支持多继承

  ```js
  function Cat(name){
    const instance = new Animal();
    instance.name = 'cat';
    return instance;
  }
  
  var cat = new Cat();
  console.log(cat.name); // cat
  console.log(cat.sleep()); // cat is sleeping
  console.log(cat instanceof Animal); // true
  console.log(cat instanceof Cat); // false
  ```

- 拷贝继承

  - 特点：
    1. 支持多继承
  - 缺点：
    1. 效率较低，内存占用高（因为要拷贝父类的属性）
    2. 无法获取父类不可枚举的方法（不可枚举方法，不能使用for in 访问到）

  ```js
  function Cat(name){
    var animal = new Animal();
    for(var p in animal){
      Cat.prototype[p] = animal[p];
    }
    this.name = name || 'Tom';
  }
  
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  console.log(cat instanceof Animal); // false
  console.log(cat instanceof Cat); // true
  ```

- 组合继承：通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用

  - 特点：
    1. 可以继承实例属性/方法，也可以继承原型属性/方法
    2. 既是子类的实例，也是父类的实例
    3. 不存在引用属性共享问题
    4. 可传参
    5. 函数可复用
  - 缺点：
    1. 调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了）

  ```js
  function Cat(name){
    Animal.call(this);
    this.name = name || 'Tom';
  }
  Cat.prototype = new Animal();
  Cat.prototype.constructor = Cat;
  
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  console.log(cat instanceof Animal); // true
  console.log(cat instanceof Cat); // true
  
  ```

- 寄生组合继承：通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

  ```js
  function Cat(name){
    Animal.call(this);
    this.name = name || 'Tom';
  }
  Cat.prototype.constructor = Cat; // 需要修复下构造函数
  
  (function(){
    // 创建一个没有实例方法的类
    var Super = function(){};
    Super.prototype = Animal.prototype;
    //将实例作为子类的原型
    Cat.prototype = new Super();
  })();
  
  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  console.log(cat instanceof Animal); // true
  console.log(cat instanceof Cat); //true
  ```

  

## 4. 作用域链

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



## 6. 异步

### 6.1 **JS 异步解决方案的发展历程以及优缺点**

- 回调函数callback：

  缺点：

  - **回调地狱**
  - **不能用 try catch 捕获错误**
  - **不能 return**

  ```js
  // 假定有两个函数f1和f2，后者等待前者的执行结果。
  f1();
  f2();
  
  
  function f1(callback){
    setTimeout(function () {
      // f1的任务代码
      callback();
    }, 1000);
  }
  f1(f2);
  ```

- promise：

  优点：解决回调地域问题

  缺点：错误需要回调函数捕捉

  ```js
  f1().then(f2).then(f3);
  ```

- Generator生成器

  ```js
  // 第一次调用：Generator 函数开始执行，直到遇到第一个yield表达式为止。g.next()返回一个包含yield后的值的对象{ value: 'a', done: false }。此时并不会发生赋值赋值，只会执行yield 'a'就不会执行了。
  // 第二次调用：此时调用g.next(1)后函数会继续向下执行代码直到执行到第二个yield就挂起。这时候就会继续之前的赋值行为， g.next(1)里的1会作为「上一个」yield 'a'的返回值赋值给a，所以此时a=1。
  // 第三次调用：和上一次一样b接收到了g.next(2)里的返回值2。此时g.next()对象里的done: true表示生成器里的yield操作已经结束。
  function* ab(){    
    let a = yield 'a'    
    console.log(`a:${a}`)    
    let b = yield 'b'    
    console.log(`b:${b}`)
  }
  let g = ab()
  console.log(g.next())   // { value: 'a', done: false }
  console.log(g.next(1))  // a: 1 { value: 'b', done: false }
  console.log(g.next(2))  // b: 2 { value: undefined, done: true }
  ```

- async/await

  优点：**不用像 Promise 写一大堆 then 链，处理了回调地狱的问题**

  ```js
  async function test() {
    // 以下代码没有依赖性的话，完全可以使用 Promise.all 的方式
    // 如果有依赖性的话，其实就是解决回调地狱的例子了
    await fetch('XXX1')
    await fetch('XXX2')
    await fetch('XXX3')
  }
  ```

### 6.2 **你了解过Promise吗**

- Promise的出现是为了解决什么问题
  - Promise是解决异步编程的一种方案。以前我们处理异步操作，一般都是通过回调函数来处理，会存在回调地域问题
  - Promise的出现，能够让异步编程变得更加可观，把异步操作按照同步操作的流程表达出来，避免层层嵌套的回调函数。
- Promise的状态
  - Promise对象有三种状态，进行中`pending`、完成成功`fulfilled`、失败`rejected`
  - Promise的状态一旦确定了，就不会再更改了
- promise不足的地方
  - 如果没有执行捕获错误的函数（如下述说的catch，then的第二个参数），则Promise内部发生的错误（虽然会报错但）是无法传递到Promise外部代码上的，因此外部脚本并不会因为错误而导致不继续执行下去。
  - 一旦新建了，就无法中断它的操作。不像`setTimeout`那样，我还可以使用`clearTimeout`取消掉。

### 6.3 <font color=red>**async/await 如何通过同步的方式实现异步**</font>

> https://segmentfault.com/a/1190000038985579

Async/Await就是一个**自执行**的generate函数。利用generate函数的特性把异步的代码写成“同步”的形式。

```js
// Generator
run(function*() {
  const res1 = yield Promise.resolve({a: 1});
  console.log(res1);

  const res2 = yield Promise.resolve({b: 2});
  console.log(res2);
});

// async/await
const aa = async ()=>{
  const res1 = await Promise.resolve({a: 1});
  console.log(res1);

  const res2 = await Promise.resolve({b: 2});
  console.log(res2);

  return 'done'；
}
const res = aa();
```



### 6.4 <font color=red> **setTimeout、Promise、Async/Await 的区别**</font>

- settimeout的回调函数放到宏任务队列里，等到执行栈清空以后执行；
- promise new的时候会立即执行里面的代码；promise.then里的回调函数会放到相应宏任务的微任务队列里，等宏任务里面的同步代码执行完再执行；
- async函数表示函数里面可能会有异步方法，await后面跟一个表达式，async方法执行时，遇到await会立即执行表达式，然后把表达式后面的代码放到微任务队列里，让出执行栈让同步代码先执行。

- promise、setTimeout的执行顺序

  ```js
  const promise = new Promise((resolve, reject) => {
    console.log(1);
    resolve(5);
    console.log(2);
  }).then(val => {
    console.log(val);
  });
  
  promise.then(() => {
    console.log(3);
  });
  
  console.log(4);
  
  setTimeout(function() {
    console.log(6);
  });
  // 执行结果: 124536
  
  
  console.log("script start");
  async function async1() {
    await async2();
    console.log("async1 end");
  }
  async function async2() {
    console.log("async2 end");
  }
  async1();
  setTimeout(function () {
    console.log("setTimeout");
  }, 0);
  new Promise(resolve => {
    console.log("Promise");
    resolve();
  })
    .then(function () {
      console.log("promise1");
    })
    .then(function () {
      console.log("promise2");
    });
  console.log("script end");
  //script start / async2 end / promise / script end/  async1 end / promise1 / promise2 / setTimeout
  
  
  
  function wait() {
  	return new Promise(resolve =>
  		setTimeout(resolve, 10 * 1000)
  	)
  }
  
  async function main() {
  	console.time();
  	const x = await wait(); // 每个都是都执行完才结,包括setTimeout（10*1000）的执行时间
  	const y = await wait(); // 执行顺序 x->y->z 同步执行，x 与 setTimeout 属于同步执行
  	const z = await wait();
  	console.timeEnd(); // default: 30099.47705078125ms
  	
  	console.time();
  	const x1 = wait(); // x1,y1,z1 同时异步执行， 包括setTimeout（10*1000）的执行时间
  	const y1 = wait(); // x1 与 setTimeout 属于同步执行
  	const z1 = wait();
  	await x1;
  	await y1;
  	await z1;
  	console.timeEnd(); // default: 10000.67822265625ms
  	
  	console.time();
  	const x2 = wait(); // x2,y2,z2 同步执行，但是不包括setTimeout（10*1000）的执行时间
  	const y2 = wait(); // x2 与 setTimeout 属于异步执行
  	const z2 = wait();
  	x2,y2,z2;
  	console.timeEnd(); // default: 0.065185546875ms
  }
  main();
  
  ```

​	

### 6.5 **用 setTimeout 实现 setInterval，阐述实现的效果与 setInterval 的差异**

```js
// 不能清除计时器
var mySetInterval = function(){
  // arguments[0]为function，arguments[1]为间隔毫秒数
  const args = arguments;
  const timer = setTimeout(() => {
    args[0]();
    // callee属性是一个指针，指向拥有这个arguments对象的函数，即mySetInterval
    args.callee(...args);
  }, args[1]);
  return timer;
};

var time = mySetInterval(()=> {
  console.log(1);
},1000);
```

```js
function mySetInterval() {
  mySetInterval.timer = setTimeout(() => {
    arguments[0]()
    mySetInterval(...arguments)
  }, arguments[1])
}

mySetInterval.clear = function() {
  clearTimeout(mySetInterval.timer)
}

mySetInterval(() => {
  console.log(11111)
}, 1000)

setTimeout(() => {
  // 5s 后清理
  mySetInterval.clear()
}, 5000)
```

### 6.6 **promise实现ajax**

> XMLHttpRequest：
>
> 1. onreadystatechange事件：每次readystate发生变化就会被调用一次
> 2. readystate属性：0—请求未初始化；1—请求已建立链接；2—请求已接收；3—请求处理中；4—请求已完成
> 3. status属性：200—"OK"；404—"未找到页面"

```JS
const ajax = (method, url, data) => {
  	// 将data对象转化为&拼接的键值对字符串
  	data = transfer(data);
    let request = new XMLHttpRequest();
    return new Promise((resolve, reject) => {
        request.onreadystatechange = () => {
            if(request.readyState === 4) {
                if(request.status === 200) {
                    resolve(request.responseText);
                } else {
                    reject(request.status);
                }
            }
        }
        if(method === 'get'){
          request.open(method, url+'?'+ data, true);
          request.send(data);
        }else{
          request.open(method, url, true);
          request.send(data);
        }
    });
}
ajax('GET', '/api/categories').then(function (text) {   // 如果AJAX成功，获得响应内容
    log.innerText = text;
}).catch(function (status) { // 如果AJAX失败，获得响应代码
    log.innerText = 'ERROR: ' + status;
});
```

### 6.7 async function A() { return 1 }，当我们调用 A 的时候，返回结果是什么

async函数一定会返回一个promise对象。如果一个async函数的返回值看起来不是promise，那么它将会被隐式地包装在一个promise中。

```js
async function A() { 
  return 1;
}

// 等价于
async function A() { 
  return new Promise.resolve(1); 
}
```

### 6.8 实现一个 sleep 函数,比如 sleep(1000) 意味着等待1000毫秒

1. Promise

   ```js
   const sleep = function(time){
     return new Promise(resolve => setTimeout(resolve, time));
   }
   
   sleep(1000).then(() => {
     console.log(1);
   });
   ```

2. Generator

   ```js
   function* sleep(time){
     yield new Promise(resolve => setTimeout(resolve, time));
   }
   
   sleep(1000).next().value.then(() => {
     console.log(1);
   });
   ```

3. async

   ```js
   function sleep(time){
     return new Promise(resolve => setTimeout(resolve, time));
   }
   
   async function output(){
     await sleep(1000);
     console.log(1);
   }
   
   output();
   ```

4. setTimeout

   ```js
   function sleep(func, time){
     setTimeout(func, time);
   }
   sleep(() => {
     console.log(1);
   }, 1000);
   ```

   


### 6.9 介绍下 Promise.all 使用、原理实现及错误处理

- Promise.all 使用：

  - Promise.all方法接受一个数组作为参数，p1、p2、p3都是 Promise 实例，如果不是，就会先调用下面讲到的Promise.resolve方法，将参数转为 Promise 实例，再进一步处理。
  - Promise.all方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例。
  - 使用Promise.all()生成的Promise对象（p）的状态是由数组中的Promise对象（p1,p2,p3）决定
    - 如果所有的Promise对象都变成fullfilled状态的话，生成的Promise对象（p）也会变成fullfilled状态，产生的结果会组成一个数组返回给传递给p的回调函数；
    - 如果有一个Promise对象变为rejected状态的话，p也会变成rejected状态，第一个被rejected的对象的返回值会传递给p的回调函数。

- **原理实现（实现一个 Promise.all）**：

  ```js
  function promiseAll(promises){
       return new Promise(function(resolve,reject){
              if(!Array.isArray(promises)){
               return reject(new TypeError("argument must be anarray"))
             }
      var countNum=0;
      var promiseNum=promises.length;
      var resolvedvalue=new Array(promiseNum);
      for(var i=0;i<promiseNum;i++){
        (function(i){
           Promise.resolve(promises[i]).then(function(value){
              countNum++;
             resolvedvalue[i]=value;
            if(countNum===promiseNum){
                return resolve(resolvedvalue)
            }
         },function(reason){
          return reject(reason)
        )
       })(i)
      }
  })
  }
  var p1=Promise.resolve(1),
  p2=Promise.resolve(2),
  p3=Promise.resolve(3);
  promiseAll([p1,p2,p3]).then(function(value){
  	console.log(value)
  });
  ```

- 错误处理

  Promise.all()方法生成的Promise对象也会有一个catch方法来捕获错误处理，但是如果数组中的Promise对象变成rejected状态时，并且这个对象还定义了catch的方法，那么rejected的对象会执行自己的catch方法，并且返回一个状态为fullfilled的Promise对象，Promise.all()生成的对象会接受这个Promise对象，不会返回rejected状态。

### 6.10 函数实现`arr`的串行调用

- 题目：写一个函数实现`arr`的串行调用，让`arr`依次输出run1/run2/run3。必须按照`arr`的顺序执行，需要用`Promise`的状态去实现先后顺序（`resolve`或者`reject`函数执行状态改变后才能执行下一个）

  ```js
  let arr = [()=>{
  	return new Promise(res=>{
  		console.log("run1", Date.now());
  		res()
  	})
  },()=>{
  	return new Promise(res=>{
  		console.log("run2", Date.now());
  		res()
  	})
  },()=>{
  	return new Promise(res=>{
  		console.log("run3", Date.now());
  		res()
  	})
  }];
  ```

- 题解

  - 使用async

    ```js
    async function p(arr){
      // 遍历arr用for循环并没有用forEach。因为forEach里就是另一个函数隔开了外层的async函数，
        for(let v of arr){
            await v;
        }
    }
    ```

  - 利用`Promise.resolve()`

    ```js
    function p(arr){
        let res = Promise.resolve();
        arr.forEach( v => {
    		res = res.then(() => v());
        });
    }
    ```

  - 利用`reduce`函数的性质

    ```js
    function p(arr){
        arr.reduce( (pre, cur) => {
    		return pre.then(() => cur())
        }, Promise.resolve());
    }
    ```

    

### 6.11 设计并实现 Promise.race()

Promse.race就是赛跑的意思，意思就是说，Promise.race([p1, p2, p3])里面哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态。

```js
const _race = (p)=>{
	return new Promise((resolve, reject)=>{
		p.forEach((item)=>{
			Promise.resolve(item).then(resolve, reject)
		})
	})
}
```

### 6.12  <font color=red>手写实现Promise</font>

> https://juejin.cn/post/6844903625769091079

## 7. 事件循环

## 8. 基础

### 8.1 **undefined和null的区别**

1. typeof得到的结果不一样：
   1. typeof undefined为undefined
   2. typeof null为object
2. 含义不一样：
   1. undefined表示已声明但没有初始化值
   2. null表示该值后面会赋值对象，初始时先赋值为null
3. 转为数值的结果不一样：
   1. Number(undefined)等于NaN
   2. Number(null)等于0

### 8.2 **JavaScript中怎么判断相等**

1. 相等运算符==：先转换类型再比较

   |           | Undefined | Null    | Number                | String                        | Boolean                         | Object                          |
   | :-------- | --------- | ------- | --------------------- | ----------------------------- | ------------------------------- | ------------------------------- |
   | Undefined | `true`    | `true`  | `false`               | `false`                       | `false`                         | `IsFalsy(B)`                    |
   | Null      | `true`    | `true`  | `false`               | `false`                       | `false`                         | `IsFalsy(B)`                    |
   | Number    | `false`   | `false` | `A === B`             | `A === ToNumber(B)`           | `A=== ToNumber(B)`              | `A== ToPrimitive(B)`            |
   | String    | `false`   | `false` | `ToNumber(A) === B`   | `A === B`                     | `ToNumber(A) === ToNumber(B)`   | `ToPrimitive(B) == A`           |
   | Boolean   | `false`   | `false` | `ToNumber(A) === B`   | `ToNumber(A) === ToNumber(B)` | `A === B`                       | `ToNumber(A) == ToPrimitive(B)` |
   | Object    | `false`   | `false` | `ToPrimitive(A) == B` | `ToPrimitive(A) == B`         | `ToPrimitive(A) == ToNumber(B)` | `A === B`                       |

2. 严格比较运算符===：先判断类型，如果不是同一类型直接为false。**NaN不等于自身，以及+0等于-0**

3. Object.is()：与严格比较运算符（===）基本一致，不同之处只有两个：**一是+0不等于-0，二是NaN等于自身。**

### 8.3 **JS的数据类型有哪些**

1. 基本数据类型（6种）：null、undefined、string、number、boolean、symbol（ ES6）
2. 引用数据类型：object

### 8.4 **symbol的应用场景**

1. 定义常量

   ```js
   log.levels = {
       DEBUG: Symbol('debug'),
       INFO: Symbol('info'),
       WARN: Symbol('warn'),
   };
   log(log.levels.DEBUG, 'debug message');
   log(log.levels.INFO, 'info message');
   
   // 假如我们使用数字来代替上面的Symbol
   // 在新增一个 ERROR级别的时候，我们还要小心不能跟之前的值（1，2，3中的任意一个）相同；
   log.levels = {
       DEBUG: 1,
       INFO: 2,
       WARN: 3,
   };
   ```

2. 在对象中存放自定义元数据，作为对象内部的私有属性

3. 作为key使用

### 8.5 自执行函数（IIFE）的作用是什么

- 放在 IIFE 里面的变量，并不会影响到其他外层的变量，也不会被外层的变量影响。

### 8.6 <font color=red>如何实现一个深拷贝</font>

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

### 8.7  <font color=red>实现双向数据绑定</font>

```js
<body>
  <input type="text" id="message" />
  <div id="msg"></div>
</body>
<script>
   var input=document.getElementById('message');
   var show=document.getElementById('msg');
   var obj={};
   Object.defineProperty(obj,'data',{
     get:function(){
       return obj;
     },
     set:function(newValue){
        input.value=newValue;
        show.innerText=newValue;
     }
   })
   input.addEventListener('keyup',function(e){
     show.innerText=e.target.value;
   })
</script>
```



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