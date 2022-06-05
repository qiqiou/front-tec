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

### 1.1 谈谈对闭包的理解，闭包的用途，闭包的缺点

#### 是什么（what）

闭包是指有权访问另外一个函数作用域中的变量的函数

#### 使用场景（where）

任何闭包的使用场景都离不开这两点：

- 创建私有变量
- 延长变量的生命周期

使用闭包模拟私有方法：在`JavaScript`中，没有支持声明私有变量，但我们可以使用闭包来模拟私有方法

```js
var Counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }
})();

var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value()); /* logs 0 */
Counter1.increment();
Counter1.increment();
console.log(Counter1.value()); /* logs 2 */
Counter1.decrement();
console.log(Counter1.value()); /* logs 1 */
console.log(Counter2.value()); /* logs 0 */
```

#### 闭包的缺点

1. 闭包会使得函数中的变量都被保存在内存中，滥用闭包可能导致内存泄漏。解决方法是在函数退出之前，将不使用的局部变量全删了。
2. 闭包会在父函数外部，改变父函数内部变量的值。

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

### 2.3 bind模拟实现

```js
Function.prototype.myBind = function(context = window) {
    if (typeof this != 'function') {
        throw new TypeError('error');
    }
    const _this = this;
    let args = [...arguments].slice(1);
    return function F() {
      	// 根据调用方式，传入不同绑定值
        if (this instanceof F){
            return new _this(...args, ...arguments);
        }

        return _this.apply(context, args.concat(arguments));
    }
}

const obj = {  
  name : 'xiaoxiao',  
  getName : function (age) {  
    console.log(`我是${this.name}里面的,我有${age}岁`);  
  }  
}  

const obj2 = {  
  name : 'huahua'  
}
obj.getName.myBind(obj2, 18)();



function Point(x, y){
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return this.x + ',' + this.y;
};

var YAxisPoint = Point.myBind({}, 0);
var axisPoint = new YAxisPoint(5);
console.log(axisPoint.toString()); // '0,5'
```

### 2.4 new 模拟实现

#### new 操作符做了什么

```js
function Func() {};
var func = new Func();
```

1. 创建一个空对象

   ```js
   var obj = new Object();
   ```

2. 把对象的\__proto\__指向**构造函数**的prototype 实现继承

   ```js
   obj.__proto__ = Func.prototype;
   ```

3. 让构造函数中的`this`指向对象，并执行构造函数。

   ```js
   var result = Func.call(obj);
   ```

4. 判断构造函数的返回值类型：如果是值类型，返回`obj`。如果是引用类型，就返回这个引用类型的对象。

   ```js
   if (typeof(result) == "object"){
     func = result;
   } else {
     func = obj;;
   }
   ```

#### 实现一个new

```js
// 实现一个new
var Dog = function(name) {
  this.name = name
}
Dog.prototype.bark = function() {
  console.log('wangwang')
}
Dog.prototype.sayName = function() {
  console.log('my name is ' + this.name)
}
let sanmao = new Dog('三毛')
sanmao.sayName()
sanmao.bark()

const _new = function(fn, ...args){
  // Object.create(proto) 将对象的_proto_指向proto参数
  const obj = Object.create(fn.prototype);
  const ret = fn.apply(obj, args);
  return ret instanceof Object ? ret : obj;
}
var simao = _new(Dog, 'simao')
simao.bark()
simao.sayName()
console.log(simao instanceof Dog) // true
```

### 2.5 function与箭头函数的区别

- 普通函数this：

1. this总是代表它的直接调用者。
2. 在默认情况下，没找到直接调用者，this指的是window。
3. 在严格模式下，没有直接调用者的函数中的this是undefined。
4. 使用call,apply,bind绑定，this指的是绑定的对象。

- 箭头函数this：

1. 在使用=>定义函数的时候，this的指向是 **定义时所在的对象**，而不是使用时所在的对象；
2. **没有prototype属性，不能够用作构造函数**，这就是说，不能够使用new命令，否则就会抛出一个错误；
3. 不能够使用 **arguments** 对象；
4. 不能使用 **yield** 命令；

### 2.6 call、apply、bind 的区别

- **call、apply与bind的差别：**
  - 三者都可以改变函数的`this`对象指向
  - 三者第一个参数都是`this`要指向的对象，如果如果没有这个参数或参数为`undefined`或`null`，则默认指向全局`window`
  - 三者都可以传参，但是`apply`是数组，而`call`是参数列表，且`apply`和`call`是一次性传入参数，而`bind`可以分为多次传入
  - `bind `是返回绑定this之后的函数，`apply `、`call` 则是立即执行

### 2.7 谈谈this对象的理解

#### 是什么

`this` 关键字是函数运行时自动生成的一个内部对象，只能在函数内部使用，总指向调用它的对象

同时，`this`在函数执行过程中，`this`一旦被确定了，就不可以再更改

#### 绑定规则

根据不同的使用场合，`this`有不同的值，主要分为下面几种情况：

- 默认绑定：全局环境中定义`person`函数，内部使用`this`关键字

  ```js
  // 代码输出Jenny，原因是调用函数的对象在游览器中位window，因此this指向window，所以输出Jenny
  // 注意：严格模式下，不能将全局对象用于默认绑定，this会绑定到undefined，只有函数运行在非严格模式下，默认绑定才能绑定到全局对象
  var name = 'Jenny';
  function person() {
      return this.name;
  }
  console.log(person());  //Jenny
  ```

- 隐式绑定：函数还可以作为某个对象的方法调用，这时`this`就指这个上级对象

  ```js
  function test() {
    console.log(this.x);
  }
  
  var obj = {};
  obj.x = 1;
  obj.m = test;
  
  obj.m(); // 1
  ```

- new绑定：通过构建函数`new`关键字生成一个实例对象，此时`this`指向这个实例对象

  ```js
  function test() {
  　this.x = 1;
  }
  
  var obj = new test();
  obj.x // 1
  ```

- 显示绑定：`apply()、call()、bind()`是函数的一个方法，作用是改变函数的调用对象

#### 箭头函数

#### 优先级

new绑定优先级 > 显示绑定优先级 > 隐式绑定优先级 > 默认绑定优先级

## 3. 原型对象

### 3.1 实例属性和原型属性的关系

> 参考：https://www.cnblogs.com/cirry/p/13303153.html

1. 实例属性是通过this关键字创建的属性，是属于每一个实例对象的私有属性
2. 原型属性是通过prototype创建的属性，属于**构造函数**的原型属性，每一个实例对象都共享的属性
3. 对象实例中添加与实例原型中的同名属性时，会在实例上创建该属性，并且屏蔽原型中的那个属性
4. 可以通过hasOwnProperty()方法来检测一个属性是存在于实例中还是存在于原型中。

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
//   [[Prototype]]: {
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
//   [[Prototype]]: {
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
//   [[Prototype]]: {
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
//   [[Prototype]]: {
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

当调用**构造函数**创建一个新实例后，该实例的内部将包含一个指针[[Prototype]]，该指针指向创建它的构造函数的原型，支持通过`__proto__`来访问

```js
function Parent() {
  this.name = "parent";
}
Parent.prototype.getName = function () {
  return this.name;
};

// Parent = {
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
//   prototype: {
//     name: "parent",
//     [[Prototype]]: {
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
//   prototype: {
//     name: "parent",
//     getName: function () {
//       return this.cName;
//     },
//     [[Prototype]]: {
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
// c = {
//   cName: "children",
//   [[Prototype]]: {
//     name: "parent",
//     getName: function () {
//       return this.cName;
//     },
//     [[Prototype]]: {
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
console.log(c.getName()); //children
```

### 3.3 JS继承的实现方式

> https://www.cnblogs.com/humin/p/4556820.html

#### 原型链继承： 将父类的实例作为子类的原型

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
//     // Animal
//     constructor: function(name){
//       this.name = name || 'animal';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
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
//     name: 'cat',
//     sleep: function(){
//         console.log(this.name + ' is sleeping');
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(name){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//         console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }

var cat = new Cat();
// cat = {
//   [[prototype]]: {
//     name: 'cat',
//     sleep: function(){
//         console.log(this.name + ' is sleeping');
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(name){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//         console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }
console.log(cat.name); // cat
console.log(cat.eat('fish')); // cat eat fish
console.log(cat.sleep()); // cat is sleeping
console.log(cat instanceof Animal); //true 
console.log(cat instanceof Cat); //true 
```

#### 构造继承：使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类

- 特点：
  - 解决了子类实例共享父类引用属性的问题
  - 创建子类实例时，可以向父类传递参数
  - 可以实现多继承（call多个父类对象）
- 缺点：
  - 实例并不是父类的实例，只是子类的实例
  - **只能继承父类的实例属性和方法，不能继承原型属性/方法**
  - 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

```js
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}

// Cat = {
//   prototype: {
//     // Cat
//     constructor: function(name){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//   }
// }

// Test Code
var cat = new Cat();
// cat = {
//	 name: 'Tom',
//	 sleep: function(){
//		 console.log(this.name + ' is sleeping');
//	 },
//   [[prototype]]: {
//     // Cat
//     constructor: function(name){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//   }
// }
console.log(cat.name);
console.log(cat.sleep());
console.log(cat instanceof Animal); // false
console.log(cat instanceof Cat); // true
```

#### 实例继承：为父类实例添加新特性，作为子类实例返回

- 特点：
  - 不限制调用方式，不管是`new 子类()`还是`子类()`,返回的对象具有相同的效果
- 缺点：
  - **实例是父类的实例，不是子类的实例**
  - 不支持多继承

```js
function Cat(name){
  const instance = new Animal();
  instance.name = 'cat';
  return instance;
}

// Cat = {
//   prototype: {
//     // Cat
//     constructor: function(){}
//   }
// }

var cat = new Cat();
// 等同于 var cat = new Animal();
// cat.name = 'cat';

// cat = {
//   name: 'cat',
//   sleep: function() {
//     console.log(this.name + ' is sleeping');
//   },
//   [[prototype]]: {
//     // Animal
//     constructor: function(name){
//       this.name = name || 'animal';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//     eat: function(food){
//       console.log(this.name + ' eat ' + food);
//     }
//   }
// }
console.log(cat.name); // cat
console.log(cat.sleep()); // cat is sleeping
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); // false
```

#### 拷贝继承

- 特点：
  1. 支持多继承
- 缺点：
  1. 效率较低，内存占用高（因为要拷贝父类的属性）
  2. 无法获取父类不可枚举的方法（不可枚举方法，不能使用for in 访问到，enumerable=false）
  3. 实例并不是父类的实例，只是子类的实例

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

#### 组合继承：构造函数继承+原型链继承

- 特点：
  1. 可以继承实例属性/方法，也可以继承原型属性/方法
  2. 既是子类的实例，也是父类的实例
  3. 不存在引用属性共享问题
  4. 可传参
  5. 函数可复用
- 缺点：
  1. **调用了两次父类构造函数**，生成了两份实例（子类实例将子类原型上的那份屏蔽了）

```js
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}

// Cat = {
//   prototype: {
//     // Cat
//     constructor: function(){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     }
//   }
// }
Cat.prototype = new Animal();

// Cat = {
//   prototype: {
//     name: 'animal',
//     sleep: function() {
//       console.log(this.name + ' is sleeping');
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//          console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }

Cat.prototype.constructor = Cat;
// Cat = {
//   prototype: {
//     name: 'animal',
//     sleep: function() {
//       console.log(this.name + ' is sleeping');
//     },
//     // Cat
//     constructor: function(){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//          console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }
// Test Code
var cat = new Cat();
// cat = {
//   name: 'Tom',
//   sleep: function(){
//     console.log(this.name + ' is sleeping');
//   },
//   [[prototype]]: {
//     // Cat
//     constructor: function() {
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//     name: 'animal',
//     sleep: function(){
//       console.log(this.name + ' is sleeping');
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//          console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }
console.log(cat.name); // Tom
console.log(cat.sleep()); // Tom is sleeping
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); // true
```

#### 寄生组合继承：通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

```js
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}

// Cat = {
//   prototype: {
//     // Cat
//     constructor: function(name){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//   }
// }

(function(){
  // 创建一个没有实例方法的类
  var Super = function(){};
  Super.prototype = Animal.prototype;


  // Super = {
  //   prototype: {
  //     // Animal
  //     constructor: function(name){
  //       this.name = name || 'animal';
  //       this.sleep = function(){
  //         console.log(this.name + ' is sleeping');
  //       }
  //     },
  //     eat: function(food){
  //       console.log(this.name + ' eat ' + food);
  //     }
  //   }
  // }

  //将实例作为子类的原型
  Cat.prototype = new Super();

  // Cat = {
  //   prototype: {
  //     [[prototype]]: {
  //       // Animal
  //       constructor: function(name){
  //         this.name = name || 'animal';
  //         this.sleep = function(){
  //           console.log(this.name + ' is sleeping');
  //         }
  //       },
  //       eat: function(food){
  //         console.log(this.name + ' eat ' + food);
  //       }
  //     }
  //   }
  // }
})();

Cat.prototype.constructor = Cat; // 需要修复下构造函数

// Cat = {
//   prototype: {
//     // Cat
//     constructor: function(name){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(name){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//         console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }

// Test Code
var cat = new Cat();
// cat = {
//   name: 'Tom',
//   sleep: function(){
//     console.log(this.name + ' is sleeping');
//   },
//   [[prototype]]: {
//     // Cat
//     constructor: function(name){
//       this.name = name || 'Tom';
//       this.sleep = function(){
//         console.log(this.name + ' is sleeping');
//       }
//     },
//     name: 'animal',
//     sleep: function(){
//       console.log(this.name + ' is sleeping');
//     },
//     [[prototype]]: {
//       // Animal
//       constructor: function(name){
//         this.name = name || 'animal';
//         this.sleep = function(){
//           console.log(this.name + ' is sleeping');
//         }
//       },
//       eat: function(food){
//         console.log(this.name + ' eat ' + food);
//       }
//     }
//   }
// }
console.log(cat.name); // Tom
console.log(cat.sleep()); // Tom is sleeping
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); //true
```

### 3.4 实现一个instanceof

```js
const _instance_  = function(left, right){
  let proto = left._proto_;
  let prototype = right.prototype;
  while(1){
    if(proto === null){
      return false;
    }
    if(proto === prototype){
      return true;
    }
    proto = proto._proto_;
  }
}
```

### 3.5 JavaScript原型，原型链 ? 有什么特点？

#### 原型

`JavaScript` 常被描述为一种基于原型的语言——每个对象拥有一个原型对象

当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾

准确地说，这些属性和方法定义在Object的构造器函数（constructor functions）之上的`prototype`属性上，而非实例对象本身

下面举个例子：

函数可以有属性。 每个函数都有一个特殊的属性叫作原型`prototype`

```
function doSomething(){}
console.log( doSomething.prototype );
```

控制台输出

```
{
    constructor: ƒ doSomething(),
    __proto__: {
        constructor: ƒ Object(),
        hasOwnProperty: ƒ hasOwnProperty(),
        isPrototypeOf: ƒ isPrototypeOf(),
        propertyIsEnumerable: ƒ propertyIsEnumerable(),
        toLocaleString: ƒ toLocaleString(),
        toString: ƒ toString(),
        valueOf: ƒ valueOf()
    }
}
```

上面这个对象，就是大家常说的原型对象

#### 原型链

原型对象也可能拥有原型，并从中继承方法和属性，一层一层、以此类推。这种关系常被称为原型链 (prototype chain)，它解释了为何一个对象会拥有定义在其他对象中的属性和方法

在对象实例和它的构造器之间建立一个链接（它是`__proto__`属性，是从构造函数的`prototype`属性派生的），之后通过上溯原型链，在构造器中找到这些属性和方法

下面举个例子：

```
function Person(name) {
    this.name = name;
    this.age = 18;
    this.sayName = function() {
        console.log(this.name);
    }
}
// 第二步 创建实例
var person = new Person('person')
```

根据代码，我们可以分析出：

- 构造函数`Person`存在原型对象`Person.prototype`
- 构造函数生成实例对象`person`，`person`的`__proto__`指向构造函数`Person`原型对象
- `Person.prototype.__proto__` 指向内置对象，因为 `Person.prototype` 是个对象，默认是由 `Object `函数作为类创建的，而 `Object.prototype` 为内置对象
- `Person.__proto__` 指向内置匿名函数 `anonymous`，因为 Person 是个函数对象，默认由 Function 作为类创建
- `Function.prototype` 和 `Function.__proto__ `同时指向内置匿名函数 `anonymous`，这样原型链的终点就是 `null`

## 4. 作用域链和执行上下文

### 4.1 说说你对作用域链的理解

#### 是什么

作用域，即变量（变量作用域又称上下文）和函数生效（能被访问）的区域或集合

换句话说，作用域决定了代码区块中变量和其他资源的可见性

我们一般将作用域分成：

- 全局作用域：全局作用域下声明的变量可以在程序的任意位置访问
- 函数作用域：如果一个变量是在函数内部声明的它就在一个函数作用域下面。这些变量只能在函数内部访问，不能在函数以外去访问
- 块级作用域：ES6引入了`let`和`const`关键字,和`var`关键字不同，在大括号中使用`let`和`const`声明的变量存在于块级作用域中。在大括号之外不能访问这些变量

#### 词法作用域

词法作用域，又叫静态作用域，变量被创建时就确定好了，而非执行阶段确定的。也就是说我们写好代码时它的作用域就确定了，`JavaScript` 遵循的就是词法作用域

#### 作用域链

当在`Javascript`中使用一个变量的时候，首先`Javascript`引擎会尝试在当前作用域下去寻找该变量，如果没找到，再到它的上层作用域寻找，以此类推直到找到该变量或是已经到了全局作用域

如果在全局作用域里仍然找不到该变量，它就会在全局范围内隐式声明该变量(非严格模式下)或是直接报错

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

#### 是什么

只要有`Javascript`代码运行，那么它就一定是运行在执行上下文中

执行上下文的类型分为三种：

- 全局执行上下文：只有一个，浏览器中的全局对象就是 `window `对象，`this` 指向这个全局对象
- 函数执行上下文：存在无数个，只有在函数被调用的时候才会被创建，每次调用函数都会创建一个新的执行上下文
- Eval 函数执行上下文： 指的是运行在 `eval` 函数中的代码，很少用而且不建议使用

#### 生命周期

执行上下文的生命周期包括三个阶段：创建阶段 → 执行阶段 → 回收阶段

**创建阶段**：

创建阶段即当函数被调用，但未执行任何其内部代码之前，创建阶段做了三件事：

- 确定 this 的值，`this`的值是在执行的时候才能确认，定义的时候不能确认

- **（词法环境） 组件被创建**，词法环境有两个组成部分：

  - 全局环境：是一个没有外部环境的词法环境，其外部环境引用为` null`，有一个全局对象，`this` 的值指向这个全局对象
  - 函数环境：用户在函数中定义的变量被存储在环境记录中，包含了`arguments` 对象，外部环境的引用可以是全局环境，也可以是包含内部函数的外部函数环境

- **（变量环境） 组件**被创建：

  变量环境也是一个词法环境，因此它具有上面定义的词法环境的所有属性

  在 ES6 中，词法环境和变量环境的区别在于前者用于存储函数声明和变量（ `let` 和 `const` ）绑定，而后者仅用于存储变量（ `var` ）绑定

`let`和`const`定义的变量在创建阶段没有被赋值，但`var`声明的变量从在创建阶段被赋值为`undefined`

这是因为，创建阶段，会在代码中扫描变量和函数声明，然后将函数声明存储在环境中

但变量会被初始化为`undefined`(`var`声明的情况下)和保持`uninitialized`(未初始化状态)(使用`let`和`const`声明的情况下)

这就是变量提升的实际原因

**执行阶段**

在这阶段，执行变量赋值、代码执行

如果 `Javascript` 引擎在源代码中声明的实际位置找不到变量的值，那么将为其分配 `undefined` 值

**回收阶段**

执行上下文出栈等待虚拟机回收执行上下文

#### 执行栈

执行栈，也叫调用栈，具有 LIFO（后进先出）结构，用于存储在代码执行期间创建的所有执行上下文

当`Javascript`引擎开始执行你第一行脚本代码的时候，它就会创建一个全局执行上下文然后将它压到执行栈中

每当引擎碰到一个函数的时候，它就会创建一个函数执行上下文，然后将这个执行上下文压到执行栈中

引擎会执行位于执行栈栈顶的执行上下文(一般是函数执行上下文)，当该函数执行结束后，对应的执行上下文就会被弹出，然后控制流程到达执行栈的下一个执行上下文

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

- 引入`JavaScript`的方式

  - 内联形式：这种方式指的是在 `html `文件中，添加一个`<script></scritp>`标签
  - 外置形式：在 `html` 文件中通过 `<script>` 标签的 `src` 属性引入

- `<script>`标签加载顺序

  - `<script>`标签放在`<head>`元素中：必须等到全部 JavaScript 代码都被下载、解析和执行完成以后，才能开始呈现页面的内容
  - `<script>`标签放在`<body>`元素中页面内容后面：在解析包含的 JavaScript 代码之前，页面的内容将完全呈现在浏览器中

- `script`标签不阻塞页面的方法：

  >   `<script>` 标签没有 `src` 特性，那么 `async` 和`defer`特性会被忽略。

  - `defer`特性：

    - 具有 `defer` 特性的脚本总是要等到 DOM 解析完毕，但在 `DOMContentLoaded` 事件之前执行。
    - **具有 `defer` 特性的多个js脚本并行下载（下载没有顺序），文档顺序执行**
    - `defer` 用于需要整个 DOM 的脚本，和/或脚本的相对执行顺序很重要的时候。

  - `async`特性：

    - **`async` 异步脚本以“加载优先”的顺序执行。**
    - `DOMContentLoaded` 可能在 `async` 之前或之后触发，不能保证谁先谁后。
    - `async` 用于独立脚本，例如计数器或广告，这些脚本的相对执行顺序无关紧要。

  - 动态添加脚本：

    - **默认情况下，动态脚本的行为是“异步”的**

    - 先加载完成的脚本先执行（“加载优先”顺序）

      ```js
      let script = document.createElement('script');
      script.src = "/article/script-async-defer/long.js";
      document.body.append(script); // (*)
      ```

- `<script>`代码块：

  - 代块之间相互独立，但是**「变量」**和**「方法」**共享，Js只有全局作用域和函数作用域，没有块级作用域
  - 前面的JavaScript代码运行错误并不影响后面`<script>`标签里的代码运行

  ```js
  // 代码块一报错并不影响代码块二运行，这就是代码块间的独立性。
  // 代码块二能够调用到代码块一的变量，有人把这个叫做块间共享性。
  
  // 浏览器解析阶段：代码执行之前使用var定义的变量会「声明提前」（声明式函数也会声明提前），先定义后执行时赋值，所以在运行到console.log(s)之前变量test1已经被声明为undefined。
  
  // 浏览器执行阶段：执行到console.log(s)因为s未定义报错，后面的代码都不执行了。此时的test1就没有完成将'我是test1'赋值给test1变量的操作，所以代码块二拿到undefined。
  
  <!DOCTYPE html>
    <html lang="en">
      <head>
  		<script>    
        console.log(s); //s未定义，控制台报错，后面的都不执行了    
  			console.log('test1');     
  			var test1 = '我是test1';  
  		</script>  
  		<script>    
        console.log('test2'); //控制台输出test2    
  			console.log(test1); //输出undefined。并不是报错未定义  
  		</script>
  		</head>
  		<body>  
      </body>
  </html>
  ```

- 声明提前：JavaScript的函数定义分为两种：声明式函数和赋值式函数

  ```js
  //声明式function fn() {    }
  
  //赋值式函数var fn = function (){    }
  ```

  在浏览器解析阶段，声明式函数会和之前的变量一样先被声明，未赋值。

  ```js
  // 因为function这种形式定义函数会在浏览器预解析的时候声明提前，所以fn函数调用在函数定义之前也可以。
  <!DOCTYPE html>
    <html lang="en">
    <head>  
      <script>    
      fn(); //'2'    
  		function fn() {      
        console.log('1');    
      }    
  		function fn() {      
        console.log('2');    
      }  
  		</script>  
  		<script>    
        fn(); //声明    
  			var fn = function () {      
          console.log('赋值');    
        }    
        function fn() {      
          console.log('声明');    
        }    
  			fn(); //赋值  
  		</script>
  	</head>
  	<body>
    </body>
  </html>
  ```

  ```js
  // JS引擎是按照代码块顺序执行的，其实完整的说应该是按照代码块来进行预处理和执行的。
  // 也就是说预处理的只是「执行到的代码块的声明函数和变量」，而对于还未加载的代码块，是没法进行预处理的
  <!DOCTYPE html>
    <html lang="en">
      <head>
      <script>    
      fn(); //报错，未定义的fn函数  
  		</script>  
  		<script>    
        function fn() {      
        	console.log('fn');    
      	}  
  		</script>
  		</head>
  		<body>
      </body>
  </html>
  ```



## 6. 异步

### 6.0  **JavaScript运行机制是什么？**

1. JavaScript语言是单线程的，同一个时间只能做一件事；

2. 遵循事件循环机制，当JS解析执行时，会被引擎分为两类任务，同步任务 和 异步任务。

   1. 对于同步任务来说，会被推到执行栈按顺序去执行这些任务。
   2. 对于异步任务来说，当其可以被执行时，会被放到一个 任务队列里等待JS引擎去执行。
   3. **当执行栈中的所有同步任务完成后，JS引擎才会去任务队列里查看是否有任务存在**，并将任务放到执行栈中去执行，执行完了又会去任务队列里查看是否有已经可以执行的任务。

   这种循环检查的机制，就叫做事件循环。对于任务队列，其实是有更细的分类。其被分为 微任务队列 & 宏任务队列。

3. 一般来说，有以下四种会放入异步任务队列：

   1. setTimeout和setlnterval
   2. DOM事件
   3. ES6中的Promise
   4. Ajax异步请求

```js
// for循环一次碰到一个 setTimeout()，并不是马上把setTimeout()拿到异步队列中，而要等到一秒后，才将其放到任务队列里面，
// 一旦"执行栈"中的所有同步任务执行完毕（即for循环结束，此时i已经为5），系统就会读取已经存放"任务队列"的setTimeout()（有五个），于是答案是输出5个5。
for(var i = 0; i < 5; i++) {
  setTimeout(function() => {
    console.log(i);   
  }, 1000);
}
```



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
  - 如果不设置回调函数，Promise内部抛出的错误，不会反应到外部，因此外部脚本并不会因为错误而导致不继续执行下去。
  - 一旦新建了，就无法中断它的操作。不像`setTimeout`那样，我还可以使用`clearTimeout`取消掉。

### 6.3 **async/await 如何通过同步的方式实现异步**

`async/await`是参照`Generator`封装的一套异步处理方案，可以理解为`Generator`的语法糖。

而`Generator`又依赖于迭代器`Iterator`，而`Iterator`的思想呢又来源于单向链表，

`Iterator`的遍历过程(类似于单向链表)：

- 创建一个**指针对象**，指向当前数据结构的起始位置
- 第一次调用指针对象的`next`方法，将指针指向数据结构的第一个成员
- 第二次调用指针对象的`next`方法，将指针指向数据结构的第二个成员
- 不断的调用指针对象的`next`方法，直到它指向数据结构的结束位置

其中的`next()`方法，须返回一个对象,对象有两个必要的属性：

- `done`（bool）
  - true：迭代器已经超过了可迭代次数。这种情况下,value 的值可以被省略
  - 如果迭代器可以产生序列中的下一个值，则为 false。这等效于没有指定 done 这个属性
- `value`迭代器返回的任何 JavaScript 值。done 为 true 时可省略

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

可以看到，`async function`代替了`function*`，`await`代替了`yield`

### 6.4 **setTimeout、Promise、Async/Await 的区别**

- settimeout：

  - settimeout的回调函数放到宏任务队列里，等到执行栈清空以后执行；

- promise 

  - new的时候会立即执行里面的代码；
  - promise.then里的回调函数会放到相应宏任务的微任务队列里，等宏任务里面的同步代码执行完再执行；

- async：

  - async方法执行时，遇到await会立即执行表达式，**然后把表达式后面的代码放到微任务队列里，让出执行栈让同步代码先执行**。

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

内存方面，setTimeout只需要进入一次队列，不会造成内存溢出，

setInterval因为不计算代码执行时间，有可能同时执行多次代码，导致内存溢出。

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
          request.open(method, url+'?'+ data);
          request.send(data);
        }else{
          request.open(method, url);
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

### 6.6 ajax原理是什么？如何实现

#### 是什么

即异步的` JavaScript` 和` XML`，是一种创建交互式网页应用的网页开发技术，可以在不重新加载整个网页的情况下，与服务器交换数据，并且更新部分网页

`Ajax`的原理简单来说通过`XmlHttpRequest`对象来向服务器发异步请求，从服务器获得数据，然后用`JavaScript`来操作`DOM`而更新页面

#### 实现过程

实现 `Ajax `异步交互需要服务器逻辑进行配合，需要完成以下步骤：

- 创建 `Ajax `的核心对象 `XMLHttpRequest `对象
- 通过 `XMLHttpRequest` 对象的 `open()` 方法与服务端建立连接
- 构建请求所需的数据内容，并通过` XMLHttpRequest` 对象的 `send()` 方法发送给服务器端
- 通过 `XMLHttpRequest` 对象提供的 `onreadystatechange` 事件监听服务器端你的通信状态
- 接受并处理服务端向客户端响应的数据结果
- 将处理结果更新到 `HTML `页面中

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

- 错误处理

  - Promise.all()方法生成的Promise对象也会有一个catch方法来捕获错误处理
  - 如果数组中的Promise对象变成rejected状态时，并且这个对象还定义了catch的方法，那么rejected的对象会执行自己的catch方法，并且返回一个状态为fullfilled的Promise对象，Promise.all()生成的对象会接受这个Promise对象，不会返回rejected状态。

- **原理实现（实现一个 Promise.all）**：

  ```js
  function promiseAll(promiseArray){
      return new Promise((resolve, reject) => {
          if(!Array.isArray(promiseArray)){
              return reject(new TypeError());
          }
          let count = 0;
          let resolveRes = [];
  
          for(let i = 0;i < promiseArray.length; i++){
              Promise.resolve(promiseArray[i]).then(res => {
                  count++;
                  resolveRes[i] = res;
                  if(count === promiseArray.length){
                      return resolve(resolveRes);
                  }
              }, err => {
                  return reject(err);
              })
          }
      });
  }
  
  const p1 = Promise.resolve(1);
  const p2 = Promise.resolve(2);
  const p3 = Promise.resolve(3);
  promiseAll([p1, p2, p3]).then(res => {
      console.log(res);
  })
  ```

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
function myPromiseRace(promiseArr) {
  	if(!Array.isArray(promiseArray)){
      	return Promise.reject(new TypeError());
    }

    return new Promise((resolve, reject) => {
        for(let promise of promiseArr) {
            promise.then(res=>{
                resolve(res);
            }).catch(err=>{
                reject(err);
            })
        }
    })
}
```

### 6.12  手写实现Promise

> https://juejin.cn/post/6844903625769091079

### 6.13 ajax的用途，ajax请求的五种状态

- AJAX是“Asynchronous JavaScript And XML”的缩写，是一种实现**无页面刷新**获取服务器数据的混合技术。

- **XMLHttpRequest** 对象是浏览器提供的一个API，用来顺畅地向服务器发送请求并解析服务器响应，当然整个过程中，浏览器页面不会被刷新。

- .open()方法接收三个参数：请求方式（get or post），请求URL地址和是否为异步请求的布尔值。“同步”意味着一旦请求发出，任何后续的JavaScript代码不会再执行，“异步”则是当请求发出后，后续的JavaScript代码会继续执行，当请求成功后，会调用相应的回调函数。

  ```js
  // 该段代码会启动一个针对“example.php”的GET同步请求。
  xhr.open("get", "example.php", false)
  ```

- xhr实例的readystatechange事件会监听xhr.readyState属性的变化，有以下五种变化：

  | readyState    | 对应常量             | 描述                                                         |
  | ------------- | -------------------- | ------------------------------------------------------------ |
  | 0(未初始化)   | xhr.UNSENT           | 请求已建立, 但未初始化(此时未调用open方法)                   |
  | 1(初始化)     | xhr.OPENED           | 请求已建立, 但未发送 (已调用open方法, 但未调用send方法)      |
  | 2(发送数据)   | xhr.HEADERS_RECEIVED | 请求已发送 (send方法已调用, 已收到响应头)                    |
  | 3(数据发送中) | xhr.LOADING          | 请求处理中, 因响应内容不全, 这时通过responseBody和responseText获取可能会出现错误 |
  | 4(完成)       | xhr.DONE             | 数据接收完毕, 此时可以通过responseBody和responseText获取完整的响应数据 |

### 6.14 setTimeout(fn,0)的作用

- setTimeout(fn,0)的含义是，指定某个任务在主线程最早可得的空闲时间执行，也就是说，尽可能早得执行。
- 它在"任务队列"的尾部添加一个事件，因此要等到主线程把同步任务和"任务队列"现有的事件都处理完，才会得到执行。
- 用处就在于我们可以 **改变任务的执行顺序**,因为浏览器会在执行完当前任务队列中的任务，再执行setTimeout队列中积累的的任务。

### 6.15 async 及 await 的特点，它们的优点和缺点分别是什么？await 原理是什么

- 一个函数如果加上 async ，那么该函数就会返回一个 Promise。async 就是将函数返回值使用 Promise.resolve() 包裹了下，和 then 中处理返回值一样，并且 await 只能配套 async 使用。
- async 和 await 可以说是异步终极解决方案了，相比直接使用 Promise 来说，优势在于处理 then 的调用链，能够更清晰准确的写出代码，毕竟写一大堆 then 也很恶心，并且也能优雅地解决回调地狱问题。当然也存在一些缺点，因为 await 将异步代码改造成了同步代码，如果多个异步代码没有依赖性却使用了 await 会导致性能上的降低。

### 6.16 setTimeout、setInterval、requestAnimationFrame 各有什么特点？

- setTimeout(code, millseconds) 用于延时执行参数指定的代码，如果在指定的延迟时间之前，你想取消这个执行，那么直接用clearTimeout(timeoutId)来清除任务，timeoutID 是 setTimeout 时返回的；
- setInterval(code, millseconds)用于每隔一段时间执行指定的代码，永无停歇，除非你反悔了，想清除它，可以使用 clearInterval(intervalId)，这样从调用 clearInterval 开始，就不会在有重复执行的任务，intervalId 是 setInterval 时返回的；
- requestAnimationFrame(code)，一般用于动画，与 setTimeout 方法类似，区别是 setTimeout 是用户指定的，而 requestAnimationFrame 是浏览器刷新频率决定的，一般遵循 W3C 标准，它在浏览器每次刷新页面之前执行。
- setTimeout和setInterval的问题是，它们都不精确。它们的内在运行机制决定了时间间隔参数实际上只是指定了把动画代码添加到浏览器UI线程队列中以等待执行的时间。如果队列前面已经加入了其他任务，那动画代码就要等前面的任务完成后再执行。requestAnimationFrame采用系统时间间隔，保持最佳绘制效率，不会因为间隔时间过短，造成过度绘制，增加开销；也不会因为间隔时间太长，使用动画卡顿不流畅，让各种网页动画效果能够有一个统一的刷新机制，从而节省系统资源，提高系统性能，改善视觉效果。著作权归作者所有。

### 6.17 async函数对 Generator 函数的改进

1. 内置执行器 Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。
2. 更好的语义 async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。
3. 更广的适用性 co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时会自动转成立即 resolved 的 Promise 对象）。
4. 返回值是Promise async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。

### 6.18 设计并实现promise.finally

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

> https://github.com/febobo/web-interview/issues/73
> https://segmentfault.com/a/1190000038985579

#### 是什么

`JavaScript` 在设计之初便是单线程，即指程序运行时，只有一个线程存在，同一时间只能做一件事

为什么要这么设计，跟`JavaScript`的应用场景有关

`JavaScript` 初期作为一门浏览器脚本语言，通常用于操作 `DOM` ，如果是多线程，一个线程进行了删除 `DOM` ，另一个添加 `DOM`，此时浏览器该如何处理？

为了解决单线程运行阻塞问题，`JavaScript`用到了计算机系统的一种运行机制，这种机制就叫做事件循环（Event Loop）

#### 事件循环

在`JavaScript`中，所有的任务都可以分为

- 同步任务：立即执行的任务，同步任务一般会直接进入到主线程中执行
- 异步任务：异步执行的任务，比如`ajax`网络请求，`setTimeout `定时函数等

同步任务进入主线程，即主执行栈，异步任务进入任务队列，主线程内的任务执行完毕为空，会去任务队列读取对应的任务，推入主线程执行。上述过程的不断重复就是事件循环

#### 宏任务与微任务

#### 微任务

常见的微任务有：

- Promise.then
- MutaionObserver
- Object.observe（已废弃；Proxy 对象替代）
- process.nextTick（Node.js）

#### 宏任务

常见的宏任务有：

- script (可以理解为外层同步代码)
- setTimeout/setInterval
- UI rendering/UI事件
- postMessage、MessageChannel
- setImmediate、I/O（Node.js）

它的执行机制是：

- 执行一个宏任务，如果遇到微任务就将它放到微任务的事件队列中
- 当前宏任务执行完成后，会查看微任务的事件队列，然后将里面的所有微任务依次执行完

#### async与await

`async` 是异步的意思，`await `则可以理解为等待

放到一起可以理解`async`就是用来声明一个异步方法，而 `await `是用来等待异步方法执行

##### async

`async`函数返回一个`promise`对象，

##### await

正常情况下，`await`命令后面是一个 `Promise `对象，返回该对象的结果。如果不是 `Promise `对象，就直接返回对应的值

不管`await`后面跟着的是什么，`await`都会阻塞后面的代码，先执行 `async `外面的同步代码，同步代码执行完，再回到 `async` 函数中，再执行之前阻塞的代码

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

4. 用法不一样：

   1. Null典型用法是：

      （1）作为函数的参数，表示该函数的参数不是对象。

      （2）作为对象原型链的终点。

   2. Undefined典型用法是： 

      （1）变量被声明了，但没有赋值时，就等于undefined。 

      （2) 调用函数时，应该提供的参数没有提供，该参数等于undefined。 

      （3）对象没有赋值的属性，该属性的值为undefined。 

      （4）函数没有返回值时，默认返回undefined。


### 8.2 **JavaScript中怎么判断相等**

1. 相等运算符==：先转换类型再比较，在比较`null`的情况的时候，我们一般使用相等操作符`==`

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

变量的值在编译阶段是无法获取的，只有等到程序运行时才能知道，如果运算的类型与预期不符合，会触发类型转换机制，常见的类型转换有：强制转换（显示转换）、自动转换（隐式转换）

1. 基本数据类型（6种）：null、undefined、string、number、boolean、symbol（ ES6）
2. 引用数据类型：object

### 8.3 显示转换和隐式转换

#### 显示转换

显示转换，即我们很清楚可以看到这里发生了类型的转变，常见的方法有：

- Number()：Number`转换的时候是很严格的，只要有一个字符无法转成数值，整个字符串就会被转为`NaN

  ```js
  Number(324) // 324
  
  // 字符串：如果可以被解析为数值，则转换为相应的数值
  Number('324') // 324
  
  // 字符串：如果不可以被解析为数值，返回 NaN
  Number('324abc') // NaN
  
  // 空字符串转为0
  Number('') // 0
  
  // 布尔值：true 转成 1，false 转成 0
  Number(true) // 1
  Number(false) // 0
  
  // undefined：转成 NaN
  Number(undefined) // NaN
  
  // null：转成0
  Number(null) // 0
  
  // 对象：通常转换成NaN(除了只包含单个数值的数组)
  Number({a: 1}) // NaN
  Number([1, 2, 3]) // NaN
  Number([5]) // 5
  ```

- parseInt()

  `parseInt`相比`Number`，就没那么严格了，`parseInt`函数逐个解析字符，遇到不能转换的字符就停下来

  ```js
  parseInt('32a3') //32
  ```

- String()

  ```js
  // 数值：转为相应的字符串
  String(1) // "1"
  
  //字符串：转换后还是原来的值
  String("a") // "a"
  
  //布尔值：true转为字符串"true"，false转为字符串"false"
  String(true) // "true"
  
  //undefined：转为字符串"undefined"
  String(undefined) // "undefined"
  
  //null：转为字符串"null"
  String(null) // "null"
  
  //对象
  String({a: 1}) // "[object Object]"
  String([1, 2, 3]) // "1,2,3"
  ```

- Boolean()

  ```js
  Boolean(undefined) // false
  Boolean(null) // false
  Boolean(0) // false
  Boolean(NaN) // false
  Boolean('') // false
  Boolean({}) // true
  Boolean([]) // true
  Boolean(new Boolean(false)) // true
  ```

#### 隐式转换

在隐式转换中，我们可能最大的疑惑是 ：何时发生隐式转换？

我们这里可以归纳为两种情况发生隐式转换的场景：

- 比较运算（`==`、`!=`、`>`、`<`）、`if`、`while`需要布尔值地方
- 算术运算（`+`、`-`、`*`、`/`、`%`）

除了上面的场景，还要求运算符两边的操作数不是同一类型

##### 自动转换为布尔值

在需要布尔值的地方，就会将非布尔值的参数自动转为布尔值，系统内部会调用`Boolean`函数

可以得出个小结：

- undefined
- null
- false
- +0
- -0
- NaN
- ""

除了上面几种会被转化成`false`，其他都换被转化成`true`

##### 自动转换成字符串

遇到预期为字符串的地方，就会将非字符串的值自动转为字符串

具体规则是：先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串

常发生在`+`运算中，一旦存在字符串，则会进行字符串拼接操作

```js
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

##### 自动转换成数值

除了`+`有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值

```js
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1
undefined + 1 // NaN
// null`转为数值时，值为`0` 。`undefined`转为数值时，值为`NaN
```

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

放在 IIFE 里面的变量，并不会影响到其他外层的变量，也不会被外层的变量影响。

### 8.7  实现双向数据绑定

```js
<body>
  <input type="text" id="message" />
  <div id="msg"></div>
</body>
<script>
   var input=document.getElementById('message');
   var show=document.getElementById('msg');
   var obj={};
   Object.defineProperty(obj,'inputValue',{
     get:function(){
       return input.value;
     },
     set:function(newValue){
        input.value=newValue;
        show.innerText=newValue;
     }
   })
   input.addEventListener('keyup',function(e){
     show.innerText=obj.inputValue;
   })
</script>
```

### 8.8 写一个jsonp的实现

**Jsonp是实现跨域请求的一种方式，原理是利用`<script>`标签的src属性不受同源策略限制的特点，来发送跨域请求。**

```js
var url = "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler";
var script = document.createElement('script');
script.setAttribute('src', url);
document.getElementsByTagName('head')[0].appendChild(script);
```

### 8.9 函数去抖

**函数防抖**：当持续触发事件时，一定时间段内没有再触发事件，事件处理函数才会执行一次，如果设定的时间到来之前，又一次触发了事件，就重新开始延时

**举个栗子**：我连续输入100个字母，在输入最后一个字母后，再等200毫秒，执行请求接口。

**原理**：维护一个计时器，规定在delay时间后触发函数，但是在delay时间内再次触发的话，就会取消之前的计时器而重新设置。

**应用场景**：

- 手机号、邮箱输入检测
- 输入框搜索输入（只需最后后一次输入完成后，在发起ajax请求）
- 窗口大小resize  （只需窗口调整完成后，计算窗口大小，防止重复渲染）
- 滚动事件scroll（只需执行触发的最后一次滚动事件的处理程序）
- 文本输入的验证（连续输入文字后发送 AJAX 请求进行验证，（停止输入后）验证一次就好）

<img src="../assert/debounce and throttle.png" style="zoom:80%;" />



**代码实现**：

```js
function debounce(fn,delay=200){
    let timer = null;
    return function(){
        if(timer) clearTimeout(timer);
        timer = setTimeout(()=>{
            fn.apply(this,arguments);
            timer = null;
        },delay);
    }
}
```



### 8.10 函数节流

**函数节流**：当持续触发事件时，保证一定时间段内只调用一次事件处理函数

**举个例子**：我连续输入100个字母，在输入过程中，每隔300毫秒就请求一次接口

**原理**：通过判断是否到达一定时间来触发函数

**应用场景**：

- DOM 元素的拖拽功能实现（mousemove）
- 射击游戏的 mousedown/keydown 事件（单位时间只能发射一颗子弹）
- 计算鼠标移动的距离（mousemove）
- Canvas 模拟画板功能（mousemove）
- 搜索联想（keyup）
- 滚动事件 scroll，(只要页面滚动就会间隔一段时间去判断一次)

**代码实现**：

```js
function throttle(fn,delay=100){
    //首先设定一个变量，在没有执行我们的定时器时为null
    let timer = null;
    return function(){
        //当我们发现这个定时器存在时，则表示定时器已经在运行中，需要返回
        if(timer) return;
        timer = setTimeout(()=>{
            fn.apply(this,arguments);
            timer = null;
        },delay);
    }
}
```

### 8.11 模拟Object.create

Object.create只是对对象的一个浅复制，在修改引用类型的属性时，所有属性都会被修改。

这是由于Object.create的实现机制，只是利用new，相当于是在中间加了一个中间层，多了一个`__proto__`指向原对象。

```js
const create = (o) => {
    function F() {};
    F.prototype = o;

    return new F();
}

const person = {
    name: 'willem',
    colors: ['red', 'green', 'blue']
};

const p1 = create(person);
const p2 = create(person);

p1.colors.push('white');
console.log(person, p1, p2);
```

### ※8.12 深拷贝浅拷贝的区别？如何实现一个深拷贝

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

```js
let arr = [1, 2, [3, 4, 5, [6, 7], 8], 9, 10, [11, [12, 13]]]
// 方法一
const flat = (arr) => {
    let arrs = [];
    arr.map((item) => {
        if (Array.isArray(item)) {
            arrs.push(...flat(item));
        } else {
            arrs.push(item);
        }
    });
    return arrs;
}
// 方法二
arr.toString().split(',').map(item => {return +item})
```

### 8.14 实现 (5).add(3).minus(2) 功能

```js
Number.prototype.add = function(n) {
  return this.valueOf() + n;
};
Number.prototype.minus = function(n) {
  return this.valueOf() - n;
};
```

### 8.16 有哪些方式可以判断是否是数组，请分别介绍它们之间的区别和优劣

1. Object.prototype.toString.call()：默认返回当前对象的类型

   每一个继承 Object 的对象都有 `toString` 方法，如果 `toString` 方法没有重写的话，会返回 `[Object type]`，其中 type 为对象的类型。但当除了 Object 类型的对象外，其他类型直接使用 `toString` 方法时，会直接返回都是内容的字符串，所以我们需要使用call或者apply方法来改变toString方法的执行上下文。

   ```js
   const an = ['Hello','An']; 
   an.toString(); // "Hello,An" 
   Object.prototype.toString.call(an); // "[object Array]"
   ```

   这种方法对于所有基本的数据类型都能进行判断，即使是 null 和 undefined 。

   ```js
   Object.prototype.toString.call('An') // "[object String]" 
   Object.prototype.toString.call(1) // "[object Number]" 
   Object.prototype.toString.call(Symbol(1)) // "[object Symbol]" 
   Object.prototype.toString.call(null) // "[object Null]" 
   Object.prototype.toString.call(undefined) // "[object Undefined]" 
   Object.prototype.toString.call(function(){}) // "[object Function]" 
   Object.prototype.toString.call({name: 'An'}) // "[object Object]"
   ```

   `Object.prototype.toString.call()` 常用于判断浏览器内置对象时

2. instanceof：检测的是原型，判断一个实例是否属于某种类型

   `instanceof` 的内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。使用 `instanceof`判断一个对象是否为数组，`instanceof` 会判断这个对象的原型链上是否会找到对应的 `Array` 的原型，找到返回 `true`，否则返回 `false`。

   ```js
   []  instanceof Array; // true
   ```

   但 `instanceof` 只能用来判断对象类型，原始类型不可以。并且所有对象类型 instanceof Object 都是 true

   ```js
   []  instanceof Object; // true
   ```

3. Array.isArray()功能：

   用来判断对象是否为数组instanceof 与 isArray

   当检测Array实例时，`Array.isArray` 优于 `instanceof` ，因为 `Array.isArray` 可以检测出 `iframes`

   ```js
   var iframe = document.createElement('iframe');
   document.body.appendChild(iframe);
   xArray = window.frames[window.frames.length-1].Array;
   var arr = new xArray(1,2,3); // [1,2,3]
   
   // Correctly checking for Array
   Array.isArray(arr);  // true
   Object.prototype.toString.call(arr); // true
   // Considered harmful, because doesn't work though iframes
   arr instanceof Array; // false
   ```

   `Array.isArray()`是ES5新增的方法，当不存在 `Array.isArray()` ，可以用 `Object.prototype.toString.call()` 实现。

   ```js
   if (!Array.isArray) {
     Array.isArray = function(arg) {
       return Object.prototype.toString.call(arg) === '[object Array]';
     };
   }
   ```

4. **typeof**，可以判断原始数据类型

### ※8.17  为什么普通 `for` 循环的性能远远高于 `forEach` 的性能，请解释其中的原因。

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