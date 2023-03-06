环境变量

`window.__MICRO_APP_BASE_ROUTE__`
 由基应用下发的路由前缀，即 `<micro-app>` 标签的 baseroute 值，用于区分子路由

`window.__MICRO_APP_NAME__`
 在子应用中通过获取应用的 name 值，即 `<micro-app>` 标签的name值。

`window.__MICRO_APP_ENVIRONMENT__`
 判断子应用是否在微前端环境中

`window.__MICRO_APP_BASE_APPLICATION__`
 判断应用是否为基应用，在 `microApp.start()`  后有效

# 两个问题：

1、虽然我们已经在项目中使用vite，但是它启动的时候还是速度不够理想

2、我们有很多部门在一起开发，每次提交拉去的时候都可能会面临冲突问题

![image-20221201155229710](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201155229710.png)

# 分离

我们把项目复制一份，

![image-20221201155326966](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201155326966.png)

然后把分销作为子应用，原有项目作为基座

我们把分销的路由进行注释一下，把其全部分离出来（我们先把分离出来，随后分离成功在去删除依赖，以及删除页面，减轻打包负担，从而可以起服务更快）

![image-20221201155523655](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201155523655.png)

然后就会发现这个项目被单独分离出来了

![image-20221201155540481](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201155540481.png)

## 修改基座

 <span style="color:red">对于vite配置！官方建议关掉沙箱模式！！因为，沙箱在module script 是不支持的import  或export</span>

<span style="color:red">在嵌入vite子应用时，`micro-app`的功能只负责渲染，其它的行为由应用自行决定，这包括如何防止样式、JS变量、元素的冲突。</span>

我们需要创建一个页面，在src/pages文件下，创建一个SubApps文件,并创建index.tsx文件！

写入如下代码

```
import React, { useEffect } from 'react'
import microApp from '@micro-zoe/micro-app'

export default function Distribution() {
  return (
      <micro-app
        name='distribution'
        url='http://localhost:8003/'
        inline // 使用内联script模式
        disableSandbox // 关闭沙箱
      />
  )
}

```

我们把子应用分离出来了，但是我们的基座还有页面，我们把基座关于分销的页面路由的element都指向

![image-20221201155736504](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201155736504.png)然后就会看到页面

这一步我们就是分离成功！

以上就是关于个人项目的私有配置，可以进行参考，如下是一些官方的配置！

# 子应用修改

如上只是分离成功了，但是并没有，达到我们想要的效果，

如果有需要可以看下官方的配置[Vite (jd.com)](https://zeroing.jd.com/micro-app/docs.html#/zh-cn/framework/vite)

## **修改vite.config.js**

上面位置可以直接复制，但是需要配置跨域问题

跨域的问题

```
/* eslint-disable import/extensions */
import { writeFileSync } from 'fs'
import path from 'path'
import { join } from 'path'
...
const env = process.env.ENV

...

export default defineConfig({
  base: process.env.NODE_ENV === 'production' ? cmsPath[env] : '',
  plugins: [
  ....
    (function () {
      let basePath = ''
      return {
        name: "distribution",
        apply: 'build',
        configResolved(config) {
          basePath = `${config.base}${config.build.assetsDir}/`
        },
        writeBundle(options, bundle) {
          for (const chunkName in bundle) {
            if (Object.prototype.hasOwnProperty.call(bundle, chunkName)) {
              const chunk = bundle[chunkName]
              if (chunk.fileName && chunk.fileName.endsWith('.js')) {
                chunk.code = chunk.code.replace(/(from|import\()(\s*['"])(\.\.?\/)/g, (all, $1, $2, $3) => {
                  return all.replace($3, new URL($3, basePath))
                })
                const fullPath = join(options.dir, chunk.fileName)
                writeFileSync(fullPath, chunk.code)
              }
            }
          }
        },
      }
    })(),
  ],


 
...
  },
  server: {
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
   ....
  },
})


```

## 修改index.html

由于我们关闭了沙箱模式

我们的顶级作用于就会泄露为全局作用域

所以我们把index.html中的id="root"换成

```
    <div id="distribution-root"></div>
```

并且在mian.tsx文件中进行修改

![image-20221201172751295](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201172751295.png)这个原因是，基座的项目id是root，我们子应用的id要为root的话，可能会造成一些无法预知的问题

# 基座修改

## **处理子应用静态资源**

官方例子

![image-20221201172947339](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201172947339.png)我们进行修改，

<span style="color:red">   注意是在基座的mian.tsx文件下，进行修改!</span>

```
// entry
// eslint-disable-next-line import/order
import microApp from '@micro-zoe/micro-app'
microApp.start({
  plugins: {
    modules: {
      // appName即应用的name值
      distribution: [
        {
          loader(code) {
            if (process.env.NODE_ENV === 'development') {
              // 这里 basename 需要和子应用vite.config.js中base的配置保持一致
              code = code.replace(/(from|import)(\s*['"])(\/distribution\/)/g, all => {
                return all.replace('/distribution/', 'http://localhost:8001/distribution/')
              })
            }

            return code
          },
        },
      ],
    },
  },
})
```

## 效果

随后我们就可以看到这个样式！也就是我们成功了4分之三了!

# ![image-20221201173359904](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201173359904.png)基座与子应用传值

我们会发现，上面的基座路由与子应用路由并不拼配，我们需要用传值的方式让子应用实例去操作子应用路由

## 在基座的MicroApp的index页面修改

```
import React, { useEffect } from 'react'
import microApp, { EventCenterForMicroApp } from '@micro-zoe/micro-app'

import { useLocation } from 'react-router-dom'

export default function Distribution() {
  const location = useLocation()
  console.log(location.pathname) //distribution/goodsPromotion/list
  // 注意：每个vite子应用根据appName单独分配一个通信对象   创建实例
  window.eventCenterForDistribution = new EventCenterForMicroApp('distribution')
  useEffect(() => {
  //基座发送数据
    microApp.setData('distribution', { path: location.pathname })
  }, [location.pathname])
  return (
      <micro-app
        name='distribution'
        url='http://localhost:8001/'
        inline // 使用内联script模式
        disableSandbox // 关闭沙箱
      />
  )
}

```

## 在子应用的App.tsx文件进行useEffect的监听

```
const navigate = useNavigate()
  useEffect(() => {

    if(window.eventCenterForDistribution) {
      window.eventCenterForDistribution.addDataListener((data) => {
        // 当基座下发跳转指令时进行跳转
        if (data.path) {
          navigate(data.path)
        }
      })
    }
```

## 效果

# x去掉菜单栏头部

我们发现子应用全部嵌套早基座上，包含已经有了的菜单栏，以及头部

我们找到子应用的layou页面进行判断

```
     window.__MICRO_APP_BASE_APPLICATION__
```

这个是，如果在基座环境为true，不再基座为false

我们就可以判断

## ![image-20221201173501117](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201173501117.png)效果

# 最终效果

![image-20221201173515424](C:\Users\Dr.Panda\AppData\Roaming\Typora\typora-user-images\image-20221201173515424.png)
