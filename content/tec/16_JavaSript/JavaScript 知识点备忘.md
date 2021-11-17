记录一些自己不熟悉、不常用但是可能有用的知识点，或者是 JavaScript 中的一些坑（占多数）。

# 数据类型

---

## 种类

基本类型：

- number
- string
- boolean
- undefined
- null
- object，又有三种子类
   - 狭义的 object
   - array
   - function
- Symbol（ES6）
- BigInt（ES 2020）

检查对象的三个手段：

- typeof
- instanceof
- Object.prototype.toString

其中，由于历史原因，null 返回 object。

```javascript
typeof null // "object"
```

对于 object，typeof 不能细分类型，而 instanceof 可以。

```javascript
typeof window // "object"
typeof {} // "object"
typeof [] // "object"

var o = {};
var a = [];

o instanceof Array // false
a instanceof Array // true
```

## 布尔值

一下六个值被转换为 false，其他的值都视为 true，不过还是应避免隐式转换。

- undefined
- null
- false
- 0
- NaN
- 空字符串： `""`  `''` 

需要注意的是，空列表 `[]` 和空对象 `{}` 并不是 false 的。

## 数值

所有的数值底层都是浮点数， `1 === 1.0` 的判断结果为 true。

浮点数运算注意浮点误差。

JS 使用 IEEE 754 标准与 64 位存储浮点数，Number 对象得 MAX_VALUE 和 MIN_VALUE 是 JS 可以表现得最大值和最小值。

```javascript
Number.MAX_VALUE // 1.7976931348623157e+308
Number.MIN_VALUE // 5e-324
```

特殊类型的数字：

- 正零、负零
- NaN
- Infinity、 -Infinity

 NaN 属于 Number 类型。

```javascript
typeof NaN // 'number'
```

一些转换方法：

- Number.parseInt
- Number.parseFloat

一些检测方法

- Number.isNaN
- Number.isFinite
- Number.isSafeInteger

parse 函数的trim、截断、异常情况：

```javascript
parseInt('   81') // 81
parseInt('8a') // 8
parseInt('12**') // 12
parseInt('12.34') // 12
parseInt('15e2') // 15
parseInt('15px') // 15
```

parse 会将非字符串的第一个参数先转换为字符串。对于那些会自动转为科学计数法的数字，parseInt 会将科学计数法的表示方法视为字符串，因此导致一些奇怪的结果。故而我们应避免向 parse 函数传入 number。

```javascript
parseInt(1000000000000000000000.5) // 1
// 等同于
parseInt('1e+21') // 1

parseInt(0.0000008) // 8
// 等同于
parseInt('8e-7') // 8
```

## 函数

在使用变量赋值的方法声明函数时，仍然可以为函数赋予名字，但这个名字只在函数体内部有效：

```javascript
let print = function x(){
  console.log(typeof x);
};

x
// ReferenceError: x is not defined

print()
// function
```

上面代码在函数表达式中，加入了函数名 x。这个 x 只在函数体内部可用，指代函数表达式本身，其他地方都不可用。这种写法的用处有两个：

- 一是可以在函数体内部调用自身，用于递归
- 二是方便除错（调式工具显示函数调用栈时，将显示函数名，而不再显示这里是一个匿名函数）

此外，eval 函数有几个严重的问题：

- 性能低下
- 可以修改外部变量，引入安全风险

## 数组

数组的 length 是可以写入的，如果人为设置一个小于当前成员数的值，该数组的成员会自动减少到 length 设置的值。

```javascript
var arr = [ 'a', 'b', 'c' ];
arr.length // 3

arr.length = 2;
arr // ["a", "b"]
```

由于数组本质上是一种对象，所以可以为数组添加属性，但是这不影响 length 属性的值。

```javascript
var a = [];

a['p'] = 'abc';
a.length // 0

a[2.1] = 'abc';
a.length // 0
```

添加键名会导致 `for ... in` 读到这些键名，所以推荐使用 ES6 `for ... of` 。

```javascript
var a = [1, 2, 3];
a.foo = true;

for (var key in a) {
  console.log(key);
}
// 0
// 1
// 2
// foo
```

检查某个键名是否存在的运算符 in，适用于对象，也适用于数组。

```javascript
var arr = [ 'a', 'b', 'c' ];
2 in arr  // true，数字 2 自动转换为了字符串 '2'
'2' in arr // true
4 in arr // false
```

使用 delete 命令删除一个数组成员，会形成空位，并且不会影响 length 属性。数组的某个位置是空位，与某个位置是 undefined，是不一样的。如果是空位，使用数组的 forEach 方法、 `for...in` 结构、以及 `Object.keys` 方法进行遍历，空位都会被跳过。如果某个位置是 undefined，遍历的时候就不会被跳过。JS 对于空位的处理很不同意，所以应避免出现空位。

```javascript
var a = [1, 2, 3];
delete a[1];

a[1] 						// undefined
a								// [1, empty, 3]
a.length 				// 3
Object.keys(a)  // ["0", "2"]

a[1] = undefined
Object.keys(a)  // ["0", "1", "2"]
```

## 数据类型转换

强制转换主要指使用 Number()、String() 和 Boolean() 三个函数，手动将各种类型的值，分别转换成数字、字符串或者布尔值。

Number 相比 parse 系列函数更为严格，不满足要求字符串不会截断，而是返回 NaN。打他们都会过滤字符串的前缀、后缀空白符与转义字符

```javascript
// 数值：转换后还是原来的值
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

parseInt('\t\v\r12.34\n') // 12
Number('\t\v\r12.34\n') // 12.34
```

Number 也可以将对象转换为数字，但这是奇技淫巧，应避免使用。

Boolean 的转换规则与布尔值的六种隐式转换规则相同。


# 运算符

利用短路机制简化条件表达式。

```javascript
if (i) {
  doSomething();
}

// 等价于

i && doSomething();
```

所有的位运算都只对整数有效。二进制否运算遇到小数时，也会将小数部分舍去，只保留整数部分。所以，对一个小数连续进行两次二进制否运算，能达到取整效果。使用二进制否运算取整，是所有取整方法中最快的一种。

```javascript
~~2.9 // 2
~~47.11 // 47
~~1.9999 // 1
~~3 // 3
```

逗号运算符用于对两个表达式求值，并返回后一个表达式的值。

```javascript
'a', 'b' // "b"

var x = 0;
var y = (x++, 10);
x // 1
y // 10
```

# 标准库

## Object

Object.prototype.toString 方法返回对象的类型字符串，因此可以用来判断一个值的类型。

由于实例对象可能会自定义 toString 方法，覆盖掉 Object.prototype.toString 方法，所以为了得到类型字符串，最好直接使用 Object.prototype.toString 方法。通过函数的 call 方法，可以在任意值上调用这个方法，帮助我们判断这个值的类型。    

```javascript
Object.prototype.toString.call(value)
```

不同数据类型的 Object.prototype.toString 方法返回值如下：

- 数值：返回 [object Number]；
- 字符串：返回 [object String]；
- 布尔值：返回 [object Boolean]；
- undefined：返回 [object Undefined]；
- null：返回 [object Null]；
- 数组：返回 [object Array]；
- arguments 对象：返回 [object Arguments]；
- 函数：返回 [object Function]；
- Error 对象：返回 [object Error]；
- Date 对象：返回 [object Date]；
- RegExp 对象：返回 [object RegExp]；
- 其他对象：返回 [object Object]

根据上述特性，可以实现比 typeof 更准确的类型判断。

对象的简写：

```javascript
let user = {
  name: 'test'
};

let foo = {
  bar: 'baz'
};

console.log(user, foo)
// {name: "test"} {bar: "baz"}
console.log({user, foo})
// {user: {name: "test"}, foo: {bar: "baz"}}
```

### Object.entries() / keys() / values()

```javascript
let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```

Object.entries() 还有反向操作 Object.fromEntries()：

```javascript
Object.fromEntries([
  ['foo', 'bar'],
  ['baz', 42]
])
// { foo: "bar", baz: 42 }
```

### 解构赋值

可以用于快速地覆盖对象属性：

```javascript
let aWithOverrides = { ...a, x: 1, y: 2 };
// 等同于
let aWithOverrides = { ...a, ...{ x: 1, y: 2 } };
// 等同于
let x = 1, y = 2, aWithOverrides = { ...a, x, y };
// 等同于
let aWithOverrides = Object.assign({}, a, { x: 1, y: 2 });
```

### 链式判断运算符（ES 2020）

链式判断运算符指的是 `?.` ，可以避免连续的 null 或 undefined 检查，类似 Java 中的 Optional 和 Kotlin 中的 `?` :

```javascript
const firstName = message?.body?.user?.firstName || 'default';
const fooValue = myForm.querySelector('input[name=foo]')?.value
```

除了访问对象，还可以用于函数调用：

```javascript
if (myForm.checkValidity?.() === false) {
  // 表单校验失败
  return;
}
```

一些不能使用链式判断运算符的场景：

```javascript
// 构造函数
new a?.()
new a?.b()

// 链判断运算符的右侧有模板字符串
a?.`{b}`
a?.b`{c}`

// 链判断运算符的左侧是 super
super?.()
super?.foo

// 链运算符用于赋值运算符左侧
a?.b = c
```

### Null 判断运算符（ES 2020）

通常，为变量设置运算符使用 `||` 运算符，但是这并不适用于变量为 false 或 0 这种隐式转换的情况。

```javascript
const headerText = response.settings.headerText || 'Hello, world!';
const animationDuration = response.settings.animationDuration || 300;
const showSplashScreen = response.settings.showSplashScreen || true;
```

ES 2020 引入 null 判断运算符 `??` ，更符合这种“非 null 非 undefined 则赋值的语义”

```javascript
const headerText = response.settings.headerText ?? 'Hello, world!';
const animationDuration = response.settings.animationDuration ?? 300;
const showSplashScreen = response.settings.showSplashScreen ?? true;
```

## ES6 新增方法

- Object.is(a, b)：与 `===` 行为基本一致，区别在于对 +0 / -0 、NaN 的判断；
- Object.assign(target, ...source)：source 属性覆盖 target 属性，浅拷贝，而且对于 getter 无效；

## Array

- push / pop / shift / unshift
- concate
- reverse
- slice / splice
- forEach / map / filter
- some / every
- reduce / reduceRight
- indexOf / lastIndexOf
- find / findIndex

ES2019 规定了 Array.prototype.sort() 的实现应该是稳定的，即具有[排序稳定性](https://es6.ruanyifeng.com/#docs/array#Array-prototype-sort-%E7%9A%84%E6%8E%92%E5%BA%8F%E7%A8%B3%E5%AE%9A%E6%80%A7)。

## WeakSet

WeakSet 与 Set 的两个区别：

首先，WeakSet 的成员只能是对象，而不能是其他类型的值。

其次，WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
这是因为垃圾回收机制依赖引用计数，如果一个值的引用次数不为 0，垃圾回收机制就不会释放这块内存。结束使用该值之后，有时会忘记取消引用，导致内存无法释放，进而可能会引发内存泄漏。WeakSet 里面的引用，都不计入垃圾回收机制，所以就不存在这个问题。因此，WeakSet 适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在 WeakSet 里面的引用就会自动消失。

## Map

ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。

## Number

- Number.prototype.toString()
- Number.prototype.toFixed()
- Number.prototype.toExponential()
- Number.prototype.toPrecision()
- Number.prototype.toLocaleString()

## String

- match / matchAll ( ES 2020 ) / search / replace
- split
- includes / startsWith / endsWith

### 模板字符串

```javascript
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

## Date

日期相减得到的是毫秒级时间差，相加则是拼接的字符串。

```javascript
var d1 = new Date(2000, 2, 1);
var d2 = new Date(2000, 3, 1);

d2 - d1
// 2678400000
d2 + d1
// "Sat Apr 01 2000 00:00:00 GMT+0800 (CST)Wed Mar 01 2000 00:00:00 GMT+0800 (CST)"
```

静态方法

- Date.now() -> ms
- Date.parse(str) -> ms

实例方法

- date.valueOf() -> ms
- date.setXXX
- date.getXXX

## JSON

- JSON.stringify
- JSON.parse

JSON.stringify 对象可以传递第二个参数用于修改转码阶段的内容：

```javascript
function f(key, value) {
  if (typeof value === "number") {
    value = 2 * value;
  }
  return value;
}

JSON.stringify({ a: 1, b: 2 }, f)
// '{"a": 2,"b": 4}'
```

JSON.stringify 还可以接受第三个参数，用于增加返回的 JSON 字符串的可读性。如果是数字，表示每个属性前面添加的空格（最多不超过10个）；如果是字符串（不超过10个字符），则该字符串会添加在每行前面：

```javascript
JSON.stringify({ p1: 1, p2: 2 }, null, 2);
/*
"{
  "p1": 1,
  "p2": 2
}"
*/

JSON.stringify({ p1:1, p2:2 }, null, '|-');
/*
"{
|-"p1": 1,
|-"p2": 2
}"
*/
```

object 的 toJSON() 方法会直接被 JSON.stringify 调用，而忽略原始的 stringify 行为：

```javascript
var user = {
  firstName: '三',
  lastName: '张',
  gener: 'male',

  get fullName(){
    return this.lastName + this.firstName;
  }
};

JSON.stringify(user)
// "{"firstName":"三","lastName":"张","gener":"male","fullName":"张三"}"

user.toJSON = function(){
	return this.lastName + this.firstName;
}
JSON.stringify(user)
// "{"name":"张三"}"
```

# 面向对象

## new

在函数内部可以检查当前函数是否通过 new 调用：

```javascript
function f() {
  console.log(new.target === f);
}

f() // false
new f() // true
```

# 异步操作

## Promise

一般来说，不要在 then 方法里面定义 Reject 状态的回调函数（即 then 的第二个参数），总是使用 catch 方法。

```javascript
// bad
promise
  .then(function(data) {
    // success
  }, function(err) {
    // error
  });

// good
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

跟传统的 try/catch 代码块不同的是，如果没有使用 catch 方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码，即不会有任何反应。

### Promise.all()

```javascript
const p = Promise.all([p1, p2, p3]);
```

p 的状态由p1、p2、p3 决定，分成两种情况：

1. 只有 p1、p2、p3 的状态都变成 fulfilled，p 的状态才会变成 fulfilled，此时 p1、p2、p3 的返回值组成一个数组，传递给 p 的回调函数。
1. 只要 p1、p2、p3 之中有一个被 rejected，p 的状态就变成 rejected，此时第一个被 reject 的实例的返回值，会传递给 p 的回调函数。

### Promise.race()

### Promise.any()

### Promise.allSettled()

### Promise.resolve()

有时需要将现有对象转为 Promise 对象，Promise.resolve() 方法就起到这个作用。

```javascript
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

Promise.resolve() 方法允许调用时不带参数，直接返回一个 resolved 状态的 Promise 对象。

```javascript
const p = Promise.resolve();
p.then(function () {
  // ...
});
```

# Ref

- [JavaScript 教程 - 网道](https://wangdoc.com/javascript/index.html)
- [ECMAScript 6 入门 - 阮一峰](https://es6.ruanyifeng.com/#docs/) 
