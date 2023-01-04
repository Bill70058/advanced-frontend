## Vue2.x源码

### 简易vue2.x

在处理双向绑定的时候主要核心是defineProperty，在watch中也用到该原理

### watch实现

在定义每一个变量的时候都会触发defineReactive函数，该函数内部是递归自身处理响应式绑定。也就是说，当新变量被定义的时候会递归自身调用赋予新变量响应式，

并且在此时，get函数里会调用Dep类的add方法，这里是为了做依赖收集，Dep类为每个变量都设置了一个watcher，add方法则是把这些watcher都收集到一个栈里，在set方法的时候调用watcher里的notify方法，执行收集到的watcher

### complier实现

complier处理了模板引擎，对于不同类型的节点，比如纯文本、元素节点等做了不同的渲染判断，通过若干正则进行匹配替换，在处理的过程中对视图数据添加了Watcher做了数据劫持，当watcher发现数据变化时会做出更新

