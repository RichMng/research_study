## 搭建swagger UI 服务

[Swagger UI](https://swagger.io/tools/swagger-ui/)允许任何人（无论是您的开发团队还是最终用户）在没有任何实现逻辑的情况下可视化并与API的资源交互。它是根据OpenAPI（以前称为Swagger）规范自动生成的，可视化文档使后端实现和客户端使用变得容易。

### 下载 swagger UI
``` shell
# 在你喜欢的目录下
git clone https://github.com/swagger-api/swagger-ui.git
```

### 创建 swagger UI 主目录

比如我在 `/root/work` 目录下创建 `swagger-doc`文件夹，然后进入到 `swagger-doc`目录 执行初始化命令
```shell
npm init 
```
执行后出现引导配置项，可以什么都不写，回车即可，执行结束之后在当前目录下生成一个`package.json`文件
``` json
{
  "name": "swagger-doc",
  "version": "1.0.0",
  "description": "",
  "main": "",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```
把刚刚clone下来的swagger-ui项目里面的`dist`文件拷贝到我们创建的`swagger-doc`目录下面
```shell

swagger-doc
        |__ dist
        |__ package.json

```

### 安装 express
[express](http://www.expressjs.com.cn/)是一个基于 Node.js 平台，快速、开放、极简的 Web 开发框架

还是在当`swagger-doc`目录下执行
```shell
npm install express
```
安装成功后目录下多了几个文件
```shell
swagger-doc
        |__ dist
        |__ node_modules
        |__ package.json
        |__ package-lock.json
```
### 添加 `index.js`
还是在`swagger-doc`目录下面创建`index.js`
``` shell
touch index.js
```
现在的目录结果为
```shell
swagger-doc
        |__ dist
        |__ index.js
        |__ node_modules
        |__ package.json
        |__ package-lock.json
```
创建后并在`index.js`中添加如下内容
```javascript
// 此文件作为服务的启动文件并渲染刚刚拷贝进来的dist静态文件

var express = require('express');
var app = express();
// /docs 可以改成任意字符串
app.use('/docs', express.static('dist'));
app.get('/', function (req, res) {
      res.send('Hello World!');

});
// 服务端口
app.listen(3000, function () {
      console.log('Example app listening on port 3000!');

});
```

### 启动服务

保存后我们可以使用
```shell
node index.js
# Example app listening on port 3000!
```
服务启动成功后访问 `http://10.1.108.51:3000/docs/index.html`（此IP地址是我本地虚拟机IP地址）</br>
此时看到页面是demo的API文档</br>
接下来我们把官方文档替换成我们项目的文档

### 显示项目文档

`swagger UI`显示文档本质上就是就是加载符合其语法的`json`文件, 有大概两种加载的方案

+ 从本地加载, 加载本地json文件
+ 从网络加载, 加载接口返回的json

很显然第二种方案更符合我们的需求
下面我们更改`dist/index.html`

```diff
<!-- HTML for static distribution bundle build -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Swagger UI</title>
    <link rel="stylesheet" type="text/css" href="./swagger-ui.css" >
    <link rel="icon" type="image/png" href="./favicon-32x32.png" sizes="32x32" />
    <link rel="icon" type="image/png" href="./favicon-16x16.png" sizes="16x16" />
    <style>
      html
      {
        box-sizing: border-box;
        overflow: -moz-scrollbars-vertical;
        overflow-y: scroll;
      }

      *,
      *:before,
      *:after
      {
        box-sizing: inherit;
      }

      body
      {
        margin:0;
        background: #fafafa;
      }
    </style>
  </head>

  <body>
    <div id="swagger-ui"></div>

    <script src="./swagger-ui-bundle.js"> </script>
    <script src="./swagger-ui-standalone-preset.js"> </script>
    <script>
    window.onload = function() {
      // Begin Swagger UI call region
      const ui = SwaggerUIBundle({
-        url: "https://petstore.swagger.io/v2/swagger.json",
+        url: "https://10.1.108.51:8080/docs.json", // 我们项目API文档url
        dom_id: '#swagger-ui',
        deepLinking: true,
        presets: [
          SwaggerUIBundle.presets.apis,
          SwaggerUIStandalonePreset
        ],
        plugins: [
          SwaggerUIBundle.plugins.DownloadUrl
        ],
        layout: "StandaloneLayout"
      })
      // End Swagger UI call region

      window.ui = ui
    }
  </script>
  </body>
</html>

```
保存后重启启动`express`服务 重新访问`http://10.1.108.51:3000/docs/index.html` 可以看到我们自己的的API文档可以显示了
