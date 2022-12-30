## Vue高级用法

### Vue特征一：模板化

#### 插槽

为了解决模板化里面，结构化传递的问题所引入的

#### 默认插槽

组件外部维护参数以及结构，内部安排位置

#### 具名插槽

以name标识插槽的身份，从而在组件内部做到可区分

#### 作用域插槽

在vue2.6之前使用关键字slot-scope，在2.6之后使用v-slot指令

#### 模板对数据的二次加工

当模板内渲染数据时需要对数据二次加工的情况，比如数据是0、1，需要渲染为“是”“否”

1. 可以使用watch、computed进行数据加工
2. 使用函数返回值
3. 使用v-html

```vue
<h v-html="count > 0 ? '是' : '否'"></h>
```

4. 使用过滤器

#### jsx

##### 更自由的基于js书写

vue的编译路径：template -> render -> vm.render -> vm.render()，而jsx是直接写在render方法的返回

1. 模板编译成render函数
2. render函数挂载到当前实例上
3. 执行实例render函数

### Vue特征二：组件化

#### 传统模板化

```vue
Vue.component('component', {
    template: '<h1>组件</h1>'
})
new Vue({
    el: '#app'
})
// functional components
```
* 抽象复用
* 精简 & 聚合

#### vue的几种组件化及引用方式

- 普通组件：将一块业务功能封装为组件，包含了模板代码，js代码，也可能包含了css代码
- 混入mixin：将逻辑代码抽离出来，只有js代码的一个组件
- 继承拓展extend：作用和mixin一样
- 整体拓展extend：直接在Vue实例上调用Vue.extend，是全局的拓展
- Vue.use插件：通过use方法在全局引用插件扩展功能

#### 混入mixin

* 应用： 抽离公共逻辑（逻辑相同，模板不同，可用mixin），赋值要以数组的方式传入
* 缺点： 数据来源不明确
* 合并策略
  - 递归合并，除了data根层面的属性会合并，内部如果有对象的话也会递归合并  
  - data合并冲突时，以组件优先
  -  生命周期回调函数不会覆盖，会先后执行，优先级为先mixin后组件

```js
// minxin.js
export default {
    data() {
        return {
            msg: '我是mixin',
            obj: {
                title: 'mixinTitle',
                header: 'mixinHeader'
            }
        }
    },
    created() {
        console.log('mixin created');
    }
}
```

```vue
// mixin.vue
<template>
 // ... 
 </template>

<script>
import mixinDemo from './mixin.js'
  export default {
    mixins: [mixinDemo],
    data() {
      return {
        
      }
    }
  }
</script>
```

#### 继承extend

* 应用： 拓展独立逻辑
* 与mixin的区别，传值mixin为数组
* 合并策略
  * 同mixin，也是递归合并
  * 合并优先级 组件 > mixin > extends
  * 回调优先级 extends > mixin

```js
// extend.js
export default {
    data() {
        return {
            msg: '我是extends',
            obj: {
                title: 'extendsTitle',
                header: 'extendsHeader',
                shoes: 'extendsShoes'
            }
        }
    },
    created() {
        console.log('extends created');
    }
}
```

```vue
<template>
//...
</template>
<script>
import extendDemo from './extend.js'
 export default{
   extends: extendDemo,
   data() {
     return {}
   }
 }
</script>

```

#### 整体拓展类extend

从预定义的配置中拓展出来一个独立的配置项，进行合并

在源码中可以看到Vue实例会进行各种合并配置项 -- merge options，在全局使用Vue.extend可以创建一个全局的拓展项，其实现机制就是vue进行了merge options

```js
// main.js

// .....

// 拓展一个构造器
let _baseOptions = {
  data: function() {
    return {
      course: 'zhaowa',
      session: 'vue',
      teacher: 'yy'
    }
  },
  created() {
    console.log('extend base')
  }
}
const BaseComponent = Vue.extend(_baseOptions)
// 基于_BaseComponent，再拓展逻辑
new BaseComponent({
  created() {
    console.log('extend created')
  }
})
// 执行顺序是extend base -> extend created -> 组件自身的生命周期
```

#### Vue.use -- 插件

* 注册外部插件，作为整体实例的补充
* 会除重，不会重复注册
* 手写插件
  * 外部使用Vue.use(myPlugin, options)
  * 默认调用的是内部的install方法

```js
// 手写插件
// plugin.js
export default {
    install: (Vue, option) => {
        Vue.globalMethod = function() {
            // 主函数
        }
        Vue.directive('my-directive', {
            bind(el, binding, vnode, oldVnode) {
                // 全局资源
            }
        })
        Vue.mixin({
            created: function() {
                console.log(option.name + 'created');
            }
        })
        Vue.prototype.$myMethod = function() {}
    }
}
```

插件调用的是install方法，在install方法提供了vue实例和合并选项(外部传参)，在install方法内可以灵活处理一系列需要的操作，之后将方法挂载到实例原型链上

