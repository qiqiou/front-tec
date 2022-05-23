## JS基础

### 闭包

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

### this指向

1. call模拟实现

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

2. apply模拟实现

   ```js
   Function.prototype.myApply = function(context = window){
     context.fn = this;
     const res = arguments[1] ? context.fn(...arguments[1]) : context.fn();
     delete context.fn;
     return res;
   }
   ```

3. <font color=red>bind模拟实现</font>

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
   ```

4.  function与箭头函数的区别

   1. function可以显式命名，箭头函数是隐式命名
   2. 箭头函数没有prototype属性，不能使用new关键词，不能写成构造函数
   3. 箭头函数不会创建新的作用域，不会改变this指向

### 原型对象

- 实例属性和原型属性的关系

  > 参考：https://www.cnblogs.com/cirry/p/13303153.html

  ```js
  function Person() {}
  Person.prototype.name = "ccc";
  Person.prototype.age = 18;
  Person.prototype.sayName = function() {
    console.log(this.name);
  };
  
  // 等同于
  /* var Person = {
  *  prototype: {
  *    name: "ccc",
  *    age: 18,
  *    constructor() {},
  *    sayName: function() {
  *      console.log(this.name);
  *    }
  *  }
  *}
  */
  
  var person1 = new Person();
  var person2 = new Person();
  
  person1.name = 'www'      // 在person1中添加一个name属性
  
  // person1的原型
  /* Person = {
  *		name: 'www',
  * 	prototype: {
  *    	name: "ccc",
  *    	age: 18,
  *    	constructor() {},
  *    	sayName: function() {
  *     	 console.log(this.name);
  *    	}
  *  	}
  *	}
  */
  
  person1.sayName()      // --> 'www'————'来自实例'
  person2.sayName()      // --> 'ccc'————'来自原型'
  
  console.log(person1.hasOwnProperty('name'))      // --> true
  console.log(person2.hasOwnProperty('name'))      // --> false
  
  delete person1.name      // --> 删除person1中新添加的name属性
  
  // person1的原型
  /* Person = {
  * 	prototype: {
  *    	name: "ccc",
  *    	age: 18,
  *    	constructor() {},
  *    	sayName: function() {
  *     	 console.log(this.name);
  *    	}
  *  	}
  *	}
  */
  person1.sayName()      // -->'ccc'————'来自原型'
  
  ```

- prototype和\_proto\_的关系

  ```js
  function Parent() {
    this.name = "parent";
  }
  Parent.prototype.getName = function () {
    return this.name;
  };
  
  // Parent = {
  //     prototype: {
  //         constructor: function(){
  //             this.name = "parent";
  //         },
  //         getName: function () {
  //             return this.name;
  //         }
  //     },
  // }
  
  function Children(name) {
    this.cName = "children";
  }
  
  // Children = {
  //     prototype: {
  //         constructor: function(){
  //             this.cName = "children";
  //         }
  //     }
  // }
  
  // 通过原型链实现继承Children.prototype.__proto__ === Parent.prototype
  Children.prototype = new Parent();
  // Children = {
  //     prototype: {
  //         constructor: function(){
  //             this.cName = "children";
  //         },
  //         __proto__: {
  //             constructor: function(){
  //                 this.name = "parent";
  //             },
  //             getName: function () {
  //                 return this.name;
  //             }
  //         }
  //     }
  // }
  
  
  //重写方法一定要写在继承之后不然会被覆盖
  Children.prototype.getName = function () {
    return this.cName;
  };
  
  // Children = {
  //     prototype: {
  //         constructor: function(){
  //             this.cName = "children";
  //         },
  //         getName: function() {
  //             return this.cName;
  //         },
  //         __proto__: {
  //             constructor: function(){
  //                 this.name = "parent";
  //             },
  //             getName: function () {
  //                 return this.name;
  //             }
  //         }
  //     }
  // }
  
  var c = new Children();
  console.log(c.getName()); //children
  ```

  

### 作用域链

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



### 变量提升

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



### 异步

- **JS 异步解决方案的发展历程以及优缺点**

  - 回调函数callback：

    缺点：**回调地狱，不能用 try catch 捕获错误，不能 return**

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

- **setTimeout、Promise、Async/Await 的区别**

  - settimeout的回调函数放到宏任务队列里，等到执行栈清空以后执行；
  - promise.then里的回调函数会放到相应宏任务的微任务队列里，等宏任务里面的同步代码执行完再执行；
  - async函数表示函数里面可能会有异步方法，await后面跟一个表达式，async方法执行时，遇到await会立即执行表达式，然后把表达式后面的代码放到微任务队列里，让出执行栈让同步代码先执行。

### 事件循环



### 基础

1. **undefined和null的区别**

   1. typeof得到的结果不一样：
      1. typeof undefined为undefined
      2. typeof null为object
   2. 含义不一样：
      1. undefined表示已声明但没有初始化值
      2. null表示该值后面会赋值对象，初始时先赋值为null
   3. 转为数值的结果不一样：
      1. Number(undefined)等于NaN
      2. Number(null)等于0

2. **JavaScript中怎么判断相等**

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

3. **JS的数据类型有哪些**

   1. 基本数据类型（6种）：null、undefined、string、number、boolean、symbol（ ES6）
   2. 引用数据类型：object

4. **symbol的应用场景**

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

5. 自执行函数（IIFE）的作用是什么

   - 放在 IIFE 里面的变量，并不会影响到其他外层的变量，也不会被外层的变量影响。

6. 

## ES5、ES6

1. **ES6里新增的定义变量方式有哪些** 

   新增let、const、import、class声明变量的方法

2. **let const var的区别**

   1. var：函数级作用域，存在变量提升，声明前可以调用，允许重新赋值
   2. const：块级作用域，不存在变量提升，声明前不可以调用，存在暂时性死区，不允许重新赋值
   3. let：块级作用域，不存在变量提升，声明前不可以调用，存在暂时性死区，允许重新赋值

3. **const定义的变量哪种情况下能修改**

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

4. <font color=red>ES5/ES6的继承除了写法以外还有什么区别？es5/es6如何继承</font>

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

   - 

   

5. ES6 代码转成 ES5 代码的实现思路是什么

   > 参考Babel 的工作原理：
   >
   > 1. 解析(parse)：将代码字符串解析成抽象语法树——@babel/parser 将源代码解析成 AST。
   > 2. 转换(transform)：对抽象语法树进行转换操作。——@babel/generator 将 AST 解码生 JS 代码
   > 3. 生成(generate): 根据变换后的抽象语法树再生成代码字符串。——@babel/plugin-transform-? 完成代码转换

   1. 将代码字符串解析成抽象语法树，即所谓的 AST
   2. 对 AST 进行处理，在这个阶段可以对 ES6 代码进行相应转换，即转成 ES5 代码
   3. 根据处理后的 AST 再生成代码字符串

6. 什么是AST？

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