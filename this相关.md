## this相关

### 上下文确定作用域

#### 作用域链

```js
    let a = 'global';
    console.log(a);

    function course() {
        let b = 'bbb';
        console.log(b);

        session();
        function session() {
            let c = 'this';
            console.log(c);

            teacher();
            function teacher() {
                let d = 'yy';
                console.log(d);

                console.log('test1', b);
            }
        }
    }
    console.log('test2', b);
    course();

    if(true) {
        let e = 111;
        console.log(e);
    }
    console.log('test3', e)
```

- 对于作用域链直接通过创建态来定位作用域链
- 手动取消全局，使用块级作用域

#### this，上下文context

- this是在执行时动态读取上下文决定的，而不是创建时

#### 函数直接调用

this指向的是window => 函数表达式、匿名函数、嵌套函数

```js
    function foo() {
        console.log('函数内部this', this);
    }

    foo();
```

#### 隐式绑定

this的指向是调用堆栈的上一级 => 对象、数组等引用关系逻辑，从运行的上下文判断this

```js
function fn() {
    console.log('隐式绑定', this.a);
}
const obj = {
    a: 1,
    fn
}

obj.fn = fn;
obj.fn();
```
#### 隐式绑定面试题

```js
 const foo = {
        bar: 10,
        fn: function() {
            console.log(this.bar);
            console.log(this);
        }
    }
    // 取出
    let fn1 = foo.fn;
    // 执行
    fn1();  // this.bar => undefined，this => Window
```

取出后全局执行，this指向Window

##### 如何隐式改变this指向

直接使用上下文

```js
    const o1 = {
        text: 'o1',
        fn: function() {
            return this.text;
        }
    }
```

回调上一级的方法，但是这种this指向没有改变到，只是输出的值变了不是“o2”

```js
   const o2 = {
        text: 'o2',
        fn: function() {
            return o1.fn();
        }
    }
```

通过内部构造，返回一个新的方法

```js
    const o3 = {
        text: 'o3',
        fn: function() {
            let fn = o1.fn;
            return fn();
        }
    }
```

总结：隐式改变this指向有两种方法

1. 在执行函数时，函数被上一级调用，上下文指向上一级
2. 直接变成公共函数，指向window

##### 如果要o2输出“o2”需要做什么修改

1. 显示绑定，通过bind、call、apply改变this指向
2. 隐式将方法挂载在o2上

```js
  const o1 = {
        text: 'o1',
        fn: function() {
            return this.text;
        }
    }

    const o2 = {
        text: 'o2',
        fn: o1.fn
    }

```

### 显示绑定（bind | apply | call）

使用方法

```js
function foo() {
    console.log('函数内部this', this);
}
foo();

// 使用
foo.call({a: 1});
foo.apply({a: 1});

const bindFoo = foo.bind({a: 1});
bindFoo();
```
#### call、apply、bind的区别

1. 第一个参数都是this指向的对象
2. call的参数是依次传入，apply和bind是以数组形式传入
3. call和apply都是立即执行，bind是返回一个新的函数

### new

this指向的是new之后得到的实例

```js
    class Course {
        constructor(name) {
            this.name = name;
            console.log('构造函数中的this:', this);
        }

        test() {
            console.log('类方法中的this:', this);
        }
    }

    const course = new Course('this');
		// this指向的是course
    course.test();
```

#### 面试题：类中的异步方法，this有区别吗

```js
    class Course {
        constructor(name) {
            this.name = name;
            console.log('构造函数中的this:', this);
        }

        test() {
            console.log('类方法中的this:', this);
        }
        asyncTest() {
            console.log('异步方法外:', this);
            setTimeout(function() {
                console.log('异步方法内:', this);
            }, 100)
        }
    }

    const course = new Course('this');
    course.test();
    course.asyncTest();
```

- 执行setTimeout时，匿名方法执行时，效果和全局执行函数效果相同
- 如何解决：使用箭头函数，this指向是调用方

### bind的原理/手写bind

1. 需求：手写bind，由bind的使用方式可以知道，bind挂载在Function.prototype上

2. bind是什么，第一项参数是新this，第二项~最后一项是函数传参，数组的方式传参

   a. 返回一个函数

   b. 返回原函数执行结果

   c. 参数不变

```js
function sum(a,b,c) {
  console.log('sum', this)
  return a+b+c
}
// 1.
Function.prototype.newBind = function() {
  // 2.
  const _this = this
  const args = Array.prototype.slice.call(arguments)
  // 取出第一个参数(this)，剩下的是数组形式的参数
  const newThis = args.shift()
  // 返回一个函数，函数内容是返回原函数执行结果
  return function() {
    return _this.apply(newThis, args)
  }
}
```

### 手写apply

- 和bind一样，由使用方式得知，apply是挂载在Function.prototype上
- 与bind不同的是，直接返回执行结果

```js
Function.prototype.newApply = function(context) {
  // 边缘检测
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  // 参数检测：如果没有传入参数则指向全局
  context = context || window
  // 挂载执行函数，this就是自身函数，挂载后可以下面直接执行自身，用于返回函数执行后的值
  context.fn = this
  // 执行执行函数，如果有后续参数则将后续参数传入当前函数，如果没有则直接执行
  let result = arguments[1] ? context.fn(...arguments[1]) : context.fn()
  // 销毁临时挂载
  delete context.fn
  return result
}
```

### 如何突破作用域的束缚 -- 闭包：一个函数和它周围状态的引用捆绑在一起的组合

##### 函数作为返回值的场景

```js
    function mail() {
        let content = '信';
        return function() {
            console.log(content);
        }
    }
    const envelop = mail();
    envelop();
```

- 函数外部获取到了函数作用域内的变量值

##### 函数作为参数的时候

```js
    // 单一职责
    let content;
    // 通用存储
    function envelop(fn) {
        content = 1;

        fn();
    }

    // 业务逻辑
    function mail() {
        console.log(content);
    }

    envelop(mail);
```

#### 函数嵌套

```js
    let counter = 0;

    function outerFn() {
        function innerFn() {
            counter++;
            console.log(counter);
            // ...
        }
        return innerFn;
    }
    outerFn()();
```

#### 事件处理（异步执行）的闭包

```js
    let lis = document.getElementsByTagName('li');

    for(var i = 0; i < lis.length; i++) {
        (function(i) {
            lis[i].onclick = function() {
                console.log(i);
            }
        })(i);
    }
```

#### 立即执行的嵌套函数

```js
  (function immediateA(a) {
    return (function immediateB(b) {
      console.log(a); // 0
    })(1);
  })(0);
```

#### 当立即执行遇上块级作用域

```js
    let count = 0;

    (function immediate() {
        if(count === 0) {
            let count = 1;

            console.log(count);
        }
        console.log(count);
    })();
```

#### 实现私有变量

```js
    function createStack() {
        return {
            items: [],
            push(item) {
                this.item.push(item);
            }
        }
    }

    const stack = {
        items: [],
        push: function() {}
    }

    function createStack() {
        const items = [];
        return {
            push(item) {
                items.push(item);
            }
        }
    }
```

