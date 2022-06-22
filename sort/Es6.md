# ES6

## 1. let和const两个关键字与var之间有哪些不同？

1. var：函数级作用域，存在变量提升，声明前可以调用，允许重新赋值
2. const：块级作用域，不存在变量提升，声明前不可以调用，存在暂时性死区，不允许重新赋值
3. let：块级作用域，不存在变量提升，声明前不可以调用，存在暂时性死区，允许重新赋值

## 2. 扩展运算符（...）的用途有哪些？

扩展运算符的用途简单概括，可以分为以下三种。

#### 替代函数的apply()方法。

函数的apply()方法能够间接调用其它对象的方法，往往能收获奇效，

例如用Math对象的min()方法获取数组中的最小值，min()方法本来只接收一组参数，利用apply()方法后就能直接传递一个数组

```js
let arr = [1, 0, 2],
  min;
min = Math.min(1, 0, 2);                 //一组参数的调用方式
min = Math.min.apply(undefined, arr);    //利用apply()间接调用

min = Math.min(...arr);
```



#### 简化函数调用时传递实参的方式。

函数在被调用时，实参通常都是以逗号分隔的序列形式传递到函数体内。

如果实参的值被保存在数组中，那么就要一个一个的读取数组中指定位置的元素

```js
let date = [2018, 6, 9];
new Date(date[0], date[1], date[2]);     //2018-7-6

new Date(...date);                      //2018-7-6
```



#### 处理数组和字符串。

在扩展运算符出现之前，要执行数组的复制、合并等操作，需要调用数组的slice()、concat()、unshift()等方法。

这些方法到底是单独调用还是组合调用，由实际情况而定。下面是一个数组复制与合并的简单示例。

```js
let arr1 = [1, 2, 3],
  arr2,
  arr3;
arr2 = arr1.slice();       //复制数组
arr3 = arr2.concat(arr1);  //合并数组

arr2 = [...arr1];              //复制数组
arr3 = [...arr1, ...arr2];     //合并数组

let str = "strick";
str.split("");        //["s", "t", "r", "i", "c", "k"]
[...str];             //["s", "t", "r", "i", "c", "k"]
```



## 3. 剩余参数有什么作用？

在JavaScript的函数中，声明时定义的形参个数可以和传入的实参个数不同。

当实参个数大于形参个数时，ES6新增的剩余参数能把没有对应形参的实参收集到一个数组中。

有一点要注意，剩余参数不会影响函数的length属性，该属性的值表示形参个数。

下面是一个简单的示例。

```js
function func(name, ...args) {
  console.log(name);
  console.log(args[0]);
}
func("strick");         //首先输出"strick"，然后输出undefined
func("freedom", 29);    //首先输出"freedom"，然后输出29
console.log(func.length);         //1
```

## 4. 什么是解构

> https://zhuanlan.zhihu.com/p/53225287

解构是一种赋值语法，可从数组中提取元素或从对象中提取属性，将其值赋给对应的变量或另一个对象的属性。

解构地目的是简化提取数据的过程，增强代码的可读性。

有两种解构语法，分别是数组解构和对象解构

```js
var arr = [1, 2],
  obj = { a: 3, b: 4 },
  x, y, a, b;

var [x, y] = [1, 2];               //数组解构
var { a, b } = { a: 3, b: 4 };     //对象解构
```

#### 通用特性

解构赋值的语句可以包含或忽略声明关键字（例如var、let等），

如果包含，那么就需要初始化，否则会报语法错误，下面是两种不正确的写法。

```js
var [x, y];
var { a, b };
```

如果忽略声明关键字，那么在运行对象解构的时候，必须用圆括号包裹赋值表达式，而数组解构则并不强制，如下所示。

```js
[x, y] = [1, 2];                     //无圆括号的数组解构
({ a, b } = { a: 3, b: 4 });         //有圆括号的对象解构
```

## 5. 如果忽略声明关键字，那么在运行对象解构的时候，为何要用圆括号包裹赋值表达式（如下所示）？

```js
({ a, b } = { a: 3, b: 4 });
```

之所以要用圆括号包裹，是因为表达式左侧的花括号会被解析成代码块而不是对象字面量。如果把代码块和等号运算符放在一行，那么就会报语法错误。

## 6. 如何利用数组解构交换变量

```js
var x = 1, y = 2;
[x, y] = [y, x];     //数组解构
```

## 7. 什么是模板字面量

> https://zhuanlan.zhihu.com/p/53229895

模板字面量是一种能够嵌入表达式的格式化字符串，有别于普通字符串，它使用反引号（`）包裹字符序列，而不是双引号或单引号。

模板字面量包含特定形式的占位符（${expression}）：由美元符号、大括号以及合法的表达式组成，合法的表达式（expression）可以是变量、算术或函数调用，甚至还可以是模板字面量。

在ES6引入模板字面量后，就能避免用若干个加号来实现字符串拼接，而改用更为优雅的语法来替代

## 8. 什么是类型化数组

> https://zhuanlan.zhihu.com/p/54129176

类型化数组（Typed Array）是一种处理二进制数据的特殊数组，它可像C语言那样直接操纵字节，不过得先用ArrayBuffer对象创建数组缓冲区（Array Buffer），再映射到指定格式的视图（view）之后，才能读写其中的数据

1. [请谈谈你对Unicode的理解。](https://github.com/pwstrick/daily/issues/124)

2. [什么叫Unicode标准化？](https://github.com/pwstrick/daily/issues/125)

3. ["My name is strick".includes("name")返回的结果为__________。](https://github.com/pwstrick/daily/issues/126)

4. [正则表达式的u标志有什么作用？](https://github.com/pwstrick/daily/issues/127)

5. [正则表达式的y标志有什么作用？](https://github.com/pwstrick/daily/issues/128)

6. [如何判断一个字符是由两个编码单元组成的？](https://github.com/pwstrick/daily/issues/129)

7. [如何使用Object.assign()？](https://github.com/pwstrick/daily/issues/131)

8. [在ES6中，自有属性的枚举顺序是怎样的？](https://github.com/pwstrick/daily/issues/132)

9. [Array.of()有什么作用？](https://github.com/pwstrick/daily/issues/133)

10. [使用fill()和copyWithin()需要的注意点有哪些？](https://github.com/pwstrick/daily/issues/134)

11. [find()和indexOf()有哪些区别？](https://github.com/pwstrick/daily/issues/135)

12. [什么是类型化数组？](https://github.com/pwstrick/daily/issues/136)

13. [类型化数组与常规数组有哪些异同？](https://github.com/pwstrick/daily/issues/137)

14. [如何使用DataView？](https://github.com/pwstrick/daily/issues/138)

15. [ES6为函数做了哪些改良？](https://github.com/pwstrick/daily/issues/139)

16. [函数的length属性有什么作用？](https://github.com/pwstrick/daily/issues/140)

17. [什么是块级函数？](https://github.com/pwstrick/daily/issues/141)

18. [new.target是由ES6引入的一个元属性，它有何用途？](https://github.com/pwstrick/daily/issues/142)

19. [如何理解尾调用优化？](https://github.com/pwstrick/daily/issues/145)

20. [WeakSet和Set有哪些差异？](https://github.com/pwstrick/daily/issues/146)

21. [如何理解ES6新增的数据结构Map？](https://github.com/pwstrick/daily/issues/147)

22. [什么是迭代器？](https://github.com/pwstrick/daily/issues/148)

23. [什么样的对象是可迭代的？](https://github.com/pwstrick/daily/issues/149)

24. [如何使用for-of循环？](https://github.com/pwstrick/daily/issues/150)

25. [function*用来做什么？](https://github.com/pwstrick/daily/issues/151)

26. [如何通过生成器实现异步编程？](https://github.com/pwstrick/daily/issues/153)

27. [ES6的类比起用构造函数模拟的类，有哪些独有的特性？](https://github.com/pwstrick/daily/issues/154)

28. [类有哪些成员？](https://github.com/pwstrick/daily/issues/155)

29. [当super作为方法使用时，有哪些注意点？](https://github.com/pwstrick/daily/issues/156)

30. [怎么实现类的继承？](https://github.com/pwstrick/daily/issues/157)

31. [怎么理解Symbol.species？](https://github.com/pwstrick/daily/issues/158)

    

32. [如何理解thenable？](https://github.com/pwstrick/daily/issues/161)

    

33. [什么是代理？](https://github.com/pwstrick/daily/issues/163)

34. [什么是反射？它有什么用途？](https://github.com/pwstrick/daily/issues/164)

35. [执行[1, 2, 3, 4, 5\].copyWithin(3, 2)得到的数组为__________。](https://github.com/pwstrick/daily/issues/165)

36. [如何将Map转换成数组？](https://github.com/pwstrick/daily/issues/166)

37. [yield和return有哪些区别？](https://github.com/pwstrick/daily/issues/167)

38. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/728)

```
    function sayHi() {
      console.log(name);
      console.log(age);
      var name = "Lydia";
      let age = 21;
    }
    sayHi();
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/730)

```
const shape = {
  radius: 10,
  diameter() {
    return this.radius * 2
  },
  perimeter: () => 2 * Math.PI * this.radius
}

shape.diameter()
shape.perimeter()
```

1. [哪个选项是不正确的？](https://github.com/pwstrick/daily/issues/731)

```
const bird = {
  size: "small"
};
const mouse = {
  name: "Mickey",
  small: true
};
```

A: mouse.bird.size
B: mouse[bird.size]
C: mouse[bird["size"]]
D: All of them are valid

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/732)

```
let c = { greeting: "Hey!" };
let d;

d = c;
c.greeting = "Hello";
console.log(d.greeting);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/733)

```
let a = 3;
let b = new Number(3);
let c = 3;

console.log(a == b);
console.log(a === b);
console.log(b === c);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/734)

```
class Chameleon {
  static colorChange(newColor) {
    this.newColor = newColor;
  }

  constructor({ newColor = "green" } = {}) {
    this.newColor = newColor;
  }
}

const freddie = new Chameleon({ newColor: "purple" });
freddie.colorChange("orange");
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/735)

```
let greeting;
greetign = {};
console.log(greetign);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/736)

```
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

const member = new Person("Lydia", "Hallie");
Person.getFullName = () => this.firstName + this.lastName;

console.log(member.getFullName());
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/742)

```
function getPersonInfo(one, two, three) {
  console.log(one);
  console.log(two);
  console.log(three);
}
const person = "Lydia";
const age = 21;
getPersonInfo`${person} is ${age} years old`;
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/744)

```
function getAge(...args) {
  console.log(typeof args);
}
getAge(21);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/747)

```
const obj = { 1: "a", 2: "b", 3: "c" };
const set = new Set([1, 2, 3, 4, 5]);

obj.hasOwnProperty("1");
obj.hasOwnProperty(1);
set.has("1");
set.has(1);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/748)

```
const obj = { a: "one", b: "two", a: "three" };
console.log(obj);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/750)

```
for (let i = 1; i < 5; i++) {
  if (i === 3) continue;
  console.log(i);
}
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/751)

```
String.prototype.giveLydiaPizza = () => {
  return "Just give Lydia pizza already!";
};
const name = "Lydia";
name.giveLydiaPizza();
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/752)

```
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"));
const baz = () => console.log("Third");
bar();
foo();
baz();
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/755)

```
const person = { name: "Lydia" };
function sayHi(age) {
  console.log(`${this.name} is ${age}`);
}
sayHi.call(person, 21);
sayHi.bind(person, 21);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/756)

```
function sayHi() {
  return (() => 0)();
}
typeof sayHi();
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/758)

```
const numbers = [1, 2, 3];
numbers[10] = 11;
console.log(numbers);
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/759)

```
(() => {
  let x, y;
  try {
    throw new Error();
  } catch (x) {
    (x = 1), (y = 2);
    console.log(x);
  }
  console.log(x);
  console.log(y);
})();
```

1. [下面代码的输出是什么？](https://github.com/pwstrick/daily/issues/760)

```
[[0, 1], [2, 3]].reduce(
  (acc, cur) => {
    return acc.concat(cur);
  },
  [1, 2]
);
```

1. [发布订阅模式](https://github.com/Liqiuyue9597/front-end-interview/issues/66)[★]



### 3. 讲讲set结构

- es6方法,Set本身是一个 **构造函数**，它类似于数组，但是成员值都是唯一的。

```
const set = new Set([1,2,3,4,4])
console.log([...set] )// [1,2,3,4]
console.log(Array.from(new Set([2,3,3,5,6]))); //[2,3,5,6]
```

### 4.去除重复数组，去除字符串里面的重复字符

```
// 去除数组的重复成员
[...new Set(array)]
//去除字符串里面的重复字符
[...new Set('ababbc')].join('')
```

### 5.如何拦截变量属性访问

- 使用proxy。**new Proxy()** 表示生成一个 **Proxy** 实例，**target** 参数表示所要拦截的目标对象，**handler** 参数也是一个对象，用来定制拦截行为。

```
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});
let obj = Object.create(proxy);
obj.time // 35
```

### 6.忽略enumerable为false的属性的四个操作？

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象。描述对象的enumerable属性，称为“可枚举性”，如果该属性为false，就表示某些操作会忽略当前属性。

- for...in循环：只遍历对象自身的和继承的可枚举的属性。
- Object.keys()：返回对象自身的所有可枚举的属性的键名。
- JSON.stringify()：只串行化对象自身的可枚举的属性。
- Object.assign()： 忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性。

### 7.属性的五种遍历方法？

- for...in循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。
- Object.keys返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。
- Object.getOwnPropertyNames返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
- Object.getOwnPropertySymbols返回一个数组，包含对象自身的所有 Symbol 属性的键名。
- Reflect.ownKeys返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

### 8.CommonJS模块与es6模块的差异？

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
- CommonJS 是同步导入，因为用于服务端，文件都在本地，同步导入即使卡住主线程影响也不大。而es6模块是异步导入，因为用于浏览器，需要下载文件，如果也采用同步导入会对渲染有很大影响



### 17. WeakSet 与 Set 的区别

- WeakSet 只能储存对象引用，不能存放值，而 Set 对象都可以
- WeakSet 对象中储存的对象值都是被弱引用的，即垃圾回收机制不考虑 WeakSet 对该对象的应用，如果没有其他的变量或属性引用这个对象值，则这个对象将会被垃圾回收掉（不考虑该对象还存在于 WeakSet 中），所以，WeakSet 对象里有多少个成员元素，取决于垃圾回收机制有没有运行，运行前后成员个数可能不一致，遍历结束之后，有的成员可能取不到了（被垃圾回收了），WeakSet 对象是无法被遍历的（ES6 规定 WeakSet 不可遍历），也没有办法拿到它包含的所有元素

- [面试官：ES6中数组新增了哪些扩展?](https://github.com/febobo/web-interview/issues/35)
- [面试官：ES6中对象新增了哪些扩展?](https://github.com/febobo/web-interview/issues/36)
- [面试官：ES6中函数新增了哪些扩展?](https://github.com/febobo/web-interview/issues/37)
- [面试官：ES6中新增的Set、Map两种数据结构怎么理解?](https://github.com/febobo/web-interview/issues/38)
- [面试官：你是怎么理解ES6中 Promise的？使用场景？](https://github.com/febobo/web-interview/issues/40)
- [面试官：怎么理解ES6中 Generator的？使用场景？](https://github.com/febobo/web-interview/issues/41)
- [面试官：你是怎么理解ES6中Proxy的？使用场景?](https://github.com/febobo/web-interview/issues/42)
- [面试官：你是怎么理解ES6中Module的？使用场景？](https://github.com/febobo/web-interview/issues/43)
- [面试官：你是怎么理解ES6中 Decorator 的？使用场景？](https://github.com/febobo/web-interview/issues/44)

### 