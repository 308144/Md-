# 总结

这个京东的框架是21年7月份推出的！时间短，没有其qiankun的生态、或者经验错误的积累，但是他对新手十分友好！上手真的挺简单，官方文档写的很齐全，目前把分销提出来没有碰到什么bug，但是想要在生产环境上使用的话，建议还是要谨慎一点。

分离的时候！因为是vite的项目！确实碰到了一些问题！

# 问题

我们办法现在在一个项目中同时使用vite和vue项目React！

## 什么是微前端？

微前端是一种类似于微服务的架构，是一种由**独立交付**的多个**前端应用**组成整体的架构风格，将前端应用分解成一些更小、更简单的能够独立开发、测试、部署的应用，而在用户看来仍然是内聚的单个产品。

简单来说：就是一个项目里可以内嵌多个子项目，子项目可以单独开发、部署并且技术框架无关。

## 使用场景

- 大规模企业级 Web 应用开发；

- 跨团队及企业级应用协作开发；
- 俗称屎山的项目，就是用的技术比较老然后还堆叠了很多业务要重构需要花很大成本；

## microApp介绍

```
看官方文档发现上来microApp就表明来意，`microApp`并没有延续single-spa的思路
```

micro-app借鉴了WebComponent的思想，通过CustomElement结合自定义的ShadowDom，将微前端封装成一个类WebComponent组件，从而实现微前端的组件化渲染。并且由于自定义ShadowDom的隔离特性，是目前市面上接入微前端成本最低的方案。

##### 概念图

它在 **基座应用** 和 **子应用** 之间充当桥梁胶水的作用。

![image](https://img10.360buyimg.com/imagetools/jfs/t1/168885/23/20790/54203/6084d445E0c9ec00e/d879637b4bb34253.png)

##### micorApp的优势

- **使用简单。** 将功能封装到 WebComponent 中
- **零依赖。** 无依赖、更高的扩展性
- **兼容所有框架** 技术栈无关

## microApp项目的应用

##### 基座

通过**micro-app**标签可以渲染出一个子应用，它又三个参数

- name：名称（必填）

- url：子应用页面地址（必填）

- baseurl：baseurl是基座应用分配给子应用的路由前缀（可选）

  注意：

  - 1、如果基座是history路由，子应用是hash路由，不需要设置基础路由baseroute
  - 2、如果子应用只有一个页面，没有使用`react-router`，`vue-router`之类，也不需要设置基础路由baseroute

```
<div>
			<h1>React项目</h1>
			{/* 端口号通过.env进行设置 */}
			{/* name必填、url必填、baseroute选填 */}
			<micro-app
				name="app1"
				url="http://localhost:3001/"
				baseroute="/app1"
			></micro-app>
</div>
```

##### 基座路由

- 1、基座是hash路由，子应用也必须是hash路由

- 2、基座是history路由，子应用可以是hash或history路由

  注意：官方推荐使用history的路由

```
  <Routes>
        {/* 在'/'的时候我就渲染AppLayout组件 */}
        <Route
          path="/*"
          element={
            <AppLayout>
              {/* 如果AppLayout组件的路径事/app1/的话就加载AppOne */}
              <Routes>
                <Route path="/app1/*" element={<AppOne />} />
                <Route path="/app2/*" element={<AppTwo />} />
              </Routes>
            </AppLayout>
          }
        />
  </Routes>
```



##### 子应用

通常基座应用和子应用各有一套路由系统，为了防止冲突，基座需要分配一个路由给子应用，称之为基础路由，子应用可以在这个路由下渲染，但不能超出这个路由的范围，这就是基础路由的作用。

如下： **window**.__MICRO_APP_BASE_URL__是由基座应用下发的路由前缀，在非微前端环境下，这个值为undefined

使用：

###### react项目中路由位置进行使用

```
root.render(
	<React.StrictMode>
		<BrowserRouter basename={window.__MICRO_APP_BASE_ROUTE__ || "/"}>
			<App />
		</BrowserRouter>
	</React.StrictMode>
);
```

###### vue项目中路由位置进行使用

```
const router = createRouter({
	routes,
	history: createWebHistory(
		window.__MICRO_APP_BASE_ROUTE__ || process.env.BASE_URL
	),
});
export default router;
或者
const router = new VueRouter({
  mode: 'history',
  // __MICRO_APP_BASE_ROUTE__ 为micro-app传入的基础路由
  base: window.__MICRO_APP_BASE_ROUTE__ || process.env.BASE_URL,
  routes,
})
```

##### 跨域的问题

###### react项目中跨域

```
module.exports = {
    devServer: (_) => {
        const config = _;
        config.headers = {
            "Access-Control-Allow-Origin": "*",
        };
        return config;
    },
};
```

###### vue项目中跨域

```
module.exports = {
    devServer: {
        port: 3002,
        headers: {
            "Access-Control-Allow-Origin": "*",
        },
    },
};
```

## micorApp基础介绍

### micorApp传值（重要）

**每个应用的路由实例都是不同的，应用的路由实例只能控制自身，无法影响其它应用，包括基座应用无法通过控制自身路由影响到子应。**

常见的问题如:<span style="color:red">开发者想通过基座应用的侧边栏跳转，从而控制子应用的页面，这其实是做不到的，只有子应用的路由实例可以控制自身的页面</span>

##### 基座—>子应用发送数据

```
方式1: 通过data属性发送数据
<micro-app
  name='my-app'
  url='xx'
  data={data} // data只接受对象类型，采用严格对比(===)，当传入新的data对象时会重新发送
/>

常用的
方式2: 手动发送数据
microApp.setData('my-app', {type: '新的数据'})
```

##### 子应用获取—>基座数据

```
方式2：监听基座传来的数据
监听数据
window.microApp?.addDataListener(dataListener:function类型)
// 解绑指定函数
window.microApp?.removeDataListener(dataListener)
// 清空当前子应用的所有绑定函数(全局数据函数除外)
window.microApp?.clearDataListener()
方式2：主动获取数据
window.microApp?.getData() // 返回data数据

//应用
if (window.__MICRO_APP_BASE_ROUTE__) {
    // @ts-ignore
    window.microApp.addDataListener((data) => {
      // 当基座下发跳转指令时进行跳转
      if (data.path) {
        navigate(data.path);
      }
    });
  }
```

##### 子应用—>基座发送数据

```
window.microApp?.dispatch({type: '子应用发送的数据'})
```

##### 基座获取—>子应用数据

```
方式1: 监听自定义事件
<micro-app
  name='my-app'
  url='xx'
  data={data}
  onDataChange={(e) => console.log(e.detail.data)}
/>
方式2: 手动绑定监听函数
microApp.addDataListener(appName: string, dataListener: Function)
// 解绑监听my-app子应用的函数
microApp.removeDataListener(appName: string, dataListener: Function)
// 清空所有监听appName子应用的函数
microApp.clearDataListener(appName: string)



常用！！
方式3：主动获取数据
microApp.getData(appName) // 返回子应用发送的data数据
```

<span style="color:red">数据在event.detail.data字段中，子应用每次发送数据都会重新触发事件 <br/>onDataChange函数在子应用卸载时会自动解绑，不需要手动处理</span>

##### 全局数据通信

```2
发送数据
基座
microApp.setGlobalData({type: '全局数据'})
子应用
window.microApp?.setGlobalData({type: '全局数据'})


接收数据
microApp.addGlobalDataListener(dataListener: Function, autoTrigger?: boolean)
// 解绑指定函数
microApp.removeGlobalDataListener(dataListener)
// 清空基座应用绑定的全局数据函数
microApp.clearGlobalDataListener()
```

### JS沙箱

使用**Proxy**拦截了用户全局操作的行为，防止对**window**的访问和修改，避免全局变量污染。`micro-app`中的每个子应用都运行在沙箱环境，以获取相对纯净的运行空间。

<span style="color:red">官方建议：沙箱是默认开启的，正常情况下不建议关闭，以避免出现不可预知的问题。</span>

注意：

##### 子应用在沙箱环境下如何获取真实的window

- 1、new Function("return window")() 或 Function("return window")()

  ##### 子应用抛出错误：**xxx未定义**

- xxx is not defined
- nxxx is not a function
- Cannot read properties of undefined



这种原因事在js中var  name或者function事顶级变量并会泄露为全局变量，可以通过window.name或者name方式去访问；

<span style="color:red">但是在微前端中顶级变量不会泄露为全局变量，所以会报错；</span>

解决方式：

将 var name 或 function name () {} 修改为 window.name = xx或者使用插件系统的方式的方式

### 样式隔离

MicroApp的样式隔离是默认开启的，开启后会以`<micro-app>`标签作为样式作用域，利用标签的`name`属性为每个样式添加前缀，将子应用的样式影响禁锢在当前标签区域。

```
.test {
  color: red;
}

/* 转换为 */
micro-app[name=xxx] .test {
  color: red;
}
```

<span style="color:red">官方建议：但基座应用的样式依然会对子应用产生影响，如果发生样式污染，推荐通过约定前缀或CSS Modules方式解决。</span>

##### 所有应用中禁用样式

这主要通过`start`方法进行全局配置，设置后所有应用的**样式隔离都会停止。**

```
import microApp from '@micro-zoe/micro-app'

microApp.start({
  disableScopecss: true, // 默认值false
})
```

如果希望在某个子应用中不受全局配置控制，可以设置`disableScopecss='false'`

```
<micro-app name='xx' url='xx' disableScopecss='false'></micro-app>
```

##### 在某个应用里禁止

设置后，当前应用的所有css都不会进行样式隔离。

```
<micro-app name='xx' url='xx' disableScopecss 或 disable-scopecss></micro-app>
```

##### 在某个文件里禁用

可以在你的css文件中使用以下格式的注释来禁用样式隔离：

```
/* ! scopecss-disable */
.test1 {
  color: red;
}
/* ! scopecss-enable */
```

如果想在整个文件范围内禁用样式隔离，将 `/* ! scopecss-disable */` 注释放在文件顶部：

```
/* ! scopecss-disable */
...
```

##### 在某一行禁用

在文件中使用以下格式的注释在某一特定的行上禁用样式隔离：

```
/* ! scopecss-disable-next-line */
.test1 {
  color: red;
}

.test2 {
  /* ! scopecss-disable-next-line */
  background: url(/test.png);
}
```

### 元素隔离

```
大栗子:基座应用和子应用都有一个元素`<div id='root'></div>`，此时子应用通过`document.querySelector('#root')`获取到的是自己内部的`#root`元素，而不是基座应用的。
```

元素隔离的概念来自ShadowDom，即ShadowDom中的元素可以和外部的元素重复但不会冲突，micro-app模拟实现了类似ShadowDom的功能，元素不会逃离`<micro-app>`元素边界，子应用只能对自身的元素进行增、删、改、查的操作。

### 预加载

预加载是指在应用尚未渲染时提前加载资源并缓存，从而提升首屏渲染速度。

预加载并不是同步执行的，它会在浏览器空闲时间，依照开发者传入的顺序，依次加载每个应用的静态资源，以确保不会影响基座应用的性能。

##### microApp.preFetch（Array**<app>**|Function=>Array**<app>**）

preFetch接受app数组或一个返回app数组的函数，app的值如下：

```
app: {
  name: string, // 应用名称，必传
  url: string, // 应用地址，必传
  disableScopecss?: boolean // 是否关闭样式隔离，非必传
  disableSandbox?: boolean // 是否关闭沙盒，非必传
}
```

##### 使用方式

```js
import microApp from '@micro-zoe/micro-app'

// 方式一
microApp.preFetch([
  { name: 'my-app', url: 'xxx' }
])

// 方式二
microApp.preFetch(() => [
  { name: 'my-app', url: 'xxx' }
])

// 方式三
microApp.start({
  preFetchApps: [
    { name: 'my-app', url: 'xxx' }
  ],
})
```

### 插件系统

##### 官方定义

```js
微前端的使用场景非常复杂，没有完美的沙箱方案，所以我们提供了一套插件系统，它赋予开发者灵活处理静态资源的能力，对有问题的资源文件进行修改。插件系统的主要作用就是对js进行修改，每一个js文件都会经过插件系统，我们可以对这些js进行拦截和处理，它通常用于修复js中的错误或向子应用注入一些全局变量。
```

##### 个人理解

由于沙箱系统的并不是特别完美，而且还必须需要，所以出现了一套插件系统！主要的作用就是操作js，每个js文件都会经过js进行处理和拦截，可以用来修复js的错误！

##### 适用场景

通常我们无法控制js的表现，比如在沙箱中：

<span style="color:red">顶层的变量是无法泄漏为全局变量的（如 var xx = , function xxx 定义变量，无法通过window.xx 访问），导致js报错，此时开发者可以通过插件对js进行修改处理。</span>

##### 使用方式

```
import microApp from '@micro-zoe/micro-app'
//基座
microApp.start({
  plugins: {
    // 全局插件，作用于所有子应用的js文件
    global?: Array<{
      // 可选，强隔离的全局变量(默认情况下子应用无法找到的全局变量会兜底到基座应用中，scopeProperties可以禁止这种情况)
      scopeProperties?: string[],
      // 可选，可以逃逸到外部的全局变量(escapeProperties中的变量会同时赋值到子应用和外部真实的window上)
      escapeProperties?: string[],
      // 可选，传递给loader的配置项
      options?: any,
      // 必填，js处理函数，必须返回code值
      loader?: (code: string, url: string, options: any) => code 
    }>
  
    // 子应用插件
    modules?: {
      // appName为应用的名称，这些插件只会作用于指定的应用
      [appName: string]: Array<{
        // 可选，强隔离的全局变量(默认情况下子应用无法找到的全局变量会兜底到基座应用中，scopeProperties可以禁止这种情况)
        scopeProperties?: string[],
        // 可选，可以逃逸到外部的全局变量(escapeProperties中的变量会同时赋值到子应用和外部真实的window上)
        escapeProperties?: string[],
        // 可选，传递给loader的配置项
        options?: any,
        // 必填，js处理函数，必须返回code值
        loader?: (code: string, url: string, options: any) => code 
      }>
    }
  }
})
```

例子

操作顶层的变量是无法泄漏为全局变量的eg：var name  ===>window.name

```
import microApp from '@micro-zoe/micro-app'

microApp.start({
  plugins: {
    modules: {
      'appName1': [{
        loader(code, url, options) {
          if (url === 'xxx.js') {
            code = code.replace('var abc =', 'window.abc =')
          }
          return code
        }
      }],
      'appName2': [{
        scopeProperties: ['key', 'key', ...], // 可选
        escapeProperties: ['key', 'key', ...], // 可选
        options: 配置项, // 可选
        loader(code, url, options) { // 必填
          console.log('只适用于appName2的插件')
          return code
        }
      }]
    }
  }
})
```

