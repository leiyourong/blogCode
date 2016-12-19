---
title: JavaScript 笔记
---

## 判断数据类型:

* `Object.prototype.toString.call(obj)`： 判断 obj 的类型 `1 -> [Object Number]`
* `typeof obj`： 判断 obj 的类型，无法判断复杂的数据类型(数组/函数等) `1 -> "number"`
* `a instanceof A`： 判断a是否是A的实例（判断A的原型是否出现在a的原型链上）

## 栈和堆:

1. 栈计算的速度快，而堆的内存大
2. 对于简单的数据类型，直接放在栈中
3. 而对于复杂的数据类型，引用变量放在栈中，而引用值放在堆中，中间用指针连接

## prototype/prototype Chain/\_\_proto\_\_:原型/原型链/内置原型对象(画内存图)

* 每一个函数对象都有一个显示的 prototype 属性
* 每个对象都有一个名为 \_\_proto\_\_ 的内部隐藏属性，原型链正是基于 \_\_proto\_\_ （IE不支持）才得以形成

>Function.prototype === function.\_\_proto\_\_

```
var Fun1 = function() {}
var Fun2 = function() {}
Fun1.prototype.name = 'Fun1'
Fun2.prototype.name = 'Fun2'
var test = new Fun1()
Fun1.prototype = new Fun2 // 重新赋值了之后，函数的原型指向了一块新的内存，原来实例化的内容指向不变
console.log(test.name) // 'Fun1'
var test = new Fun1()
console.log(test.name) // 'Fun2'
```

## constructor:

* 每个 prototype 对象包含一个 constructor 属性，该属性指向了 prototype 的函数体
* 调用 xxx.constructor 时，解析器会沿着 xxx 的原型链一直查找，直到找到 constructor 属性为止

>Function.prototype.constructor === Function

## 函数对象的详细创建过程：`var dog = new Animal("旺财")`

1. 判断 Animal 是否是个 Object，若是: 将 `dog.__proto__ = Animal.prototype`，进入2.
   否则 `dog.__proto__ = Object.prototype`
2. 将 `args("旺财")` 传入 Animal 的函数体中，返回函数体的结果值。若没有返回值，则返回 undefined
   1. 找到 `Animal.prototype` 中的 constructor 属性，并将参数传入该 constructor 指向的函数体运行，constructor 指向了 Animal 函数体，然后创建当前的上下文，运行返回值，最后销毁上下文  `Animal.prototype.constructor === Animal`
   2. 若当前的返回值是 Object 对象，则返回该对象具体的值，否则返回该函数本身;
      函数的 prototype 是一个 Object 对象，所以默认情况下 `Animal.prototype.__proto__ === Object.prototype`
      Object 对象也是一个函数对象 `Object === new Function()`

## function的三种定义方式:
1. 函数声明 `function a(){}`  在初始化时会被提到顶部去执行(函数声明提升)
2. 函数表达式  `var a = function(){}`  只是定义部分会被提到顶部执行
3. `var a = new Function('xx'，'func')` 运行效率低（与eval一样的道理）

```
name = 'oldName'
var Fun = function (){
  this.changeName = function() {
    name = 'newName'
  }
  this.name = '222'
}
function Fun() {
  this.name = '111'
}
console.log(new Fun().name) // 222
console.log(name) // 'oldName'
new Fun().changeName()
console.log(name) // 'newName'
```

## arguments:  
1. arguments 对象的长度是由实参决定的 `arguments.length`（实参） `arguments.callee.length`（形参）
2. arguments 是伪数组，保存参数的信息    

## eval存在问题:
1. 由于正常的js代码采用预编译去解析，而调用eval代码的时候需要重新加载解析器
2. 安全问题:容易产生XSS安全问题
3. 会有可能会改变this的指向。

## apply、call、bind
  `A.apply(B, [arg1, arg2])    A.call(B, arg1, arg2)`
* 如果 A 为 Object，则的作用为将 A 的全部赋给B（若有一样的参数或方法，A覆盖B）
* 如果 A 为 Function，则的作用为用 A 方法代替B去执行（只是调用方法，参数还是不覆盖）<执行A的方法，this还是指向B>

bind：绑定对象，返回的是改变上下文this后的函数，并且只替换当前存在的参数

## caller与callee
* Function.caller 返回方法的调用者，若为顶层返回null
* argument.callee 返回正在被调用的方法，length属性返回的是参数的个数(可实现递归)


## 作用域与作用域链（Scope/Scope Chain）：
```
var w = 1
function outer() {
  var i = 10
  console.log(w + '-' + i + '-' + j) // 1-10-undefined
  function inner() {
    j = 100
    console.log(w + '-' + i + '-' + j) // 1-10-100
  }
  inner()
  console.log(w + '-' + i + '-' + j) // 1-10-100
}
console.log(w + '-' + i + '-' + j) // 1-underfined-undefined
outer()
console.log(w + '-' + i + '-' + j) // 1-underfined-100
var i, j
```
1. javaScript解析器在一开始执行时，会把变量的声明部分提到函数的顶部执行（声明），并且赋初值 undefined
2. 完成各个变量的赋值 `w = 1`
3. 函数执行时会根据变量的作用域链，先查找本函数内的局部变量，**如果找不到，然后一层层往外查找，直到找到最外层**
  1. 一开始，函数的活动对象为window对象，此时链表的头部为window对象.此时链表为window，
     此时 window 中 `w = 1, i = undefined , j = undefined`
  2. 执行到 outer，作用于变成 outer 函数. 此时链表为 outer->window，
     此时 outer 中 `w = undefined, i = 10 , j = undefined`，window 不变
  3. 执行到 inner，作用于变成 inner 函数. 此时链表为 inner->outer->window，
     此时 inner 中 `w = undefined, i = undefined , j = undefined`， outer 不变，
     window 变成 `w = 1, i = undefined , j = 100`
  4. 销毁活动对象 inner，销毁活动对象 outer.

## 闭包:
除自身以外的函数，访问到该函数内部的属性和方法（使用返回值）
###### 特点:
1. 外部函数能够访问到函数内部的属性和方法
2. 该方法不会被 GC 回收
```
function outer() {
  function inner() {}
  return inner // 返回给外界调用
}
```
## this的指向问题:
指向该this所属的方法的调用者(setTimeout和setInterval会把this指向window)


## toString、valueOf 方法(一般方法最后会隐式的调用这个方法)
* toString：将其他类型的参数转化成字符串型   
* valueOf： 返回目标对象的原始值
* toString(n)：将数字型转化成n进制
```
{} + {} // "[object Object][object Object]"
{} + '' // 0
'' + {} // "[object Object]"
0 + {}  // "0[object Object]"
{} + 0  // 0
```
