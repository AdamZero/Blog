# Mock指南

## 前言

由于前后端工作的不同步，在实际工作过程中，经常会发生前端在写界面逻辑的时候无法从后端获取到相应数据的情况，如果把数据直接写死在相应界面，一来污染界面逻辑，二来不方便管理数据，所以mock应运而生，可以帮我们很方便的进行数据模拟，

> ​	完整代码在最后

## 依赖

```js
npm install mockjs -D
npm install supervisor -D
npm install concurrent -D
```



### http

http可以帮我们启动服务挂载访问接口结果

### mockjs

mockjs可以动态渲染模拟数据,详细使用说明请查看[官网](http://mockjs.com/)

### supervisor

supervisor在mockjs中引用文件被修改时会自动更新服务

### concurrently

可以同时运行多个命令，非必须

## 添加mock接口

### 方法1：手动导入

* 优点：更新依赖后可以自动部署，无需重启
* 缺点：需要手动进行依赖

首先根据接口返回结果，定义mock数据data，可以采用写死的数据，也可以采用mockjs写法动态

```js
// /mocks/apis/login/login.js
module.exports={
    code:0,
    msg:'成功',
    data:{
        id:'123',
        token:'12345'
    }
}
```

之后在login文件夹中统一导入管理

```js
// /mocks/apis/login/index.js
const login=require('./login')
const logout=require('./logout')
module.exports={login,logout}
```

为了让mock的数据可以映射到请求地址，需要增加映射文件

```js
// /mocks/apis/map.js
const login=require('./apis/login')
const path="/mock"

module.exports={
    [`${path}/login`]:login.login,
    [`${path}/logout`]:login.logout,
}
```

之后只需要按照index.js中那样，根据map[req.url]来获取data并设置延时返回即可

### 方法二：自动导入

* 优点：省事，无需进行无聊的导入工作
* 缺点：每次更新数据内容之后需要手动重启mock服务

主要思路是通过node的fs模块导入/apis下的文件夹，如果是文件则读取内容并存放，是文件夹则递归

```js
/**
 * 读取文件夹下所有文件映射
 * @param {String} dirPath 绝对路径
 * @param {String} simplePath 相对路径
 */
const readAllFile = (dirPath,simplePath='') => {
  const opt = { encoding: 'utf8' };
  let files = fs.readdirSync(dirPath, opt);
  // 文件轮询
  files.forEach(it => {
    if (!it) return;
    const childPath = `${dirPath}/${it}`;
    const stats = fs.lstatSync(childPath);
    if(stats){
      // 文件则根据路径添加，文件夹则控制路径并递归
      if (stats.isFile()) {
        const name=it.slice(0,it.indexOf('.'));
        apis[`${path}/${simplePath}/${name}`] = fs.readFileSync(childPath, opt);
      } else {
        readAllFile(childPath,simplePath?`${simplePath}/${it}`:it);
      }
    }
  });
};
```

因为是直接读取内容，所以/apis下的文件夹直接定义为请求路径的二级路径，文件名定义为接口名称，路径就和mock数据关联了

> 		文件格式统一使用.json

例如：要访问/login/login接口，则在/apis/login文件夹下建立login.json

```json
// login.json
{
    "code":0,
    "msg":'成功',
    "data":{
        id:'123',
        token:'12345'
    }
}
```

之后访问localhost:1111/mock/login/login即可得到数据

## 启动

要启动服务，首先要在packge.json下scripts中添加指令

```js
"mock": "supervisor -w mock ./src/mocks/index.js"
```

之后只需要运行

```js
npm run mock
```

即可成功访问mock地址

访问http://localhost:1111/mock/login即可正确拿到定义在/mocks/apis/login/login中的数据

但是我们要同时打开web的时候就不得不重新打开一个终端，非常不方便，所以使用concurrently来同时启动两个服务

```js
"start": "concurrently \"yarn mock\" \"yarn serve\"",
```

之后只需要运行npm run start即可启动两个服务

## 配置

为了方便管理，推荐在/config/index.js中添加path并将其默认值设置为/mock，在生产环境设置为二级域名

```
// 开发环境默认配置
let _path = '/mock'; // 二级域名

// 服务端配置
if (process.env.NODE_ENV === 'production') {
  _path = '/xxx';
}
export const path = _path;
```

在调用接口的地方通过path来管理即可

## 完整代码

mocks结构:

│─ index.js
│─ map.js  
│─ apis

### index.js

```js
const http = require('http');
const url =require('url');
const Mock = require('mockjs');
const map = require('./map');
const port = 1111;
http
  .createServer((req, res) => {
    const pathname=url.parse(req.url).pathname;
    console.log(`url -> ${req.url}`);
    res.writeHead(200, {
      'Content-Type': 'application/json;charset=utf-8',
      'Access-Control-Allow-Origin': req.headers?(req.headers.origin||''):'',
      'Access-Control-Allow-Methods': '*',
      'Access-Control-Allow-Headers': '*',
      'Access-Control-Allow-Credentials': true,
      'Cache-Control': 'no-cache,no-store' // clear cache
    });
    let data = map[pathname];
    try {
      data=data?Mock.mock(JSON.parse(data)):'';
    } catch (error) {
      console.log(error);
    }finally{
      setTimeout(()=>{
        res.end(JSON.stringify(data)); 
      },Math.floor(Math.random()*2*1000));
    }
  })
  .listen(port);
console.log(`listening ${port}`);
```

### map.js

```js
const fs = require('fs');
const Path = require('path');
const path = '/mock';
const PUBLIC_PATH = Path.resolve(__dirname, './apis');

const apis={};
/**
 * 读取文件夹下所有文件映射
 * @param {String} dirPath 绝对路径
 * @param {String} simplePath 相对路径
 */
const readAllFile = (dirPath,simplePath='') => {
  const opt = { encoding: 'utf8' };
  let files = fs.readdirSync(dirPath, opt);
  // 文件轮询
  files.forEach(it => {
    if (!it) return;
    const childPath = `${dirPath}/${it}`;
    const stats = fs.lstatSync(childPath);
    if(stats){
      // 文件则根据路径添加，文件夹则控制路径并迭代
      if (stats.isFile()) {
        const name=it.slice(0,it.indexOf('.'));
        apis[`${path}/${simplePath}/${name}`] = fs.readFileSync(childPath, opt);
      } else {
        readAllFile(childPath,simplePath?`${simplePath}/${it}`:it);
      }
    }
  });
};
readAllFile(PUBLIC_PATH);

module.exports=apis;
```







