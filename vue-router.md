## vue-router

### 路由发展

路由的概念是伴随着SPA出现的，在此之前路由跳转是通过服务器端进行控制，比如jsp这些。

```mermaid
graph LR
前端向后台发送请求 --> 后台通过模板引擎的渲染,将一个新的HTML返回给前台
```

在SPA出现后，前端可以**自由的控制组件的渲染来模拟**页面的跳转

#### 总结

- 早期后端路由是根据``url``访问相关的``controller``进行数据资源和模板引擎的拼接，返回前端
- SPA的前端路由是通过`js`根据``url``返回对应的组件加载
  - ``url``的处理
  - 组件加载

### 路由的分类
- `history`路由
- `hash`路由
- `memory`路由 * 处理跨端时的路由
#### hash路由
`window.location.hash = "xx"`
##### 一个基础的hash路由
- hash路由通过控制window.location.hash值来切换url值的变化
- 封装的Router类包含了
	- routes属性，是一个对象，维护了路由栈，其key值是路由路径，value是回调方法
	- refresh方法，用于刷新页面与改变hash值时回调，需要额外通过监听`load`、`hashchange`事件传入回调方法执行，执行的回调主要是用于修改组件内容(因为是SPA所以只有内容发生改动)
	- route方法，通过取出路径对应的回调并执行回调
```html
<div id="container" >
<button onclick="window.location.hash = '#'">首页</button>
<button onclick="window.location.hash = '#about'">关于我们</button>
<button onclick="window.location.hash = '#user'">用户列表</button>
</div>
<div id="context"></div>
```

```js
class BaseRouter {
constructor() {
	this.routes = {};
	this.refresh = this.refresh.bind(this);
	window.addEventListener('load', this.refresh);
	window.addEventListener('hashchange', this.refresh);
}

route(path, callback) {
	this.routes[path] = callback || function() {}
}

refresh() {
	const path = `/${window.location.hash.slice(1) || ''}`;
	this.routes[path]();
}

}
  

const Route = new BaseRouter();

Route.route('/about', () => changeText("关于我们页面"));
Route.route('/user', () => changeText("用户列表页"));
Route.route('/', () => changeText("首页"));

function changeText(arg) {
	document.getElementById('context').innerHTML = arg;
}
```
#### history路由
`history./\(go|back|replace|push|forward)/`
通过直接操作url来实现跳转
##### 一个基础的history路由
- 与hash路由不同的是，通过直接修改url的值进行页面跳转，比如a标签也可以作为vue-router中link的存在
- 与hash路由一样，要维护一个routes属性，用于存储路由栈
- 有一个init方法，一开始进入就执行，判断当前目录是否有回调值，有就执行，比如进入根目录的时候执行一些页面渲染
- route方法，和hash路由一样，用于执行routes路由栈的回调
- go方法，调用原生的`windwo.history.pushState`跳转页面，并执行路由栈routes的回调
- `_bindPopstate`方法，从命名表示私有方法，在construct初始化的时候调用，用于监听`popState`方法，查看是否有状态变化，有的话修改路径值并执行回调更新页面渲染，因为window.history是提供了go|back|replace等方法跳转，当路由状态变化的时候popState可以监听到，所以手动实现的一个history路由的话统一通过监听popState
```html
<div id="container">
	<a href="./" >首页</a>
	<a href="./about">关于我们</a>
	<a href="./user">用户列表</a>
</div>
<div id="context"></div>
```

```js
class BaseRouter {
	constructor() {
		this.routes = {};
		this._bindPopstate();
		this.init();
	}

	init(path) {
		window.history.replaceState({path}, null, path);
		const cb = this.routes[path];
		if(cb) {
		cb();
	}
}
route(path, callback) {
this.routes[path] = callback || function() {}
}

  

go(path) {
	window.history.pushState({path}, null, path);
	const cb = this.routes[path];
	if(cb) {
	cb();
	}
}

  

	_bindPopstate() {
		window.addEventListener('popstate', e => {
		const path = e.state && e.state.path;
		this.routes[path] && this.routes[path]();
		})
	}
}

  

const Route = new BaseRouter();
Route.route('./about', () => changeText("关于我们页面"));
Route.route('./user', () => changeText("用户列表页"));
Route.route('./', () => changeText("首页"));

  
function changeText(arg) {
	document.getElementById('context').innerHTML = arg;
}

  

container.addEventListener('click' , e => {
	if(e.target.tagName === 'A') {
	e.preventDefault();
	Route.go(e.target.getAttribute('href'))
}

})
```

#### 问题1：hash路由和history路由的区别
- hash路由一般会携带一个`#`号，不够美观。history路由不存在这个符号
- 默认hash路由是不会向浏览器发出请求，一般用于锚点，history中go/back/forward以及浏览器的前进后退按钮**一般**都会向服务端发起请求，history中的所有url内容服务端都能获取到
- 基于以上情况，hash模式不支持SSR，但是history模式可以做
- history在部署的时候，茹Nginx需要只渲染首页，让首页路径重新跳转要注意如何部署
```nginx
# 单个服务器部署
location / {
  try_files uri $uri /xxx/main/index.html
}
# 存在代理的情况
location / {
  rewrite ^ /file/index.html break; #代表的是xxx.cdn的资源路径
  proxy_pass https://www.xxx.cdn.com;
}
```

#### 问题2：history.go / back一定会刷新吗
- 要根据指定页面和当前页面的构建关系动态决定，当缓存不可用的时候就会刷新
#### 问题3：pushState会触发popState事件吗
popState是监听其他的操作
- pushState/replaceState都不会触发popState事件，需要触发页面的重新渲染才会触发
- popState什么时候触发
	- 点击浏览器的前进、后退按钮
	- back / foward / go

### Router异步组件
动态路由包括`React.lazy`、`import()`就是一种对代码进行动态拆分的技术，一般叫做`code splitting`。在需要的时候才进行加载，在webpack中可以使用chunksplit配合对路由import的时候添加前缀注释进行动态路由分割
```js
{
path: '/about',
name: 'About',
component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
}
```

### 路由守卫
**路由守卫的hook触发流程**，路由守卫是通过promise链式调用来同步执行流程调用对应的hook，核心方法是navigate
1. 【组件】- 前一个组件`beforeRouteLeave`
2. 【全局】- `router.beforeEach`
3. 【组件】- 如果是路由的参数变化，触发`beforeRouteUpdate`
4. 【配置文件】里，下一个路由的`beforeEnter`
5. 【组件】- 内部声明的`beforeRouteEnter`
6. 【全局】- 调用`beforeResolve`
7. 【全局】的`router.afterEach`
