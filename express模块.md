## Express模块

### 一、express介绍

express是一个基于node.js平台的极简的、灵活的web应用开发框架。简单来说，express即为一个封装好的包，封装了很多功能。便于我们开发web应用。

### 二、express使用

### 三、路由

#### 3.1 什么是路由

官方定义：路由确定了应用程序如何响应客户端对特定端点的请求

#### 3.2 路由的使用

一个路由的组成有请求方法，路径和回调函数组成

express中提供了一系列方法，可以很方便的使用路由，使用格式如下：

```javascript
app.<method>(path,callback)
```

代码：

```javascript
//3.创建路由
app.get("/home", (req, res) => {
  res.end("hello,express");
});
```

#### 3.3 获取请求参数

express框架封装了一些API方法：

```javascript
//3.创建路由
app.get("/", (req, res) => {
  //   //原生操作
  //   console.log(req.method);
  //   console.log(req.url);
  //   let url = new URL(req.url, "http://127.0.0.1");
  //   console.log(url.searchParams);

  //express操作
  console.log(req.path);
  console.log(req.query);
  console.log(req.ip);
  res.setHeader("content-type", "text/html;charset=utf-8");
  res.end("首页");
});
```

#### 3.4 获取路由参数

路由参数指的是==url路径中的参数==，而不是请求参数

```javascript
//3.创建路由
app.get("/search/:name", (req, res) => {
  console.log(req.params.name);
  const name = req.params.name;
  res.end(`hello,${name}`);
});
```

#### 3.5 路由参数练习

根据路由参数响应歌手信息：

路径结构如下：/singer/1.html

### 四、express响应设置

express框架封装了一些API来方便给客户端响应数据，并且兼容原生HTTP模块的获取方式。

#### 一般响应：

```javascript
app.get("/response", (req, res) => {
  //express响应
  res.status(200);
  res.set("aaa", "bbb");
  res.send("你好，express");
});

app.listen(3000, () => {
  console.log("服务器已经启动...");
});
```

#### 其他响应：

```javascript
app.get("/response", (req, res) => {
  //express响应
  //重定向
  res.redirect("http://www.baidu.com");
  //下载响应
  res.download(__dirname + "/JSON/singers.json");
  //JSON响应
  res.json({
    name: "atguigu",
  });
  //响应文件内容
  res.sendFile(__dirname + "/HTTP模块/page/index.html");
});
```

### 五、express中间件

#### 5.1什么是中间件

中间件（Middle Ware）本质上是一个回调函数；

==中间件函数==可以像路由回调一样访问请求对象、响应对象

#### 5.2 中间件的作用

使用函数封装公共操作，简化代码

#### 5.3 中间件的类型

- 全局中间件
- 路由中间件

##### 5.3.1 定义全局中间件

每一个请求到达服务端之后都会执行全局中间件函数；

相当于==拦截器==

```javascript
// 记录下每个请求的url与ip地址

const express = require("express");
const fs = require("fs");
const app = express();
const ws = fs.createWriteStream("./access.log");
//1.定义中间件
function recordMiddleware(req, res, next) {
  const { url, ip } = req;
  ws.write(`${url}----${ip}`);
  next();
}

function processMiddleware(req, res, next) {
  res.set("content-type", "text/html;charset=utf-8");
  next();
}
//2.使用中间件
app.use(recordMiddleware, processMiddleware);
// app.use(processMiddleware);

app.get("/home", (req, res) => {
  //普通处理
  //   const { url, ip } = req;
  //   ws.write(url + "/r/n");
  //   ws.write(ip);
  res.end("home页");
});

app.get("/admin", (req, res) => {
  res.end("后台管理页");
});

app.listen(3000, () => {
  console.log("服务器已经启动...");
});
```

##### 5.3.2 路由中间件实践

==注意路由中间件的位置==

```javascript
const checkCodeMiddleware = (req, res, next) => {
  if (req.query.code === "521") {
    //放行
    next();
  } else {
    res.end("参数错误");
  }
};
//2.使用中间件
app.use(recordMiddleware);
app.use(processMiddleware);
app.get("/home", checkCodeMiddleware, (req, res) => {
  //普通处理
  //   const { url, ip } = req;
  //   ws.write(url + "/r/n");
  //   ws.write(ip);
  res.end("home页");
});
app.get("/admin", checkCodeMiddleware, (req, res) => {
  //   //请求url需要有code参数
  //   if (req.query.code === "521") {
  //     res.end("后台管理页");
  //   } else {
  //     res.end("参数错误");
  //   }
  res.end("后台管理页");
});

app.listen(3000, () => {
  console.log("服务器已经启动...");
});
```

##### 5.3.3 静态资源中间件实践

网站根目录：当客户端向服务器发送请求后，服务器向哪个文件夹寻找文件，则该文件便被称为静态资源目录/网站根目录。

该中间件：根据请求的文件资源url直接定位到文件的位置。如：127.0.0.1:3000/index.html则自动向静态资源目录下找到index.html；

127.0.0.1:3000/images/1.png，则向静态资源目录下找到images/1.png

###### 一些问题：

- 静态资源中间件的使用，使得index.html为默认打开的资源，
- 如果静态资源与路由规则同时匹配，谁先匹配谁就响应：use与get的响应顺序在于谁先声明
- 路由响应动态资源，静态资源中间件响应静态资源

#### 5.4 获取==请求体==数据

传统方法：

```javascript
const http = require("http");

const server = http.createServer((request, response) => {
  //1.声明变量
  let body = "";
  //2.request本身是一个可读流对象
  request.on("data", (chunk) => {
    //chunk虽然是个buffer，但在加运算时会自动转为string
    body += chunk;
  });
  request.on("end", () => {
    console.log(body);
    response.end("hello,http");
  });
});

//3.
server.listen(9000, () => {
  console.log("服务器已经启动...");
});
```

采用==body-parser==：

####  5.5 防盗链

**防盗链**，就是防有人盗用你的链接。别人在他的网站上引用了你的资源(图片,音频)，这样就会浪费你的流量，资源被引用的多了起来，你这边的[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)可能就扛不住挂了，你说这是多么悲哀的事情！

####  5.6 防盗链实践

==referer与host不一致==：表明请求资源的地址与服务器资源不同源所造成

```javascript
let url = new URL(referer);
console.log(url);
```

url是一个对象：

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231219215329288.png" alt="image-20231219215329288" style="zoom:50%;" />



```javascript
const express = require("express");

const app = express();

//只允许127.0.0.1访问图片
//声明中间件
app.use((req, res, next) => {
  let referer = req.get("referer");
  console.log(referer);
  if (referer) {
    let url = new URL(referer);
    // console.log(url);
    if (url.hostname !== "127.0.0.1") {
      console.log("1");
      res.status(403).send("<h1>forbidden</h1>");
      res.statusMessage = "forbidden";
      return;
    }
  }
  next();
});
app.use(express.static(__dirname + "/public"));
app.get("/home", (req, res) => {
  req.end("home页");
});
app.listen(9000, () => {
  console.log("服务器已经启动...");
});
```

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231219220338363.png" alt="image-20231219220338363" style="zoom:50%;" />

![image-20231219220404360](C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231219220404360.png)

#### 路由模块化

外层不写路径：

使用代码：

```javascript
const express = require("express");
import fontRouter from "./12_routes/fontRouter";
//2
const action = express();

action.use(fontRouter);
```

配置代码：

```javascript
const express = require("express");

//2.创建路由对象
const fontRouter = express.Router();

router.get("/home", (req, res) => {
  res.end("home页面");
});

router.get("/search", (req, res) => {
  res.end("内容搜索");
});

export default fontRouter;
```



### 七、模板引擎EJS

#### 7.1什么是模板引擎

模板引擎是分离用户界面和业务数据的一种技术                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                

把js代码与html结构分开

#### 7.2模板引擎ejs使用

##### 1.初体验

```javascript
const ejs = require("ejs");
const fs = require("fs");

let name = "lqy";
let age = 23;

let res = ejs.render(fs.readFileSync("./01_index.html").toString(), {
  name,
  age,
});
console.log(res);
```

##### 2.列表渲染

```javascript
const ejs = require("ejs");
const fs = require("fs");
const names = ["lily", "lhxu", "liuqy", "Alice"];

let res = ejs.render(
  `
    <h2>Names</h2>
    <ul>
      <% names.forEach((item)=>{ %>
      <li><%= item %></li>
      <% }) %>
    </ul>
`,
  { names: names }
);

console.log(res);
```

##### 3.条件渲染

```javascript
const ejs = require("ejs");

const isLogin = false;

let res = ejs.render(
  `
    <% if(isLogin){ %>
    <span>欢迎回来</span>
    <% }else{ %>
    <span>请登录</span>
    <% } %>`,
  { isLogin: isLogin }
);
console.log(res);
```

#### 7.3 express中使用ejs

```javascript
const express = require("express");

const app = express();
//设置模板引擎
app.set("view engine", "ejs");
//设置模板文件存放位置
app.set("views", __dirname + "/views");

//设置路由
app.get("/home", (req, res) => {
  let title = "你好,lqy";
  //3.render响应,模板文件名，数据两个参数
  res.render("home", { title });
  //4.创建模板文件
});

app.listen("3000", () => {
  console.log("服务器已经启动...");
});
```

#### 7.3 express-generator生成器

相当于脚手架创建服务器项目

```javascript
//1.下载
npm install -g express-generator
//-h选项显示命令选项
express -h
//创建名为myapp的Express应用程序并将视图引擎设置为ejs
express -e myapp

```



#### 7.4 文件上传

##### 查看文件上传报文

form表单文件上传时必须加上*enctype="multipart/form-data"*

==enctype就是encodetype就是编码类型的意思==。

multipart/form-data是指表单数据有多部分构成，既有文本数据，又有文件等二进制数据的意思。

需要注意的是：默认情况下，enctype的值是application/x-www-form-urlencoded，不能用于文件上传，只有使用了multipart/form-data，才能完整的传递文件数据。


##### 处理文件上传

插件：formidable插件

