# 变量

## 原始值与引用值

**原始值（简单数据类型）：** JavaScript总共包含 **6种** 简单数据类型：`Undefined`、`Null`、`Boolean`、`Number`、`String` 和 `Symbol`。原始值是直接存储在栈中的简单数据段，也就是说，它们的值直接存储在变量访问的位置，我们可以直接 **按值访问** 。

**引用值（复杂数据类型）：** JavaScript中的引用值有很弱，如：`Object`、`Function`、`Array`等。由于引用值的大小会改变，所以不能将其存放在栈中，否则会降低变量查询速度，因此，其值存储在堆存储在堆中的对象，而放在变量的栈空间中的值是该对象存储在堆中的地址，也就是说，存储在变量处的值是一个指针（内存地址），指向存储对象的堆内存中。在操作对象时，实际上操作的是对该对象的引用（reference）而非实际的对象本身。所以，保存引用值的变量是 **按引用访问** 的。

## 动态属性

对于引用值而言，可以随时添加、修改和删除其属性和方法，所以引用值的大小会改变。比如，定义一个名为`person`的`Object`，并给其添加属性`name`，在这之后就能访问这个属性，直到`name`属性被删除或者`person`对象被销毁。

```javascript
let person = new Object();
person.name = "Nicholas";
console.log(person.name);	// Nicholas

// 删除name属性
delete(person.name);

// 销毁person对象，这里将person变量指向null，这样垃圾回收程序就会自动收回person之前指向的对象
person = null
```

那么给原始值添加属性会发生什么呢？

```javascript
// 非严格模式下，不会报错，但是无法给变量添加属性
let name = "Nicholas";
name.age = 27;
console.log(name.age);	// undefined
```

```javascript
// 在严格模式下，直接在添加变量的时候报错
"use strict";
let name = "Nicholas";
name.age = 27;	// TypeError: Cannot create property 'age' on string 'Nicholas'
```

这里注意原始值必须使用字面量声明，使用包装类的是行为类似原始值的对象！！！（这里详情可以看第五章）

```javascript
let name1 = "Nicholas";
let name2 = new String("Matt");
name1.age = 27;
name2.age = 26;
console.log(name1.age); // undefined
console.log(name2.age); // 26
console.log(typeof name1); // string
console.log(typeof name2); // object
```

![image-20211020150852317](https://cdn.jsdelivr.net/gh/Limbo-Y/MyImg@master/JavaScript-Learning/image-20211020150852317.2atb4qx0qatc.png)

## 值的复制

其实原始值和引用值的复制都是将栈中的内容复制，但是由于栈内存储的内容不同，原始值将内容完整复制一份，而引用值是将指针复制了一份，所以会出现一个改动其它全部都改动的现象。

```javascript
// 原始值的复制
let num1 = 5;
let num2 = num1;
```

![image-20211020151647907](https://cdn.jsdelivr.net/gh/Limbo-Y/MyImg@master/JavaScript-Learning/image-20211020151647907.1o7p8oyg461s.png)

```javascript
// 引用值的复制
let obj1 = new Object();
let obj2 = obj1;
obj1.name = "Nicholas";
console.log(obj2.name); // "Nicholas"
```

![image-20211020152218360](https://cdn.jsdelivr.net/gh/Limbo-Y/MyImg@master/JavaScript-Learning/image-20211020152218360.44gxr6ipets0.png)

## 传递参数

JavaScript中的所有函数都是**按值传递参数**的，也就是使用了上面说的复制大法。

```javascript
// 传递一个原始值
function addTen(num) {
    num += 10;
}
let num = 20;
console.log(num);	// 20
addTen(num);
console.log(num);	// 20
```

```javascript
// 传递一个引用值
function setName(obj) {		// 这里obj复制了和person一样的指向
    obj.name = "Nicholas";	// 添加属性name
    obj = new Object();		// 这里obj换了一个新的指向
	obj.name = "Greg";		// 添加属性name，由于obj现在指向的对象与person不同，所以person指向的对象的name属性不变
}
let person = new Object();
setName(person);
console.log(person.name); 	// "Nicholas"
```

## 确定类型

### typeof

原始值可以使用`typeof`来判断数据类型。

```javascript
console.log(typeof undefined);				// undefined
console.log(typeof null);					// object
console.log(typeof true);					// boolean
console.log(typeof new Boolean(true));		// object
console.log(typeof 2);						// number
console.log(typeof new Number(2));			// object
console.log(typeof "hello");				// string
console.log(typeof new String("hello"));	// object
console.log(typeof Symbol());				// symbol
console.log(typeof {});						// object
console.log(typeof function(){});			// function
console.log(typeof []);						// object
```

### instanceof

`typeof`虽然对原始值很有用，但它对引用值的用处不大。如果要具体知道对象是属于哪一种类型，可以使用`instanceof`操作符。

`A instanceof B`用来判断A是否为B的实例。如果A是B的实例，则返回true，否则false。

> 原理：instanceof是检测原型。如果用instanceof 检测原始值，则始终会返回false，因为原始值不是对象。

用字面量的创建的字符串，数字，布尔，instanceof不能正确判断，如果被包装类实例化了就能判断了；用构造函数创建（包装类）的字符串，数字，布尔，既属于Object，又属于各自的类型，毕竟Object是个原型老祖宗。

```javascript
console.log(2 instanceof Number); 					// false
console.log(true instanceof Boolean); 				// false
console.log('str' instanceof String); 				// false

console.log([] instanceof Array); 					// true
console.log([] instanceof Object); 					// true
console.log(function(){} instanceof Function); 		// true
console.log(function(){} instanceof Object); 		// true
console.log({} instanceof Object); 					// true

console.log(new Number(2) instanceof Number);		// true
console.log(new Boolean(true) instanceof Boolean);	// true
console.log(new String('str') instanceof String);	// true
```

### constructor

使用constructor也可以判断数据类型，而且不论是由字面量创建的原始值还是包装类实例化的对象都能正确判断

> 原理：constructor就是检查实例的构造函数

```javascript
console.log((2).constructor === Number);					// true
console.log((new Number(2)).constructor === Number);		// true
console.log((true).constructor === Boolean);				// true
console.log((new Boolean(true)).constructor === Boolean);	// true
console.log(('hello').constructor === String);				// true
console.log((new Number('hello')).constructor === String);	// true
console.log(([]).constructor === Array);					// true
console.log((function() {}).constructor === Function);		// true
console.log(({}).constructor === Object);					// true

console.log((2).constructor === Object);					// false
console.log((true).constructor === Object);					// false
console.log(('str').constructor === Object);				// false
console.log(([]).constructor === Object);					// false
console.log((function() {}).constructor === Object);		// false
```

用costructor来判断类型看起来是完美的，然而，如果我创建一个对象，更改它的原型，这种方式也变得不可靠了

```javascript
function Fn(){};
Fn.prototype = new Array();

let f = new Fn();
console.log(f.constructor === Fn); 		// false
console.log(f.constructor === Array); 	// true
```

我的理解f就是实例1，new Array()创建了实例2，在f的原型对象找constructor属性，f的原型对象是实例2，所以在实例2的原型对象找constructor属性，结果是Array构造函数

### Object.prototype.toString.call()

这种方式能精准地得到数据类型

```javascript
let a = Object.prototype.toString;
console.log(a.call(2));						// [object Number]
console.log(a.call(new Number(2)));			// [object Number]
console.log(a.call(true));					// [object Boolean]
console.log(a.call(new Boolean(true)));		// [object Boolean]
console.log(a.call('hello'));				// [object String]
console.log(a.call(new String('hello')));	// [object String]
console.log(a.call([]));					// [object Array]
console.log(a.call(function(){}));			// [object Function]
console.log(a.call({}));					// [object Object]
console.log(a.call(undefined));				// [object Undefined]
console.log(a.call(null));					// [object Null]
```



# 执行上下文、作用域、闭包

## 执行上下文

执行上下文 ( execution context ) 就是当前 JavaScript 代码被解析和执行时所在环境的抽象概念。

变量或函数的上下文决定了它们可以访问哪些数据，以及它们的行为，这个概念在 JavaScript 中非常重要。每个上下文都有一个关联的**变量对象**，而这个上下文中定义的所有变量和函数都存在于这个对象上。

全局上下文是最外层的上下文，在浏览器中全局上下文就是我们常说的 window 对象（在 node.js 环境中就不是window了），也就是说window就是全局上下文关联的变量对象，所有通过 var 定义的全局变量和函数都会成为 window 对象的属性和方法。

- 全局执行上下文：默认、最底层、全局唯一；
- 函数执行上下文：每次调用函数时，都会为该函数创建一个新的执行上下文，包括调用自己；
- Eval 函数执行上下文：运行在 eval 函数中的代码拥有自己的执行上下文。

### 执行上下文堆栈

在一个 JavaScript 程序中，必定会产生多个执行上下文，JavaScript 引擎会以栈的方式来处理它们，这个栈我们称其为函数调用栈 ( call stack )。栈底永远都是全局上下文，而栈顶就是当前正在执行的上下文。

当浏览器首次载入脚本，默认进入全局执行上下文。每当发生函数调用，将创建一个新的执行上下文，并将新创建的上下文压入执行栈的顶部。浏览器将总会执行栈顶的执行上下文，一旦当前上下文函数执行结束，它将被从栈顶弹出，并将上下文控制权交给当前的栈。

### 执行上下文的创建与执行

在创建阶段，总共发生三件事情：

1. 确定 this 的值，也被称为 This Binding。
2. LexicalEnvironment（词法环境） 组件被创建。
3. VariableEnvironment（变量环境） 组件被创建。

```javascript
ExecutionContext = {  
    ThisBinding = <this value>,  
    LexicalEnvironment = { ... },  
    VariableEnvironment = { ... },  
}
```

#### This Binding

- 全局执行上下文中，this 的值指向全局对象，在浏览器中，this 的值指向 window 对象
- 函数执行上下文中，this 的值取决于函数的调用方式。如果它被一个对象引用调用，那么 this 的值被设置为该对象，否则 this 的值被设置为全局对象或 undefined（严格模式下）
- 在构造函数模式中，类中 (函数体中) 出现的 this.xxx=xxx 中的 this 是当前类的一个实例
- call、apply 和 bind，this 是第一个参数
- 箭头函数没有自己的 this，看其外层的是否有函数，如果有，外层函数的 this 就是内部箭头函数的 this，如果没有，则 this 是 window

#### LexicalEnvironment

词法环境由两部分组成：

- 环境记录（EnvironmentRecord）：存储变量和函数声明的实际位置
- 对外部环境的引用（outer）：访问外部词法环境

分为两种类型：

- 全局词法环境：
  - EnvironmentRecord 叫*对象环境记录*，包含一个全局对象（window 对象）及其关联的方法和属性（例如数组方法）以及任何用户自定义的全局变量，this 的值指向这个全局对象；
  - outer 为 `null`。

- 函数词法环境：
  - EnvironmentRecord 叫*声明性环境记录*， 除了相应的变量和函数声明，还包含了一个 `arguments` 对象，该对象包含了索引和传递给函数的参数之间的映射以及传递给函数的参数的长度（数量）；
  - outer 可以是全局环境，也可以是包含内部函数的外部函数环境。

#### VariableEnvironment

变量环境也是一个词法环境，具有上面定义的词法环境的所有属性。

在ES6中，LexicalEnvironment 组件和 VariableEnvironment 组件的区别在于前者用于存储函数声明和变量（ let 和 const ）绑定，而后者仅用于存储变量（ var ）绑定。

举个例子：

```javascript
let a = 20;  
const b = 30;  
var c;

function multiply(e, f) {  
    var g = 20;  
    return e * f * g;  
}

c = multiply(20, 30);
```

对应的执行上下文：

```javascript
// 全局执行上下文
GlobalExectionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      // 标识符绑定在这里  
      a: < uninitialized >,    // 全局变量声明（let 或 const 定义的变量，创建时未初始化，声明前使用报错，执行阶段会置为 undefined）
      b: < uninitialized >,  
      multiply: < func >  // 全局函数声明
    }  
    outer: <null>  
  },

  VariableEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      // 标识符绑定在这里  
      c: undefined,  // 全局变量声明（var 定义的变量，声明前访问不报错）
    }  
    outer: <null>  
  }  
}

// multiply函数局部执行上下文
FunctionExectionContext = {  
   
  ThisBinding: <Global Object>,

  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      // 标识符绑定在这里  
      Arguments: {0: 20, 1: 30, length: 2},  // 函数参数
    },  
    outer: <GlobalLexicalEnvironment>  
  },

  VariableEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      // 标识符绑定在这里  
      g: undefined  // 函数内部变量声明
    },  
    outer: <GlobalLexicalEnvironment>  
  }  
}

```

## 作用域

作用域是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。同样的，作用域也分为全局作用域和局部作用域。

全局作用域包含以下几种情境：

1. 最外层函数和在最外层函数外面定义的变量拥有全局作用域
2. 所有末定义直接赋值的变量自动声明为拥有全局作用域
3. 所有window对象的属性拥有全局作用域

局部作用域又叫做函数作用域，是指声明在函数内部的变量。只能在局部访问。

```javascript
var a = 1;
function fn() {
    var sum = 0;
    alert(a); // 1   局部访问全局
}
fn(); // 先执行
alert(sum); // 报错  全局访问局部
```

执行上下文与作用域的区别：

- 作用域在函数定义时确定，而不是在函数调用时确定
- 作用域中没有变量，要通过作用域对应的执行上下文环境来获取变量的值
- 同一个作用域下，不同的调用会产生不同的执行上下文环境，继而产生不同的变量的值
- 作用域中变量的值是在执行过程中产生的确定的

### 作用域链

JavaScript里一切都是对象，函数也是对象。函数对象和其它对象一样，拥有可以通过代码访问的属性和一系列仅供JavaScript引擎访问的内部属性。其中一个内部属性是`[[Scope]]`，该内部属性包含了函数被创建的作用域中对象的集合，这个集合被称为函数的作用域链，它决定了哪些数据能被函数访问。

函数在执行的过程中，先从自己内部寻找变量，如果找不到，再从 **创建当前函数所在的作用域** 去找，从此往上，也就是向上一级找，直到找到全局作用域。

通过例子理解一下：

```javascript
let name = 'wby';
let a = {
    sayName: function(){
        console.log(name);	// name这个不在当前作用域中声明的变量叫自由变量
    }
}
a.sayName(); //wby

let person1 = function (){
    let name = 'lyy';
    let skill = a.sayName;
    skill(); //wby
}
person1();
```

这里先声明了两个变量 name 和 a，a是一个对象里面包含一个名为sayName的函数，当执行通过a调用sayName函数`a.sayName()`，打印wby，这块比较好理解，当访问一个变量时，首先在当前的局部上下文查找，查找到第一行定义的name变量，打印name变量的值。

然后声明了一个person1函数，函数中又声明了一个重名的name变量，一个skill变量被赋值为对象a中的sayName函数，然后调用skill函数。调用person1还是打印了wby，不是说先在当前的局部上下文查找变量吗，明明在person1这个上下文中含有名为name的变量，为啥不打印lyy呢？因为函数的作用域是声明时的作用域，不是执行时的作用域，所以在调用skill函数时，寻找name变量的过程和之前一样，要到创建sayName函数的作用域中找，不管在哪调用。

## var、let 与 const

### var

var的作用域是函数作用域，在函数中使用 var 声明的变量，作用域只存在于该函数中，如果在函数中没有声明变量就进行了初始化，那么该变量就会自动被添加到全局上下文。

```javascript
function add(num1, num2) {
    var sum = num1 + num2;
    return sum;
}
let result = add(10, 20); // 30
console.log(sum); // 报错：sum 在这里不是有效变量
```

```javascript
function add(num1, num2) {
    sum = num1 + num2;
    return sum;
}
let result = add(10, 20); // 30
console.log(sum); // 30
```

```javascript
// var 与 function 的变量提升，能在声明变量之前访问到
// 结合之前的执行上下文过程，var声明的变量在代码运行前就被绑定在当前上下文中，并初始化为undefined

console.log(typeof foo); // function
console.log(typeof bar); // undefined

var bar = function() {
    return 'world';
};

function foo() {
    return 'hello';
}
```

### let

let的作用域是块级作用域，凡是用`{}`括起来的区域，在里面使用 let 声明的变量的作用域就是`{}`括起来的区域，而且 let 不能对一个变量进行声明二次声明。暂时性死区：变量在声明前，是不能使用的。

```javascript
{
    let d;
}
console.log(d); // ReferenceError: d 没有定义
```

```javascript
var a;
var a;
// 不会报错
{
    let b;
    let b;
}
// SyntaxError: 标识符b 已经声明过了
```

```javascript
// 暂时性死区
function fn() {
	name = 'ZhangSan';
    let name;
}
```

### const

const 基本上与 let 的用法一致，只是由 const 声明的变量的值不能进行修改，这里注意对于引用值来说，也就是栈空间中存储的指针不能更改，对象里面的内容还是可以改的。

```javascript
const o1 = {};
o1 = {}; // TypeError: 给常量赋值

const o2 = {};
o2.name = 'Jake';
console.log(o2.name); // 'Jake'

// 如果想让整个对象都不能修改，可以使用Object.freeze()，这样再给属性赋值时虽然不会报错，但会静默失败，严格模式下会直接报错
const o3 = Object.freeze({});
o3.name = 'Jake';
console.log(o3.name); // undefined
```

## 闭包

闭包的概念：能够读取其他函数内部变量的函数。

闭包产生的两种情况：

- 函数作为返回值

  ```javascript
  var x = 10;
  function fn() {
      var x = 20;
      return function f() {
          console.log(x);
      };
  }
  
  var show = fn();
  // 执行 fn() 时，返回的是一个函数。函数的特别之处在于可以创建一个独立的作用域。
  // 而正巧合的是，返回的这个函数体中，还有一个自由变量 x 要引用 fn 作用域下的上下文环境中的 x。
  // 因此到这里 fn() 的执行上下文不被销毁，全局上下文变为活跃状态
  
  show(); // 函数作为返回值，显示 20
  ```

- 函数作为返回值，显示 20函数作为参数传递

  ```javascript
  var x = 10;
  function fn() {
      console.log(x);
  }
  
  (function (f) {
      var x = 20;
      f(); 
  })(fn);	// 显示 10
  // fn 函数作为参数传递，到创建 fn 函数的作用域中找
  ```



# 垃圾回收

## 标记清理法

标记清理法分为 **标记** 和 **清理** 两个阶段。标记阶段即为所有活动对象做上标记，清理阶段则把没有标记（也就是非活动对象）销毁。

- 垃圾收集器在运行时会给内存中的所有变量都加上一个标记，假设内存中所有对象都是垃圾，把所有的标记都清空

  ![image](https://cdn.jsdelivr.net/gh/Limbo-Y/MyImg@master/JavaScript-Learning/image.1y0v42yasbv.png)

- 然后从各个根对象开始遍历（根对象可以理解为window对象，全局作用域），把不是垃圾的节点（活动对象）打上标记

  ![image](https://cdn.jsdelivr.net/gh/Limbo-Y/MyImg@master/JavaScript-Learning/image.2xuv9immlu20.png)

- 清理所有没有标记的垃圾，销毁并回收它们所占用的内存空间

- 等待下一轮垃圾回收

## 引用计数法

引用计数法顾名思义，就是记录变量被引用的次数。把对象是否不再需要简化定义为对象有没有其他对象引用到它，如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收

- 当声明了一个变量并且将一个引用类型赋值给该变量的时候这个值的引用次数就为 1

- 如果同一个值又被赋给另一个变量，那么引用数加 1
- 如果该变量的值被其他的值覆盖了，则引用次数减 1
- 当这个值的引用次数变为 0 的时候，说明没有变量在使用，这个值没法被访问了，回收空间，垃圾回收器会在运行的时候清理掉引用次数为 0 的值占用的内存

```javascript
function test(){
    let A = new Object()
    let B = new Object()

    A.b = B
    B.a = A
}
```

存在的问题：循环引用。对象 A 和 B 通过各自的属性相互引用着，按照上文的引用计数策略，它们的引用数量都是 2，但是，在函数 test 执行完成之后，对象 A 和 B 是要被清理的，但使用引用计数则不会被清理，因为它们的引用数量不会变成 0，仍然为2（A自身指向Object，B的属性a也指向A指向的Object）。假如此函数在程序中被多次调用，那么就会造成大量的内存不会被释放。

再用标记清除的角度看一下，当函数结束后，两个对象都不在作用域中，A 和 B 都会被当作非活动对象来清除掉，相比之下，引用计数则不会释放，也就会造成大量无用内存占用，这也是后来放弃引用计数，使用标记清除的原因之一。

**优雅的代码书写提高代码性能**

1. 解除引用
   其实就是把不用的变量置null，解除变量的引用不仅可以消除循环引用，而且对垃圾回收也有帮助。

2. 使用 let 和 const 声明变量
   let 和 const 声明的变量是块级作用域，相比于 var 的变量提升和函数作用域，能更早地变为非活动对象被回收

3. 警惕内存泄露

   ```javascript
   // 这里name没有使用关键字声明就初始化了，那么就会自动被添加到全局上下文，作为window的属性。如果window不销毁，name就会一直存在
   function setName() {
       name = 'Jake';
   }
   ```

   ```javascript
   // 定时器也可能会悄悄地导致内存泄漏，只要定时器一直运行，那么name变量就会一直被引用
   let name = 'Jake';
   setInterval(() => {
       console.log(name);
   }, 100);
   ```

   ```javascript
   // 闭包造成内存泄露，只要返回的函数存在，那么name变量就不会被清除
   let outer = function() {
       let name = 'Jake';
       return function() {
           return name;
       };
   };
   ```

4. 静态分配与对象池

   这里有一个addVector函数，传入a和b两个对象，计算两个对象的和并赋值给新建的对象resultant，最后返回resultant。

   如果resultant对象的生命周期很短，比如在这个addVector函数函数中，就4行代码，它会很快失去所有对它的引用，成为可以被回收的值。假如addVector函数被频繁地调用，就不断有resultant对象被创建，失去引用，被回收。

   ```javascript
   function addVector(a, b) {
       let resultant = new Vector();
       resultant.x = a.x + b.x;
       resultant.y = a.y + b.y;
       return resultant;
   }
   
   let a = {x: 10, y: 5}, b = {x: -3, y: 6};
   let resultant = addVector(a, b);
   ```

   静态分配：比起上述的函数定义方法，可以使用一个在函数外部定义的对象

   ```javascript
   function addVector(a, b, resultant) {
       resultant.x = a.x + b.x;
       resultant.y = a.y + b.y;
   }
   
   let a = {x: 10, y: 5}, b = {x: -3, y: 6}, resultant = {};
   addVector(a, b, resultant);
   ```

   静态分配需要在别的地方声明对象a、b、resultant，最后这些变量也难逃回收的宿命。

   而对象池就可以解决这个问题，对象池相当于一个初始化好的对象的集合，将对象提供给调用者，在需要的时候拿一个出来用，不用了就还回去。可以有效地减少频繁创建和销毁对象带来的成本，实现对象的缓存和复用。

   ```javascript
   // vectorPool 是已有的对象池
   let v1 = vectorPool.allocate();
   let v2 = vectorPool.allocate();
   let v3 = vectorPool.allocate();
   v1.x = 10;
   v1.y = 5;
   v2.x = -3;
   v2.y = -6;
   addVector(v1, v2, v3);
   console.log([v3.x, v3.y]); // [7, -1]
   vectorPool.free(v1);
   vectorPool.free(v2);
   vectorPool.free(v3);
   // 如果对象有属性引用了其他对象
   // 则这里也需要把这些属性设置为null
   v1 = null;
   v2 = null;
   v3 = null;
   ```

# 参考资料

1. [学习JavaScript的执行上下文、作用域和闭包](https://juejin.cn/post/6975139010918236168)
2. [「硬核JS」你真的了解垃圾回收机制吗](https://juejin.cn/post/6981588276356317214)
3. [简单了解JavaScript垃圾回收机制](https://juejin.cn/post/6844903556265279502)