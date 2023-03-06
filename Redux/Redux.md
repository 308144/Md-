# Redux

1、redux是的诞生是为了给 React 应用提供「可预测化的状态管理」机制。

2、Redux会将整个应用状态(其实也就是数据)存储到到一个地方，称为store

3、这个store里面保存一棵状态树(state tree)

4、组件改变state的唯一方法是通过调用store的dispatch方法，触发一个action，这个action被对应的reducer处理，于是state完成更新

5、组件可以派发(dispatch)行为(action)给store,而不是直接通知其它组件

6、其它组件可以通过订阅store中的状态(state)来刷新自己的视图

## State

就是我们传递的数据

1、服务端返回的数据

2、当前组件的state

3、全局的State

## Action事件

Action是把数据从应用传到store的载体，他是store数据的唯一来源，一般来说，我们通过store.dispath()讲action传递给store

组件（component）————————>Store

​									store.dispatch

​									发送一个action

1、本质上Action就是一个js的对象！

2、Action对象内部必须有一个type属性来表示要执行的动作！

3、多数情况下type要被定义成一个字符串！

4、除了type字段，action其余结构随意定义

5、它并不会直接更新state，只是描述有事情要发生、有这个意图！

6、在项目里面我们更喜欢用action去创建函数

eg:

```
//action
{
type:'字符串常量',
info:{...},
isLoading:true
...
}
```

```ini
function addAction(params){
//返回一个对象！
		return{
			type:'add',
			...params
		}
}
```

## Reducer

可以叫他为工厂函数

Reducer本质是一个函数，它用来相应发送过来的actions，然后经过处理，把state发送给Store的！

注意：在reducer函数中需要return返回值，这样Store才能接受到数据

它有两个参数，一个是初始化state，第二个是action！

![image-20221124154332133](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124154332133.png)

![image-20221124154424081](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124154424081.png)

## Store

store的作用就是把action和reducer联系在一起的<span style="color:red">对象</span>

```ini
主要职责是：
维持应用的state
提供getState()方法获取state
提供dispatch()方法发送action
提供subscribe()来注册监听//注册监听reducer的返回值从而获取到返回值
提供subscribe()的返回值来注销监听
```

```
import {createStore} from "redux"
const store =createStore(传递reducer)
```

## Redux项目

![image-20221124164506569](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124164506569.png)

![image-20221124164756725](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124164756725.png)

![image-20221124171902027](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124171902027.png)



# React-Redux

由于redux的store使用时候，都会在组件里持续引入store，有些麻烦！所以官方react官方定义了react—redux，如下！

![image-20221124180829462](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124180829462.png)

## Provider

看我上边那个代码的**顶层组件**4个字。对，你没有猜错。这个顶级组件就是Provider,一般我们都将顶层组件包裹在Provider组件之中，这样的话，所有组件就都可以在react-redux的控制之下了,**但是store必须作为参数放到Provider组件中去**

```xml
<Provider store = {store}>
    <App />
<Provider>
这个组件的目的是让所有组件都能够访问到Redux中的数据。 
```

## Connect

这个才是react-redux中比较难的部分，我们详细解释一下

首先，先记住下边的这行代码：

```
connect(mapStateToProps, mapDispatchToProps)(MyComponent)
```

#### mapStateToProps

这个单词翻译过来就是**把state映射到props中去** ,其实也就是**把Redux中的数据映射到React中的props中去。**

```
 const mapStateToProps = (state) => {
      return {
      	// prop : state.xxx  | 意思是将state中的某个数据映射到props中
        foo: state.bar
      }
    }
```

然后渲染的时候就可以使用this.props.foo

```scala
class Foo extends Component {
    constructor(props){
        super(props);
    }
    render(){
        return(
        	// 这样子渲染的其实就是state.bar的数据了
            <div>this.props.foo</div>
        )
    }
}
Foo = connect()(Foo);
export default Foo;
复制代码
```

然后这样就可以完成渲染了

#### mapDispatchToProps

这个单词翻译过来就是就是**把各种dispatch也变成了props让你可以直接使用**

```javascript
const mapDispatchToProps = (dispatch) => { // 默认传递参数就是dispatch
  return {
    onClick: () => {
      dispatch({
        type: 'increatment'
      });
    }
  };
}


class Foo extends Component {
    constructor(props){
        super(props);
    }
    render(){
        return(
        	
             <button onClick = {this.props.onClick}>点击increase</button>
        )
    }
}
Foo = connect()(Foo);
export default Foo;
```

组件也就改成了上边这样，可以直接通过this.props.onClick，来调用dispatch,这样子就不需要在代码中来进行store.dispatch了

## React-Redux的项目

效果

![image-20221124181013788](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124181013788.png)

流程

项目准备

![image-20221124182700423](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124182700423.png)

Provider

![image-20221124182920855](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124182920855.png)

Connect

![image-20221124191601939](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221124191601939.png)

代码！

```
处理器reducer
const ini = { count: 0 }
/*
接受两个参数
第一个是state
第二个是action
*/

//reducer要接受action然后进行逻辑处理的
//判断 发送过来的action 是不是我们需要的
//如果是我们需要的，就应该return一个新的state了！
const reducer = (state = ini, action: any) => {
    // console.log("reducer", action);
    switch (action.type) {
        case 'add_action':
            return {
                count: state.count + 1
            }
        case '':
            break
    }
    return state
}
export default reducer
```

```
store
// store！
//redux中的legacy_createStore
import { legacy_createStore as createStore } from 'redux'
import reducer from '../reducer'

const store = createStore(reducer)

export default store
```

```
//mian
import React from "react";
import ReactDOM from "react-dom/client";
import "@/Css/index.css";
// import BaseRouter from '@/router'

import store from "@/redux/react-redux/store";
import { Provider } from "react-redux";

import App from "@/redux/react-redux/App";

ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <Provider store={store}>
    <App />
  </Provider>
);

```

```
PageA
//A是发送方！
// 点击发送数据
import React from "react";
import { connect } from "react-redux";

const PageA = (props: any) => {
  const add = () => {
    props.sendActionL();
  };
  return <div onClick={add}>++</div>;
};

const mapDispatchToProps = (dispatch: any) => {
  return {
    sendActionL: () => {
      dispatch({
        type: "add_action",
      });
    },
  };
};

export default connect(null, mapDispatchToProps)(PageA);

```

```
PageB
import React from "react";
import { connect } from "react-redux";

const PageB = (props: any) => {
  return <div>{props.count}</div>;
};
const mapStateToProps = (state: any) => {
  return state;
};

export default connect(mapStateToProps)(PageB);
```

注意他们都是通过props来进行传递的！

获取传递的方法是props.sendActionL,获取数据是props.count