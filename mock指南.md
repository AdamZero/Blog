# Mock指南

## 前言

由于前后端工作的不同步，在实际工作过程中，经常会发生前端在写界面逻辑的时候无法从后端获取到相应数据的情况，如果把数据直接写死在相应界面，一来污染界面逻辑，二来不方便管理数据，所以mock应运而生，可以帮我们很方便的进行数据模拟

## 依赖

```js
npm install mockjs -D
npm install supervisor -D
npm install concurrent -D
```



### http

http可以帮我们启动服务挂载访问接口结果



```js
// /mocks/index.js
const http = require('http');
const mock = require('mockjs');

const map = require('./map');
const port = 1111;
http
  .createServer((req, res) => {
    console.log(`url -> ${req.url}`);
    
    res.writeHead(200, {
      'Content-Type': 'application/json;charset=utf-8',
      'Access-Control-Allow-Origin': req.headers?(req.headers.origin||''):'',
      'Access-Control-Allow-Methods': '*',
      'Access-Control-Allow-Headers': '*',
      'Access-Control-Allow-Credentials': true,
      'Cache-Control': 'no-cache,no-store' // clear cache
    });
    const data = map[req.url] ? mock(map[req.url]) : '';
    setTimeout(()=>{
      res.end(JSON.stringify(data)); 
    },Math.floor(Math.random()*5*1000));
  })
  .listen(port);
console.log(`listening ${port}`);

```

### mockjs

mockjs可以动态渲染模拟数据,详细使用说明请查看[官网](http://mockjs.com/)

### supervisor

supervisor在mockjs中引用文件被修改时会自动更新服务

### concurrently

可以同时运行多个命令，非必须

## 添加mock接口

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

