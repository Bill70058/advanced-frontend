## Vue基础

### vue是如何利用mvvm思想进行项目开发

通过数据的双向绑定

- 利用花括号(模板引擎)，构筑了数据与属兔的双向绑定，模板引擎是通过若干正则实现的
- 通过视图绑定事件来处理数据

### 生命周期

在不同的生命周期发生了什么

- beforCreate: new Vue() - 实例挂载功能
- created: data、props、method、computed - 数据操作、不涉及到vdom和dom
- beforeMount: vDom - 数据操作，但是不可涉及dom
- mounted: Dom - 任何操作
- beforeUdate: vDom更新了的，dom未更新是旧的 - 可以更新数据 
- udated: dom已经更新了 — 谨慎操作数据
- beforeDestory: 实例vm尚未被销毁 — 清空eventBus、reset store、clear计时器
- destoryed: 实例已经被销毁 - 收尾

### 定向监听 -- computed 和 watch

#### 相同点

1. 基于vue的依赖收集机制
2. 都是被依赖的变化触发，进行改变进而进行处理计算

#### 不同点

1. 入和出
   computed: 多入单出 —— 多个值变化，组成一个值的变化
   watch: 单入多出 —— 单个值的变化，进而影响一系列的状态变更

2. 性能
   computed: 会自动diff依赖，若依赖没有变化，会改从缓存中读取当前计算值
   watch: 无论监听值变化与否，都会执行回调

3. 写法上
   computed: 必须有return返回值
   watch: 不一定

4. 时机上
   computed: 从首次生成赋值，就开始计算运行了
   watch: 首次不会运行，除非——immediate：true