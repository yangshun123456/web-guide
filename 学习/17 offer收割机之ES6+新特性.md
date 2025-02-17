
## 什么是 ES6+ 新特性

ES6+ 新特性就是指 ES6(也叫 ES2015) 以后更新的所有 JavaScript 标准规范新增的 API。因为 ES6 是一个划时代的更新，并且 ES6 之后每次迭代更新内容都比较少，所以就统称为 ES6+ 了。相对的 ES5 就是原本的 JS 语法，而现今很多框架设计也都是使用 ES6 标准语法进行书写，ES6+ 语法可以通过 babel 编译成 ES5 语法。

> 目前，ES6+ 已经更新到最新的 ES10 草案。

## ES6

### ES6 - let/const

这两个关键字对应一个名词，也就是块级作用域。ES5 之前是没有块级作用域这个概念的，并且定义变量也都是通过`var`来进行声明。`var`生命的变量是函数级作用域。

> 函数作用域含义：属于这个函数的全部变量都可以在整个函数的范围内使用及复用（在嵌套的作用域中也可以使用）。

+   ES5 函数作用域

```ini
var a = 1;
var b = 1;
function foo () {
    var a = 2;
    console.log('foo:', a, b); // 2， 1
}
console.log('windoiw:', a, b); // 1， 1
```

+   ES6 块级作用域 let

```ini
  {
    let c = 1;
    let d = 1;
    console.log('scope:', c, d); // 1, 1
  }
  function foo2 () {
    let c = 2;
    let d = 2;
    console.log('foo2:', c, d); // 2, 2
  }
  foo2();
  console.log('window:', c, d); // undefined, undefined
```

+   ES6 块级作用域 const

```ini
const aa = 1;
aa = 2; // Uncaught TypeError: Assignment to constant variable.

const bb = {};
bb.a = 1;
console.log(bb);

const cc = [];
cc.push(1);
console.log(cc);

const dd = null;
dd = 1;
console.log(dd); // Uncaught TypeError: Assignment to constant variable.
```

> const 的用法与 let 基本一致，只不过 const 定义的是常量，不能被修改。而如果const 定义的是对象或者数组，则还是可以改的，因为二者属于引用类型，存储的是地址指针，也就是说 const 代表的是变量地址不能被修改。**const 定义一个 null 值也是不能被更改的，因为还没被分配内存地址。**

var 定义的变是函数作用域，没有块的概念，可以跨块访问, 不能跨函数访问。

let 定义的变量是块级作用域，只能在块作用域里访问，不能跨块访问，也不能跨函数访问。

const 用来定义常量，使用时必须初始化(即必须赋值)，只能在块作用域里访问，而且不能修改。

### ES6 - class

经常使用 React 的同学应该非常的了解，class 用来声明类组件。而 class 就是 ES6 的语法。而 class 其实也可以换种说法是 ES5 继承的一个语法糖，class 特性可以使用 ES5 语法全部实现，只不过 class 更为便捷。

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  say() {
    const sayStr = `Hello, my name is ${this.name}, ${this.age} years old! I'm Person`;
    console.log(sayStr);
  }
}

class Chinese extends Person {
  constructor(name, age) {
    super(name, age);
    this.country = 'Chinese';
  }

  say() {
    const sayStr = `Hello, my name is ${this.name}, ${this.age} years old! I'm ${this.country} Person`;
    console.log(sayStr);
  }
}

/* ES6 class */
const person = new Person('luffy', 28);
const chinesePerson = new Chinese('周', 33);

person.say();
chinesePerson.say();
```

这里有个注意点就是，如果是子类继承父类，那么子类的`constructor`内部必须有`super()`，否则会报错，如下图：

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/17/170533adf55ba227~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

### ES6 - 箭头函数

箭头函数，ES6 最有特点的一个特性，让代码看起来简洁了很多，并且也解决了 this 指针的问题。

+   几种箭头函数的写法

```ini
() => console.log(1);
const a = () => 1;
const add = (a, b) => a + b;
const resFunc = (a, b) => () => a + b; // 返回一个函数
```

+   this 指针的指向

```javascript
var scope = 'window';
var obj = {
scope: 'obj',
funcScope: function() {
  console.log('当前作用域：', this.scope);
},
arrayFuncScope: () => console.log('当前作用域：', this.scope)
}
var obj2 = {
scope: 'obj2'
}
obj.funcScope();                 // obj
obj.arrayFuncScope();            // window
obj.funcScope.call(obj2);        // obj2 
obj.arrayFuncScope.call(obj2);   // window
```

可以看到，箭头函数与创建调用它的代码所在作用域共享一个 this，上面代码箭头函数在调用的时候在第2个和第4个都是在 window 环境下，因此 this 就是 window。

### ES6 - 模板字符串

关于模板字符串就是很简单了，如下两段代码就非常的清晰。

```ini
  /* 字符串 */
  var name = 'luffyZh';
  var es5Str = 'Hello ' + name; 
  const es6Str = `Hello ${name}`;
```

由上面代码可知，ES5 使用符号进行拼接，而 ES6 的模板字符串可以将变量嵌入字符串内，非常方便。

### ES6 - 解构赋值以及属性简写

+   解构赋值

解构赋值也是 ES6 新特性中被使用非常频繁的一个，并且在实际开发过程中用处很大。

```arduino
// 对象解构

const obj = { name: 'luffy', email: 'luffy@163.com' };
const { name, email, age } = obj;
console.log(name, email, age); // luffy luffy@163.com undefined
```

```scss
// 数组解构
let [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1, 2, 3

let [aa, bb, cc] = [1, [2 , 3], [4, 5, 6]];
console.log(aa, bb, cc); // 1 [2, 3] [4, 5, 6]
```

```ini
// 解构应用 —— 一行代码交换 a b 的值
let a = 1;
let b = 2;
[a, b] = [b, a];
console.log(a, b); // 2, 1
```

+   属性简写

这个也算是一个小语法糖吧，方便开发的时候快速简洁。

```ini
var name = 'luffyzh';
var email = 'luffyzh@163.com';
var person = {
    name,
    email
};
console.log(person); // {name: 'luffyzh', email: 'luffyzh@163.com'}
```

### ES6 - 模块化

ES6 为我们提供了新的模块导入/导出规范。

+   导入模块 —— import

```javascript
import React from 'react';
import { useState } from 'react';
import * as Sentry from 'sentry';
```

+   导出模块 —— export

```javascript
export const foo = () => 1; // 引入的时候需要带 {}
export default foo = () => 2; // 引入的时候不需要带 {}
```

### ES6 - Promise

### ES6 - Generator

ES6 的 Generator 也是一种异步编程解决方案，当初学的时候也是不理解，为啥 ES6 要提供两种异步方案，一个是 Promise 一个是 Generator，不过呢两者确实也都是被应用的十分广泛，各有千秋吧。

Generator 是一个迭代器生成函数，它被调用并不会立即执行，而是返回一个迭代器，然后可以进行异步调用，同时还可以挂起操作，非常的牛X但是使用起来也相对复杂。它有两个特性：

+   function关键字与函数名之间有一个星号
+   函数体内部使用yield表达式，定义不同的内部状态。

```javascript
function* testGenerator() {
    yield 'a';
    yield 'b';
    return 'c';
  }
var g = testGenerator();
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/170566323d1d039a~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

如上图所示，调用 Generator 函数它返回的是一个迭代器对象，然后通过调用此对象的方法`.next()`来一步一步执行函数内部的状态，每一个 yield 关键字对应着一个状态，而 yield 关键字则是表示在此处暂停执行的意思 （直到使用 next 调用），每个状态是一个对象有两个属性`{value: 此状态的值, done: 迭代器函数是否结束}`。

+   应用一： redux-saga

非常流行的 redux 异步处理方案 redux-saga 采用的就是 Generator 实现的。

+   应用二： antd dva

dva 框架内置处理redux 异步也是使用 Generator 来进行处理的。

### ES6 - Set

Set 对象是 ES6 为我们提供的一个新的数据结构，它是以键值对儿的形式存在，类似于 Map，但是区别在于 Set 对象具有自动去重的功能。

```scss
var setObj = new Set([1, 1, 2, 3, 4, 4]);
console.log(setObj); // {1, 2, 3, 4}
setObj.add(1);     // {1, 2, 3, 4}
setObj.add(5);   // {1, 2, 3, 4, 5}
setObj.keys(); // {1, 2, 3, 4, 5}
```

+   应用：一行代码实现数组去重

```vbnet
var arr = [1, 3, 3, 4, 5];
function uniqueArr(arr) {
    return [...new Set(arr)];
    // return Array.from(new Set(arr));
}
```

从上面可以看出来如下几点：

第一：Set 对象的构造函数可以接收一个数组，然后主动去重。

第二：Set 对象返回的是一个对象，可以通过`Array.from`方法转换成数组。

### ES6 - Symbol

Symbol 是 ES6 新增的一种基础数据类型，ES5 的时候，基础类型只有五种，它们是：`Number`、`String`、`Boolean`、`null`和`undefined`。而 ES6 新增了 `Symbol`，所以 ES6 的时候，基本数据类型就变成了 6 种。

> 强调一下，Object 属于复杂类型，并不是基础类型。

```javascript
var a = 1;
var b = '2';
var c = false;
var d = {};
var e = null;
var f = undefined;
var g = Symbol();
console.log(typeof a); // number
console.log(typeof b); // string
console.log(typeof c); // boolean
console.log(typeof d); // object
console.log(typeof e); // object
console.log(typeof f); // undefined
console.log(typeof g); // symbol
```

Symbol 是用来创建唯一性变量的，使用 Symbol 可以为程序创建唯一性的 ID，在创建的时候可以为其新增参数描述该变量，即使参数相同两个 Symbol 也是不同的。

```ini
var s1 = Symbol()
var s2 = Symbol('another symbol')
var s3 = Symbol('another symbol')

s1 === s2 // false
s2 === s3 // false
```

具体来说它的应用有如下几种：

+   可以用作对象的 key 值
+   可以用来定义系统常量
+   可以使用 Symbol 定义类的私有属性/方法

定义私有化方法很有趣，我们都知道，ES5 之前 JS 并没有私有化方法，类的属性和方法都是可以被访问的，以前也就是通过一些约定来实现，比如`_`下划线定义的是内部的，外部不要访问，但是也不是不能访问。而通过 symbol 和模块化机制，就可以实现私有化方法，因为类内部属性/方法通过 symbol 声明，在外部是无法新建一个一样的，这就是 symbol 唯一性的应用。

### ES6 - 扩展运算符

ES6 扩展运算符可以将数组对象转化成逗号分隔的参数序列，这同样也是非常便捷的 API。

```ini
var arr = [1, 2, 3];
console.log(arr);
console.log(...arr);          // 等价于 console.log(1, 2, 3);
```

+   应用 - 简便的实现数组复制

```ini
var arr = [1, 2, 3];
// ES5
var arrCopy = JSON.parse(JSON.stringify(arr));
console.log(arrCopy2 !== arr); // true
// ES6
var arrCopy2 = [...arr];
console.log(arrCopy2 !== arr); // true
```

### ES6 - 函数参数默认

```scss
function foo(name = 'luffy') {
    console.log(name);
}
foo(); // luffy
```

### ES6 - 其他好用便捷的 API

+   Array.from
    
    用来将类数组（伪数组对象转换成数组的）。
    

```css
var objLikeArr = {
   0: 'aaa',
   1: 'bbb',
   2: 'ccc',
   length: 3
 }
 console.log(Array.from(objLikeArr)); // ['aaa', 'bbb', 'ccc']
```

通常我们接触过的伪数组对象有，arguments、NodeList以及 Set 对象等等，都可以通过此函数进行数组的转化。

+   Array.isArray
    
    此函数用来判断对象是否是数组。
    

```javascript
// ES5判断
Object.prototype.toString.call([1, 2, 3]);  // "[object Array]"

// ES6 判断
Array.isArray({a: 1, b: 2}); // false
Array.isArray(Array.from([1, 2, 3])); // true
```

+   isNaN

```ini
var nan;
isNaN(nan + 1); true
nan + 1 === NaN; false
```

NaN 与 NaN 也不相等。

## ES7

### ES7 - Array.prototype.includes

includes 函数是 String 和 Array 共有的一个函数，他们都是用来判断子元素是否在一个对象（字符串/数组）内部。

```arduino
// String

'aaabbbccc'.includes('bbb'); // true
'aaabbbccc'.includes('ddd'); // false
```

```scss
// Array

[1, 2, 3, 4].includes(1); // true
[1, 2, 3, 4].includes(5); // false
[1, 2, NaN].indexOf(NaN); // -1
[1, 2, NaN].incldes(NaN); // true
```

可以看到，includes 与之前的 indexOf 的区别有两点：

+   indexOf 返回的是位置，number 类型；includes 返回的是 bool 类型
+   includes 可以判断 NaN 是否在数组里；indexOf 则不能

### ES7 - 幂指运算

这里就是一个语法糖。

```arduino
// ES5 求幂
Math.pow(3, 2); // 9

// ES7 求幂

3 ** 2； // 9
```

## ES8

### ES8 - Async/Await

### ES8 其他特性 API

之所以汇总是因为个人觉得除了 Async/Await 算是大块更新外，其他的都是某个小 API 或者语法糖，因此就简单说明一下。

+   Object.values/Object.entries

这个算是一个简化需求吧，之前我们想获取对象的value值，只能通过如下方法。

```javascript
var obj = {
    name: 'luffyzh',
    email: 'luffyzh@163.com'
}
// ES5
Object.keys(obj).map((key) => obj[key]);
```

现在，不仅能直接获取到 values，还能把 key-value 一起返回。

```javascript
// ES8
Object.values(obj);
Object.entries(obj);
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/1705671373642046~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

+   String Padding

字符串填充，新增了两个有关填充字符串的方法`String.prototype.padStart`和`String.prototype.padEnd`，允许将空字符串或其他字符串添加到原始字符串的开头或结尾。。

> targetLength:当前字符串需要填充到的目标长度。如果这个数值小于当前字符串的长度，则返回当前字符串本身。

> padString:(可选)填充字符串。如果字符串太长，使填充后的字符串长度超过了目标长度，则只保留最左侧的部分，其他部分会被截断，此参数的缺省值为 " "，注意，缺省值并不是空字符串，而是一个空格字符串。

```ini
var str = '0.0';
console.log(str.padStart(4, '10'));
console.log(str.padStart(4));
console.log(str.padEnd(10, 'x'));
console.log(str.padEnd(10));
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/1705678afce98919~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

+   Object.getOwnPropertyDescriptors()

用来返回一个对象的所有自身属性/方法的描述符,如果没有任何自身属性，则返回空对象。

```javascript
var obj = {
    name: 'luffyzh',
    email: 'luffyzh@163.com',
    say() {
      console.log(this.name);
    }
}
Object.getOwnPropertyDescriptors(obj);
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/170567bc6058ea53~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

## ES9/ES10

这两个放在一起来说是因为比较新，而且实际用到的个人感觉不是特别多，就把个人觉得可能会用到的列出来了。

### ES9 - Rest/Spread 对象扩展运算

ES6 出现了扩展运算符，但是只是给数组准备的，而扩展运算符非常好用，ES9 因此就扩展到了对象，现在对象也可以使用扩展运算来进行便捷操作。

```ini
var obj = {
    name: 'aaa',
    email: 'luffyzh@163.com'
}
var obj2 = {...obj};
console.log(obj2);
console.log(obj2 !== obj);
var obj3 = { age: 20, ...obj2 };
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/170569710ef18189~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

### ES9 - Promise.finally

### ES10 - String.prototype.matchAll

这个 API 个人感觉出现的有些晚了，因为匹配所有是一个很常见的需求，但是原来只能匹配第一个。

```javascript
 var str = '11223344113311';
console.log(str.match('11')); // 返回一个数组
console.log(str.matchAll('11')); // 返回一个迭代器

```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/170568e975062142~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

可以看到，matchAll 是通过 Generator 来实现的。

### ES10 - String的 trimStart()方法和 trimEnd()方法

这两个方法就很简单明了了，就是去除字符串前面和后面的空格。非常简单，就不做介绍了。

### ES10 - 新增基础类型 BigInt

所以现在基础类型就变成了：String、Number、Boolean、Null、Undefined、Symbol、BigInt 七种了。

ES5 - 五种 ES6 - 新增 Symbol，六种 ES10 - 新增 BigInt，七种

```ini
 var bi = BigInt(1111111111);
console.log(bi);
console.log(typeof bi);
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/2/18/1705689527ca9233~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

> bigint 在控制台会呈现绿色，number 则是蓝色，并且 bigint 后面会携带一额字符 n。如果不初始化还会报错，从报错可以看出 bigint 会在内部有一个转换。

## 总结

至此，ES6+当面试官面试的时候，你可以回答上来基本用法就可以了。不过每一个特性都是值得深入研究的，感兴趣的可以去查阅官方文档以及源码实现，这样能够加理解。

