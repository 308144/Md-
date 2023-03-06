# npm包打包部署上传阿里云oss

## 目录

1. 前言
2. 阿里云oss配置npm包Bucket
3. 创建简单的npm包demo
4. 打包部署脚本开发
5. 在jenkins上部署使用

## 一. 前言

公司项目多了总会有一些通用的组件和业务逻辑，为了更好的进行复用和维护，一般会封装成npm包进行使用。在开发完成后可以发布到npm网站使用，也有公司为了保护核心源码或者更快的访问速度，也会自己搭建npm私服。使用的时候会从npm远程库下载到本地，在package.json里面的依赖对象以依赖名称和版本号的方式:

```json
"devDependencies": {
   "typescript": "^4.8.4"
 }
```

除了搭建npm私服之外，也有其他的方法。比如公司原先采用git仓库tag方法，和现在的oss存储tgz地址方案，上传到oss上面，在本地package.json引用oss包的tgz路径使用。

```json
"dependencies": {
	 "new-assets": "https://h5-fc-code.oss-cn-hangzhou.aliyuncs.com/package/new-assets/new-assets-1.1.0.tgz"
}
```

使用tgz地址方案后，在执行npm i下载依赖时，npm会把远程的tgz包下载下来，解压到node_modules中，解压后的文件夹是包的名称，我们就可以正常使用了。

这样实现了npm包的使用，也区分了版本，但是目前每次发布新版本包流程比较麻烦，需要经过以下步骤：

1. 在npm包本地进行开发。
2. 开发完成后进行打包。
3. 新建和包同名的文件夹(因为解压后的文件名称和压缩时的名称是一样的，所以要创建一个和包名称同名的文件夹，把打包后的资源和package.json复制进来，打包tgz文件)。
4. 把package.json和打包后资源复制到和包同名的文件夹中。
5. 借助工具或命令行压缩和包同名的文件夹为 **包名-版本号.tgz文件**。
6. 登录阿里oss，把tgz包上传到指定包所在的位置。

虽然可以把中间几步由脚本来实现，但是把脚本写到npm包工程里面的话，对多很多额外的代码和依赖，每次开发新npm包都要把打包脚本复制过来。所以更好的方式是把打包部署脚步单独抽离出来，实现给各个npm包进行打包和部署oss。

## 二. 阿里云oss配置npm包Bucket

### 2.1 开通阿里云oss

先登录[阿里云](https://www.aliyun.com)，登录成功后，点击右上角控制台，再在左侧菜单中找到对象存储oss。要先开通一下对象存储oss，现在有一元开3个月的活动。

### 2.2 创建Bucket

开通完成后，点击Bucket列表，新建一个bucket，每个bucket相当于一个单独的存储空间。

![1](https://guojiongwei.top/static/1.png)

创建bucket成功后，点击创建的bucket名称查看详情，可以看到几个关键的信息，当前bucket的文件列表，对外公网内网的访问域名，以及一个域名管理。

![2](https://guojiongwei.top/static/2.png)

- 文件列表：存储着在当前bucket空间下所有的文件。
- 公网内网的访问域名：可以通过阿里云自动分配的域名加上文件路径来访问oss中我们上传的文件。
- 域名配置：可以在这里绑定我们自己的域名，通过自己的域名来访问oss文件。

### 2.3 绑定自己的域名(非必须)

点击上图下方的域名管理，点击绑定域名按钮，然后在弹窗输入自己访问oss的域名，如果是在阿里云备案的域名会自动添加域名和配置验证记录，我们也可以在这里申请费的ssl证书给域名加上https。

绑定好自己的域名和证书后的页面如下图。

![3](https://guojiongwei.top/static/3.png)

### 2.4 测试对象oss文件访问

找到上面文件管理中的文件列表，点击上传文件按钮，上传一个文件，上传成功后打开文件详情，可以看到文件的访问地址，通过这个地址就可以访问到我们上传的文件了。

![5](https://guojiongwei.top/static/5.png)

配置了自有域名的话这里就会显示自有域名的url地址，如果没配置或者设置不使用自有域名，那访问地址的url域名会变成阿里云为我们分配的公网访问域名。

这里还有一个问题，就是阿里云oss对了安全，默认开启了文件访问的密钥token和过期时间，这个url地址在一段时间后就不能用了，但我们想让它作为npm包的访问地址，写在package.json里面的。肯定会想让地址永久有效不会过期。

### 2.5 控制bucket文件访问权限

为了解决阿里云oss存储文件访问自带过期时间的情况，需要手动改一下bucket中文件的读写读写策略。

点击权限控制-读写权限，把Bucket ACL由私有的改为公共读的。

![6](https://guojiongwei.top/static/6.png)

修改完成后，再返回文件列表看文件的url地址，就只剩下文件url域名和路径，没有前面的过期时间等query参数了。

开启了公共访问后，为了避免流量浪费，可以在数据安全选项中开启防盗链和跨域设置等策略来限制其他人访问。

### 2.6 创建AccessKey

在上面的介绍里面，上传文件和管理Bucket都是在阿里云oss界面操作，如果要在自己代码中去操作oss，需要创建AccessKey，通过AccessKey来连接oss进行文件的写入查询删除等操作。

在阿里云鼠标划入右上角个人头像，可以看到AccessKey管理，在页面可以看到已经创建的AccessKey列表，没有得话点击创建AccessKey按钮，进行创建，创建完成后可以得到AccessKey ID和Secert。

![7](https://guojiongwei.top/static/7.png)

通过AccessKey ID和Secert可以登录阿里云oss可视化工具[OSSBrowser](https://help.aliyun.com/document_detail/61872.html?spm=5176.8466032.help.dexternal.2ffd1450ooerqQ)，在js中也可以配合oss官方的npm包[ali-oss](https://www.npmjs.com/package/ali-oss)来操作oss。

到这里阿里云对象存储oss的配置就进行的差不多了，需要编写打包脚本的代码了。

## 三. 创建简单的npm包demo

因为要开发打包部署脚本，所以需要先创建一个npm包的demo项目。

新建文件夹**npm-demo**，在文件夹中创建项目的目录结构：

```
├─src              
│  ├─index.ts
│─.gitignore 
│─package.json 
│─tsconfig.json 
```

**index.ts**的代码

```js
/** 返回当前npm-demo包的版本 */
export function getVersionn() {
  return '1.0.0'
}
```

**package.json**：使用tsc对ts文件进行编译打包

**因为第三包不确定性太多，有的需要打包有的不需要打包，最终打包到tgz压缩包的文件不好确认打包哪些文件件，借用npm包发布规则，把要最终需要构建发布的文件定义在package.json的files数组里面。**

```json
{
  "name": "npm-demo",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "dev": "ts-node-dev ./src/index.ts",
    "build": "tsc"
  },
  "devDependencies": {
    "ts-node-dev": "^2.0.0",
    "typescript": "^4.8.4"
  },
  "files": [
    "dist",
    "package.json"
  ]
}
```

**tsconfig.json**：ts配置文件

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "commonjs",
    "lib": ["es2015", "es2016", "es2017", "es2018", "es2019", "es2020"],
    "outDir": "./dist", // 输出的目录
    "rootDir": "./src", // 打包的目录文件
    "resolveJsonModule": true,
    "declaration": true, // 输入声明类型文件
    "declarationDir": "./dist",
    "esModuleInterop": true,
    "typeRoots": ["./types", "node_modules/@types"]
  },
  "include": ["./src/**/*"],
}
```

## 四. 打包部署脚本开发

### 4.1 初始化打包部署脚本项目

和上面npm-demo同目录创建一个部署npm的deploy-npm项目，目录结构：

```
|- packages # 用来放置部署过程中新建的文件夹和tgz包
|- index.js // 脚本主要逻辑
|- oss.js		// 阿里云oss AccessKey存放位置，为git忽略文件，不提交到代码仓库。
│─.gitignore // git忽略文件
│─package.json // 包管理文件
```

**index.js** 下一节讲

**oss.js**，oss密钥配置文件(注意在git代码中忽略上传)

```js
const Oss = require('ali-oss')

/** 创建ali-oss连接实例 */
function createOss() {
  return new Oss({
    accessKeyId: 'accessKeyId',
    accessKeySecret: 'accessKeySecret',
    bucket: 'bucket',
    region: 'region'
  })
}

module.exports = {
  createOss,
  ossHost: 'https://oss.xxx'
}
```

**.gitignore**

```
node_modules
package-lock.json
yarn-lock.json
oss.js
packages/* # 用来放置部署过程中新建的文件夹和tgz包
.DS_Store
```

**package.json**

```js
{
  "name": "deploy-npm",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "ali-oss": "^6.17.1",
    "compressing": "^1.6.2",
    "fs-extra": "^10.1.0",
    "optimist": "^0.6.1"
  }
}
```

在package.json中使用到了四个依赖：

1. ali-oss：连接阿里云oss的npm包;
2. compressing：压缩文件为tgz格式；
3. fs-extra：基于fs封装的npm包，功能更强大。
4. optimist：获取命令行参数。

### 3.2 部署脚本要做的工作

在前言里面提到，现在的npm包新版本发布需要经过以下步骤：

1. 在npm包本地进行开发。
2. 开发完成后进行打包。
3. 新建和包同名的文件夹(因为解压后的文件名称和压缩时的名称是一样的，所以要创建一个和包名称同名的文件夹，把打包后的资源和package.json复制进来，打包tgz文件)。
4. 把package.json和打包后资源复制到和包同名的文件夹中。
5. 借助工具或命令行压缩和包同名的文件夹为 **包名-版本号.tgz文件**。
6. 登录阿里oss，把tgz包上传到指定包所在的位置。

所以部署脚本要做的也是这些事情，只是把逻辑单独抽离出来，打包部署脚本项目的使用思路延续了web端项目部署的思路，例如执行：

```bash
node deploy-npm/index.js --target npm-demo
```

deploy-npm是部署脚本的项目，--target指向的npm-demo是要打包的npm包项目文件夹。

### 3.3 执行流程

**执行命令后的大致思路**

根据命令行参数找到对应的npm包项目所在路径，判断用不用打包，需要就执行npm run build打包。完成后就在packages临时目录下面新建和npm包名称一样的文件夹，然后把package.json中的files文件复制到同名文件夹中，最终把npm同名文件夹压缩为tgz文件，把tzg上传到oss对应npm包目录下面。

之所以要创建和npm包同名的文件夹，是为了保证tgz文件解压后在node_modules中生成的文件夹名称和npm包的名称一致，这样我们才可以引入到。

**具体的实现代码**

```js
const fs = require('fs-extra')
const path = require('path')
const { exec } = require('child_process')
const compressing = require('compressing')
const { argv } = require('optimist')
/** 从oss.js引入阿里云连接函数 */
const { createOss, ossHost } = require('./oss.js')

/** 根据命令行参数获取到本次打包部署的npm包工作目录 */
const target = path.resolve('../', argv.target)
console.log('工作目录: ', target)

/** 封装获取npm包工作目录下文件路径方法 */
const resolveTarget = file => path.join(target, file)
 /** 获取对应npm包文件下package.json的name和version，files，scripts字段。*/
const { name, version, files, scripts } = require(resolveTarget('package.json'))

/** 获取当前packages目录路径 */
const packagesPath = path.join(__dirname, './packages')
/** 如果没有packages文件就创建 */
if(!fs.existsSync(packagesPath)) fs.mkdirSync(packagesPath)
const resolvePackages = file => path.join(__dirname, './packages', file)
/** 在packages目录下npm包同名的文件夹路径 */
const destPath = resolvePackages(name)
const resolveDest = file => path.join(destPath, file)

/** 入口函数 */
function start() {
  if(scripts && scripts.build) {
    /** 借助node子进程child_process模块的exec方法在对应的npm包目录下执行打包操作 */
    exec('npm run build', { cwd: target }, function(err) {
      if(!err) {
        build()
      } else throw err
    })
  } else build()
}

start()

/** 开始打包部署上传 */
async function build() {
  /** 1. 检测在packages目录下是否有npm包同名的文件夹，有就删掉 */
  if(fs.existsSync(destPath)) fs.removeSync(destPath)
  /** 2. 然后再新建文件夹 */
  fs.mkdirSync(destPath)
  /** 3. 把package.json中的files文件列表复制到当前同名文件夹中(关键) */
  await Promise.all(files.map(file => fs.copy(resolveTarget(file), resolveDest(file))))
  /** 4. 定义tgz压缩包名称，为包名称-版本号.tgz */
  const tgzName = `${name}-${version}.tgz`
  /** 5. 定义tgz压缩包放置目录位置，放在packages文件下面 */
  const tgzPath = resolvePackages(tgzName)
  /** 6. 把npm包同名文件夹压缩为指定位置和名称的tgz包 */
  await compressing.tgz.compressDir(destPath, tgzPath)
  /** 7. 创建oss上传链接 */
  const client = createOss()
  /** 8. 把压缩包上传到对应包文件目录里面 */
  await client.multipartUpload(`${name}/${tgzName}`, fs.readFileSync(tgzPath))
  console.log('打包部署完成, 包文件oss地址为: ', `${ossHost}/${name}/${tgzName}`)
  /** 9. 异步删除destPath和tgzPath文件 */
  Promise.all([fs.remove(destPath), fs.remove(tgzPath)])
}

/** 监听Promise的reject时间，抛出异常 */
process.on('unhandledRejection', (p) => {
  throw p
})
```

## 五. 在jenkins上部署使用

### 5.1 部署打包脚本deploy项目

在jenkins上部署使用时比较简单，部署脚本项目deploy-npm只简单创建一个流水线，配置好deploy-npm的git仓库地址，代码拉取下来后，在shell中配置一下npm i安装依赖。

然后记得把oss.js配置文件手动在服务器对应jenkins项目目录下面添加一下。

### 5.2 部署各个npm包项目

部署方式也比较简单，创建对应的jenkins流水线后，配置好项目的git地址，在shell里面添加命令

```bash
npm i --registry https://registry.npm.taobao.org
node ../deploy-npm/index.js --target npm-demo
```

要注意deploy-npm目录的相对路径，可以直接用绝对路径。