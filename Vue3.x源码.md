## Vue3.x源码

### 变化

- 双向绑定的底层从defineProperty变为proxy代理
- watch
- 源码complier部分拆出来，方便外部人员重写，比如mpvue就是将vue2.x fork了一份重写了complier部份
  - complier-core
  - complier-dom
  - complier-sfc
  - complier-ssr
- 新增了ts支持
- 从option api变为composition api
- 模板引擎，vue2.x是通过正则，3.x是通过状态机

### 双向绑定

和vue2.x类似，最大的区别就是2.x用的defineProperty进行数据监听，3.x用的proxy进行监听

在vue2.x中，自定义了defineReactive函数，内部封装了defineProperty对数据监听的操作，而3.x中定义了reactive函数，也是封装了proxy对数据进行监听的操作

在2.x的defineReactive函数中调用了Dep类，Dep类里封装了一些监听操作，将Dep类也叫做watcher(监听者)，而在vue3.x中则是定义了activeEffect对象充当watcher，里面有个run方法可以执行数据的更新，vue3.x分别调用了track函数和trigger函数，分别做了watcher的挂载和收集，track和trigger函数都是调用了deps Map这个栈进行get、set操作，这个depsMap是一个WeekMap实例，充当了watcher栈

对于Ref Hook则单独定义了ref方法，针对单数字的数据做双向绑定处理，ref函数内部封装了一个RefImpl类，返回了这个类实例

###  数据更新

vue3.x定义了effect函数，该函数作用是收集依赖和执行依赖，内部有一个run方法执行依赖，通过触发ref函数和reactive函数内部的get方法收集依赖。

当数据改变的时候和节点挂载等需要变化数据更新的时候都会触发mount函数，mount函数内部执行了effect函数，传入effect函数的第一个参数是回调函数，在回调函数里执行update方法，update方法是通过``el.innerHTML = instance.render()``触发视图的更新变化

```mermaid
graph LR
trigger --> acriveEffect --> run --> mount --> effect --> update --> el.innerHTML=instance.render
```

### diff

vue3.x中的diff算法核心是找到变化的最大上升子序列，再通过倒序对比子序列，复用最大上升子序列，其他新增节点进行mount插入。展开来说就是，将旧数组的顺序作为标准，新数组的排序要符合旧数组的“上升序列”，也就是按照旧数组的规则排列，在排列的这个过程进行移动用到了查找最大上升子序列，提高了查找排列的效率。

vue2.x则是通过双循环暴力遍历找到不同的节点