## React的class组件的问题

- 不同的生命周期会使逻辑变得分散且混乱，不易维护和管理；
- 时刻需要关注`this`的指向问题；

## StrictMode

`StrictMode`见名知意，严格模式，用于检测`react`项目中的潜在的问题，`StrictMode` 不会渲染任何可见的 `UI` 。它为其后代元素触发额外的检查和警告。

- ①识别不安全的生命周期。
- ②关于使用过时字符串 `ref API` 的警告
- ③关于使用废弃的 `findDOMNode` 方法的警告
- ④检测意外的副作用
- ⑤检测过时的 `context API`

## JSX

实际上，JSX 仅仅只是 `React.createElement(component, props, ...children)` 函数的语法糖。如下 JSX 代码：

```
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>

编译成
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

```
如果没有子节点，你还可以使用自闭合的标签形式，如：
<div className="sidebar" />

编译成
React.createElement(
  'div',
  {className: 'sidebar'}
)
```

## 生命周期

https://segmentfault.com/img/bVcZbV5/view

## React与babel关系

JSX通过babel编译，而babel实际上把JSX编译给`React.createElement()`调用。如下JSX代码：

```ini
const element=(
		<h1 className='greeting'>Hello,world</h1>
)
```

是等同于以下的语句的：

```php
const elem = React.createElement(
 'h1',
    {className:'greeting'},
    'Hello world'
)
```

`React.createElement()`方法会首先进行一些避免BUG的检查，然后返回类似以下例子的对象：

```css
const element = {
    type: 'h1',
    props: {
        className: 'greeting',
        children: 'Hello, world'
    }
}
```

这样的对象，则称为`React元素`，代表所有呈现在屏幕上的东西。React正是通过读取这些对象来构建DOM，并且保持数据和UI同步的

## React-router



### 基础

react-router出口是Outlet

跳转的是 <Link to=""></Link>

### 跳转传参数

#### useSearchParams

它可以获取到url上的传入的参数！

用法

url:http://localhost:8000/#/automationTools/applicationManagement/pageDecoration?template=distribution_bilingual&pageId=637d77b63b38bf15a0c47196

```
const [query] = useSearchParams()
  const pageId = query.get('pageId')
  const template = query.get('template')
  console.log(template,'template');//distribution_bilingual
console.log(pageId,'pageId');//637d77b63b38bf15a0c47196
```

window.loaction.origin:可以获取到该项目的域名

#### useParams

```ini
			<Switch>
             <Route path="/:id" children={<Child />} />
           </Switch>
           
//局限性需要配置路由，返回的是个对象类型{id:''}
const params=useParams()
```

### 事件方式传参数

```ini
const navigate=useNavigate()
const dianji=()=>{
		navigate('/详情页',{state:{useName:'张三'}})
}
<button onclick={dianji}>点击</button>


//获取数据
const location=useLocation()
console.log(location.state.useName)
```

