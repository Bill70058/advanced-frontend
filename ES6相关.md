## ES6相关

### 什么是ECMA

**ECMA国际，是一个专门为技术制定标准的组织**

在1997年6月， 该公司发布了一个标准 - **ECMA-262**

ECMA-262作为一个标准，它为通用编程语言（ECMAScript）定义了相关的规范。
 所以，ECMA-262是标准的名称，它的内容描述了**ECMAScript语言规范（ECMAScript Language Specification）**

ES6，又叫ES2015，是2015年六月ESMA机构定下的JS标准，但是每年都做个迭代版本号会很累赘，所以当有重大内容改革的时候才会变更版本号，其中ES6新特性最经典，所以最出名，此后每年六月都会开会定标准，六月前提交的建议都会采用到当年版本迭代里。

### 什么是ESnext

使用 TypeScript 开发项目的过程中，经常需要使用 `tsc` 命令将 TS 代码编译成特定版本的 ECMAScript，在`tsconfig.json`配置文件中有一个 `target` 字段，决定编译后输出的 ECMAScript 版本，默认输出 `ES3`。`target`字段有多个值可以选择，有一个名为 `ESNext` 的值。esnext 是一个 JavaScript 库，可以将 ES 草案规范语法转成今天的 JavaScript 语法

### const常量

- 通常以大写字母作为变量名

```js
const LIMIT = 19
```

- 不允许重复声明，必须赋初值

```js
// var,let 可以为变量重新赋值，并且不一定要赋初值
var arg1 = '111'
arg1 = '222'

// 如果不想变量重新赋值，可以使用defineProperty的writeable属性
Object.defineProperty(window, 'arg2', {
  value: '123',
  writeable: false // 设置为false则只读
})

// ES6
const arg3 = '333'
arg3 = '123' // TypeError: Assignment to constant variable
```

- 块级作用域

```js
if (true) {
  var arg1 = '123'
}
console.log('arg1', arg1) // 123

// ES6
if (true) {
  const arg2 = '222'
}
console.log('arg2', arg2) // Uncaught ReferenceError: arg2 is not defined
```

- 变量提升，var可以变量提升，const和let则没有

```js
console.log(arg1)
var arg1 = '123'

// 相当于
var arg1
console.log(arg1) // 声明但未赋值，所以是undefined
arg1 = '123'

// 无变量提升
console.log(arg2) // Uncaught ReferenceError: arg2 is not defined
const arg2 = '123'

// 全局作用域，var会被提升到window里，let和const不会
var arg1 = '123'
console.log(window.arg1) // 123

const arg1 = '123'
console.log(window.arg1) // undefined
```

- 暂时性死区(Temporal Dead Zone)

一个概念，是对于某些遇到在区块作用域绑定早于声明语句时的状况时，所使用的专用术语。与变量提升和块级作用域相关，简单来说就是声明前无法使用该变量。

TDZ最一开始是为了const所设计的，但后来的对let的设计也是一致的。

```js
    if(true) {
        console.log(arg1);
        const arg1 = '云隐';
    }
```

- 引用型

虽然对于const，初始化之后的常量不能再赋值，但是如果该常量是引用型(对象或者数组)，则可以修改其内部属性的值

```js
const obj = {
  a: '123',
  b: '456'
}
obj.a = 'aaa'

const arr = ['123','456']
arr[0] = 'aaa'

// 使用Object.freeze()可以冻结该对象，但是只能冻结第一层结构
const obj2 = {
  a: 'aaa',
  b: 'bbb',
  c: 'ccc'
}
Object.freeze(obj2)
obj2.a = '123'
console.log(obj2.a) // aaa，无报错无修改

// 如果想要冻结该对象底下所有层级的对象则需要递归
function deepFreeze(obj) {
        // 2. 确定主执行步骤
        Object.freeze(obj);
        // 3. 逐级深入
        (Object.keys(obj) || []).forEach(key => { // for in - hasOwnProperty
            let innerObj = obj[key];
            
            if (typeof innerObj === 'object') {
                // 1. 递归模式确定
                deepFreeze(innerObj);
            }
        })
    }
```

### 解构

**解构赋值**语法是一种 Javascript 表达式。可以将数组中的值或对象的属性取出，赋值给其他变量

```js
let obj = {
  a: 'aaa',
  b: 'bbb'
}
let {a, b} = obj
console.log(a) // aaa
```

### 解构的使用场景

#### 形参结构

```js
    const sum = arr => {
        let res = 0;
        arr.forEach(each => {
            res += each;
        })
    }
// 解构，从each中取出a、b、c这三个参数出来相加，当然这种方式需要已知参数有哪些
    const sum = ([a, b, c]) => {
        return a + b + c;
    };
```

#### 结合初始值

在结构同时可以为其参数赋予初始值

```js
    const course = ({ a, b, c = 'ccc' }) => {
        // …… 
    }

    course({
        a: 'aaa',
        b: 'bbb'
    })
```

#### 返回值

比较经常用到的，将返回值解构出变量，不用通过链式写一堆，也用于按需导入

```js
    const getCourse = () => {
        return {
            a: '',
            b: ''
        }
    }

    const { a, b } = getCourse();
```

#### 变量交换

```js
// 普通写法，需要创建一个额外的变量作为中间人
let a = 1, b = 2
let temp
temp = a
a = b
b = temp

// 解构写法
let a = 1, b = 2
[b, a] = [a, b]
```

#### json处理

与上面的返回值等用法和目的差不多

```js
const json = '{"a": "aaa", "b": "bbb"}';

const obj = JSON.parse(json);

const {
    a,
    b
} = JSON.parse(json);
```

### 箭头函数

```js
    // 传统函数声明
    function test(a, b) {
        return a + b;
    }
		// 函数表达式
    const test2 = function(a, b) {
        return a + b;
    }
    // 匿名函数
    (function() {
      // ...
    })()
    
    // 箭头函数
    const test3 = () => {
      console.log('test3')
    }
    // 当返回值只有一个时可以省略花括号和return
    const test3 = (a,b) => a+b
```

#### 上下文

```js
    // ES6
    const obj = {
        a: 'aaa',
        b: 'bbb',
        arr: ['123', '456'],
        getProp: function() {
            console.log('a is:', this.a);
            return this.a;
        },
        getProp2: () => {
            console.log('b is:', this.b);
            return this.b;
        }
    }
    
    obj.getProp() // a is aaa
		obj.getProp2() // b is undefined
```

因为箭头函数不会形成独立上下文，内部this指向了window，所以找不到obj内部的属性

#### 类操作

- 箭头函数无法成为完整构造类

```js
const Obj = (a, b) => {
  this.a = a;
  this.b = b;
}
const o1 = new Obj('aaa', 'bbb')
console.log(o1) // Obj is not a constructor
```

- 箭头函数无法构造原型方法

```js
// Cannot set properties of undefined (setting 'learn')
Obj.prototype.learn = () => {
        console.log(this.a, this.b);
    }
```

- 箭头函数的参数特性 - 无法使用arguments

```js
const test = arg => {
        console.log(arguments);
    }
test() // Uncaught ReferenceError: arguments is not defined
```

### Class

传统对象是通过构建一个函数，在函数原型链上挂载方法和属性达到创建实例方法的效果，比如Vue框架

```js
function Vue(a,b){
  this.a = a;
  this.b = b;
}
Vue.property.getA = function() {
  return `this a is :${this.a}`
}
const vue = new Vue('aa', 'bb')
vue.getA() // aa
```

ES6则可以通过class助力面向对象

```js
class Vue {
  // 初始化会执行的代码
  constructor(a, b) {
    this.a = a;
    this.b = b;
  }
  // 拓展方法
  getA() {
    return `this a is :${this.a}` 
  }
}
const vue = new Vue('aa', 'bb')
vue.getA() // aa
```

#### 属性定义 构造器 & 顶层定义 两种定义方式

在class中，属性还有个get、set属性

```js
class Vue {
  constructor(a) {
    this.a = a
  }
  get a() {
    return this.a
  }
  set a(val) {
    this.a = val
  }
}
```

巧妙的运用这两个属性可以做到 -- 创建只读的属性

没有暴露set方法，则不允许修改a属性

```js
class Vue {
  constructor(a) {
    this.a = a
  }
  get a() {
    return this.a
  }
}
```

- 创建一个私有变量

```js
class Vue {
  constructor(a, b) {
    // 在命名上，对私有属性前加下划线更好区分
    this._a = a
    
    // 在构造器内部定义一个局部变量
    let _temp = 'es6'
    // 内部通过闭包的方式去暴露该变量
    this.getTemp = () => {
      return _temp
    }
  }
}
```

es新版本有个``#``号，通过#号声明也可以将该变量声明为私有变量

```js
class Vue {
  #temp = 'es6'
  constructor(a, b) {
    this._a = a;
    this._b = b;
  }
	get temp() {
    return this.#temp
  }
	set temp(val) {
    if (val) {
      this.#temp = val
    }
  }
}
```

- 适配器模式创建私有变量

适配器模式是设计模式的一种，简单来说就是核心代码封装一层，暴露一些接口出去给第三方用，但是可以控制哪些属性是不能被修改的

```js
class Vue {
  constructor(core) {
    this._main = core
    this._name = 'my-utils'
    this._id = 'id'
  }
  get name() {
    // 这里暴露出去的属性中，fullName是可以通过外部传参的值改变的，但是name属性一直是私有的不能修改的
    return {
      fullName: this._main.fullName,
      name: this._name
    }
  }
}
```

#### 静态方法 -- 直接挂载在类上面无需实例化获取

```js
// ES5写法
function Vue() {
  // ...
}
Vue.mounted = function() {
  // ...
}

// ES6
class Vue {
  constructor() {
    // ...
  }
  static mounted() {
    // ...
  }
}

// 调用
Vue.mounted()
```

#### 继承

```js
// es5写法
function Vue() {
  // ...
}
Vue.mounted = function() {}
Vue.prototype.send = function() {
  // ...
}
function Child() {
  Vue.call(this)
  this.run = function() {
    // ...
  }
}
Child.prototype = Vue.prototype

// es6
    class Vue {
        constructor() {
            //……
        }
        static mounted() {}
        send() {}    
    }
    // => 工厂模式：父类写了通用方法，子类实现其他业务功能
    class Child extends Course {
        constructor() {
            super('es6')
        }
        run() {}
    }
```

#### 面试题

- class的类型是

```js
console.log(typeof Vue) // function
```

- class的prototype是

class的prototype会多一个class 类名和construct属性，而function的只有construct属性

```js
console.log(Vue.prototype) // 有区分，本质上是一样的
```

- class&函数对象属性

两者一样

```js
console.log(Vue.hasOwnProperty('a')) // true
```

