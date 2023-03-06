# Egg

### 创建egg项目

npm init egg --type-simple

### 使用egg英文快速形成模板

![微信截图_20221229103912](C:\Users\Dr.Panda\Desktop\Md\图片\微信截图_20221229103912.png)

### 配置下view的插件

绝大多数情况，我们都需要读取数据后渲染模板，然后呈现给用户。故我们需要引入对应的模板引擎。

框架并不强制你使用某种模板引擎

```ini
npm i egg-view-nunjucks --save

// config/plugin.js
  nunjucks : {
    enable: true,
    package: 'egg-view-nunjucks',
  }
// config/config.default.js
 config.view = {
    defaultViewEngine: "nunjucks",
    mapping: {
      ".tpl": "nunjucks",
    },
  };
```

### Scheam、Model

- `Schema`: 一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力
- `Model`: 由Schema发布生成的模型，具有抽象属性和数据库操作能力

### this.ctx.curl()

```ini
var data=await this.ctx.curl('www.baidu.com')
异步请求，记得加上await
curl(url, optionsopt) 

eg:
const result = await this.ctx.curl('http://127.0.0.1:7002/audit/log/batchLog', {
      method: 'POST',
      contentType: 'json',
      dataType: 'json',
      data: ctx.request.body,
    });
```
![微信截图_20221226171259](C:\Users\Dr.Panda\Desktop\Md\图片\微信截图_20221226171259.png)
