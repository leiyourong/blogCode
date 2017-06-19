---
title: ECMAScript6 笔记
---
### 1. let & const：声明变量 & 常量
仅在块级作用域有效（弥补JS没有块级作用域的概念），不存在变量提升，会封闭作用域。

### 2. 解构赋值：
按照一定的模式，从对象或数组中提取到对应的值 （默认值为 undefined ,当值为 undefined 不生效）
* 数组解构： 两边必须都是数组，否则报错。
```
let [x, y = 1, z] = [3] // x:3 y:1 z:undefined
```
* 对象、字符串解构： 查找两边同名的属性，再赋值给对应变量  
```
let { x, y } = {x: '1'} // { x, y } = { x: x, y: y } -> x:1 y:undefined
```
* 数值、布尔值解构： 先将右边的值转成对象再解构
* 函数参数解构： 按相同模式传入，正常赋值

### 3. 字符串的扩展：可以识别双字符，例如：'𠮷'（超过\uFFFF）
* Unicode表示法: 采用\u{xxxxx}代替原来的\uxxxx,
* codePointAt(index)/fromCodePoint(code): code编码与char互相转化。index为0时得到双字符(若有)
* for..of: 遍历字符串(类似for...in)      at:根据下标读取字符串(类似charAt)  exp:for(let x of str/arr)
* include/startsWith/endsWith(str,index): 包含str/以str开头/结尾            exp:"xx".include/startsWith/endsWith("xxx",index)
* repeat: 字符串的重复   exp:"xy".repeat('3') //'3'->3->"xyxyxy"
* padStart/padEnd(len.str): 前/后补全字符串  exp:"123".padStart(6,"0")  //"000123"
* 字符串模版:  ``(反引号) 将字符串包在``里,变量用${xxx}定义

4 数值的扩展:
          二进制/八进制前缀: 0b/0o(在严格模式下单独0开头会报错,不能代表八进制)
          Number API: parseInt/Float: 转化成int/float型数值(减少全局空间的函数)     isInteger: 是否整型数值(3与3.0都为整型)  
                                isSafeInteger: 整型数值是否在(-2^53,2^53)之间(超出无法精确表示,not safe)
          Math API: trunc 去除小数部分  sign:判断符号(1/-1/0/-0/NAN)  ...
          **(ES7):指数函数   b**=3 //b=b*b*b

5 数组的扩展: 数组会将空字符串转化成'undefined'
          Array.from(xx): 将伪数组转化成数组(与[].slice.call一致)
          Array.of(x,y,z): 将一组值转化成数组
          [].copyWithin(target,start,end): 将数组自身替换 target 被替换坐标 start/end 开始/结束坐标   exp:[1,2,3].copyWithin(0,1,2)  //[2,2,3]
          [].find/findIndex: 找到满足规则的值/下标(相当于some方法返回值/下标)
          [].fill(item,start,end): 将数组中的项替换成item
          [].keys/values/entries: 数组的键/值/键值对
          数组推导(ES7): 在数组中设定对应的规则,得到新的数组. var a1=[1,2,3,4];var a2=[for(i of a1)if(i>2) i*2]; //a2:[6, 8]

6 函数的扩展: length属性代表函数的没有默认值的入参个数(rest参数也不算)
          设置入参的默认值: 可以与解构赋值结合(有默认值的参数最好放后面),作用域默认是先找函数内部以及默认参数,然后才是函数外
          rest参数: 将若干个参数序列转化成数组values,rest参数后不能有其他正常参数    exp:function xx(...values){}  xx(1,2,3)
          ...运算符: 将数组转化成若干个项(与rest相反),可将字符串和实现Iterator的对象转化成数组Map exp:var b=[1,...[2,3]]; //b:[1,2,3]
          name :函数的名字,构造函数返回"anonymous" 通过bind得到的函数前缀加"bound"
          箭头函数=>: 简化函数体,箭头函数的this指向定义该this的函数,而非引用  exp:var t=(x,y)=>x+2-y;  t(1,2)//1
          尾递归优化: 尾递归就是当返回值是个函数(尾调用)调用自身 优化方法:将中间的变量转化变成函数的参数(减少运行的次数)

7 对象的扩展:
          属性和方法的简写: 属性简写允许只写属性名,默认的属性值为与属性名一致的变量代表的值   exp:var x=1,a={x} // a={x:1}
                                        方法简写允许省略function:      exp: var obj={hello(){...}} 等价于 obj{hello:function(){...}}
          对象定义方式: 可用中括号的形式,并且可以进行运算  exp: var obj={['a'+'bc']:'test'}  //obj.['abc']='test'
          name: 对象(属性)的名字,构造函数返回"anonymous" 通过bind得到的对象(属性)前缀加"bound"
          Object.is: 与"==="类似,添加了+0与-0不等,NaN与自身相等
          Object.assign: 合并多个object,后面会覆盖前面的属性,若为一个对象,则只是得到引用
          对象的可枚举性(enumerable): 当某个属性的enumerable为false时,采用for..in就无法遍历到该属性,还是能正常使用(相当于秘密属性)
                                                           for...in会遍历到继承的属性,Object.keys只会得到自身的属性
          __proto__属性代替: Object.setPrototypeOf/getPrototypeOf 设置/获取当前的__proto__对象

8 Symbol: 原始数据类型(防止属性名的冲突)
          特性: 独一无二的值.属于原始数据类型因此不能添加属性.  exp: let sym = Symbol(xxx); //xxx只是一个描述
                  作为属性时,只能用[]的赋/取值方式或者Object.defineProperty(obj,sym,{value:'xxx'})
          获取symbol属性: 可以通过Object.getOwnPropertySymbols获取所有的symbol属性,for..in/of都无法取到symbol属性
          Symbol.for/keyfor: Symbol.for(key)在创建Symbol之前会在全局变量中查找是否含有相同的key.如果不存在,才新建一个Symbol.
                                          而Symbol()每次都去创建一个Symbol.
                                          Symbol.keyFor(sym):获取Symbol的key

9 Proxy和reflect:
          Proxy:代理,在对象执行某些操作之前拦截,添加自己的逻辑。new Proxy(target,handler),target为目标对象,handler为拦截后具体的
                    操作.添加代理后,必须对代理对象进行操作才能触发拦截(有些操作对原对象操作也会)
                    get: 当获取某个对象的属性时触发(proxy.key)
                    set: 当对象的某个属性进行赋值的时候触发(proxy.key='xxx')
                    apply: 当对象方法被调用/apply/call函数调用时生效(proxy()/proxy.apply(null,xxx))
                    has: 当使用in对象时生效("xxx" in proxy)
                    construct :当new代理对象时生效(new proxy())
                    deleteProperty :当delete对象属性时生效,返回false则阻止删除操作(delete proxy.xxx)
                    emunerate: 当执行for..in的时候被拦截(for xxx in proxy)
                    get/setPrototypeOf: 当执行原型操作时触发(proxy instanceOf xxx)
          Proxy.revocable: 返回一个可取消的Proxy对象(proxy,invoke),其中proxy为代理对象,invoke为一个函数,用来取消proxy对象
          Reflect: 1 将Object上的一些方法明显属于内部的方法放到Reflect上
                       2 Proxy上的方法Reflect一样拥有。
                       3 将一些命令式的操作变成函数行为. exp: delete obj.xxx 变成 Reflect.deleteProperty(obj,xxx)

10 二进制数组: 以字节为单位,主要为了以二进制与底层硬件交互(如显卡),提高效率
          ArrayBuffer 二进制数组      TypedArray 简单视图(单类型)    DataView 复杂视图(可定义多种类型)

11 Set和Map:
          Set: 与Array类似,但内部的成员是不会重复(唯一),并且set的内部不会进行类型的转换,
                 keys/values: 遍历set(得到的结果一样)  size属性: set的元素个数
          WeakSet: 与Set相比有两点差异 1 元素必须都是Object  2 内部的元素,若失去其他引用则自动被回收
          Map: 与Object类似,但key可以是一个对象
          WeakMap: 与Map相比有两点差异 1 key必须是Object  2 内部的元素,若失去其他引用则自动被回收

12 Iterator:  遍历器(array/set/map/字符串<理解成字符数组>默认实现了Iterator接口)
          作用: 1 为各种数据结构提供了统一的访问接口
                   2 数据结构的成员能按某种顺序排列
                   3 提供了新的遍历方式for...of...(可以结合return/break使用)
                   伪数组或者类似数组的对象要实现Iterator接口,只需设置 obj[Symbol.iterator]= Array.prototype[Symbol.iterator]
                   调用 arr[Symbol.iterator]() 可以得到一个遍历器对象,可以使用next()遍历,类似指针

13 Generator函数: 是一种异步编程的解决方案,封装了多个内部状态(yield),返回一个遍历器对象(es7：async await)
          特点: 函数内部状态yield,只有当next()调用到,才会执行
          yield: 相当于函数暂停执行的标记,调用next()继续执行,yield后的表达式不会立即求值,只有当被next调用才求值(函数记忆)
                    next执行传递的参数相当于上一次yield返回的结果,
          Generator.prototype.throw: 遍历器throw异常(参数),可在函数体外throw,函数体内catch到. exp:var f=Generator(); f.throw('a');
          Generator.prototype.return: 结束当前generator中的yield,若有try..finally.则等到finally执行完毕再return
          yield*: 后面跟随一个实现Iterator接口的对象,代表对该对象执行(for xxx of 对象){yield xxx}
                    generator返回的遍历器对象是Generator的实例,可以访问到(Generator.prototype的属性,但是访问不到generator构造方法)
         用途: 可以用于控制函数执行顺序的控制,同步的控制。

14 Promise函数: 异步编程的一种解决方案,与jQuery的promise有点类似
          实例化: 创建一个promise对象 new Promise(function(resolve,reject))
          then/catch/finally/done: 状态为resolve/reject/运行完毕/出现异常触发
          Promise.all: 传个多个promise(若不是则通过Promise.resolve转化),返回一个promise对象,
                              当全部为resolve(返回全部data)/有一个为reject(返回reject的data)执行
          Promise.race: 传入多个promise,当有promise状态发生改变的时候触发
          Promise.resolve/reject: 返回一个状态为resolve/reject的promise对象
                              1 当参数为promise对象时,返回promise该对象
                              2 当参数为非promise对象,并且还有then方法时,返回promise对象,并且执行该对象then方法
                              3 当参数为非上述对象或者不是对象,返回一个promise对象,并且执行该promise对象的then方法,参数为上述对象
                              4 没有参数时,返回一个promise对象,

15 thunkify、Co模块:
          thunk函数: 将多参数的函数转化只接受回调函数的参数,实际上是将参数和回调函数分开调用
          thunkify模块: 传入一个正常的异步函数(有回调 exp:readFile),返回一个thunk函数
                                 用于generator函数的自动调用,前提yield后面必须是thunk函数
          co模块: 用于generator函数的自动调用,前提yield后面必须是thunk函数/Promise对象,接受一个generator函数,返回一个promise对象

16 class: 类 实际上是function的语法糖
          写法: 用class关键字定义一个类,类内部定义的方法实际上是定义在类的原型上,不会变量提升,方法补不可枚举
          new/constructor: new实例化一个类,默认去调用constructor方法
          extends: 1  继承 默认实现父类的constructor方法,要使用this,必须实现父类的constructor(super), (ES5先建立子类的实例,再将父类的方法
                             加入,而ES6是先实例化父类,然后构造子类)
                         2  A extends B :  A.__proto__ === B / A.prototype.__proto__ === B.protototype  
                         3  利用extends可以修改原生的数据类型  
          get/set方法:  定义: get type(){}/set type(value){}    使用: XXX.type/XXX.type='xxx'
          静态方法: ES6没有静态属性,只有静态方法(static). 对于属性,可以通过类定义,不能在内部定义
