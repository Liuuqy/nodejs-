## HTTP模块

### 创建HTTP服务

```javascript
const http = require("http");

//2.创建服务对象
//回调函数的执行时间：服务器接收到请求时便会执行
const server = http.createServer((request, response) => {
  response.end("Hello HTTP Server");
});

//3.监听端口，启动服务器，执行时间
server.listen(9000, () => {
  console.log("服务已经启动...");
});
```

访问：

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231218103757784.png" alt="image-20231218103757784" style="zoom:50%;" />

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                

### HTTP服务的注意事项

- ctrl+c停止服务
- 服务器启动后，更新代码必须重启服务器才能生效；
- 响应内容 中文乱码的解决方法：

```javascript
response.setHeader("content-type", "text/html;charset=utf-8")
```

- 端口被占用：换端口；
- http协议默认端口：80；https协议默认端口：443

如果端口被其他程序占用，可以使用==资源监视器==找到占用断口的程序，并关停该程序

### 浏览器查看HTTP报文

略

### 提取HTTP请求报文

分析请求报文内容，返回正确结果：

**请求头部分内容**：

![image-20231218111247976](C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231218111247976.png)

**请求体内容：**

==request本身是一个可读取流对象==

```javascript
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
```

**获取请求路径与查询字符串：**

```javascript
const url = require("url");
const server = http.createServer((request, response) => {
  //虽然url中有，但是使用起来较为不方便
  //   const url = request.url;
  //   console.log(url);
  const res = url.parse(request.url);
  console.log(res);
  response.end("hello,server");
});
```

解析如下：

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231218112951809.png" alt="image-20231218112951809" style="zoom:50%;" />

**获取参数：**

方式1：

```javascript
const url = require('url')
const server = http.createServer((request, response) => {
  //虽然url中有，但是使用起来较为不方便
  //   const url = request.url;
  //   console.log(url);
  const res = url.parse(request.url, true);
  console.log(res);
  response.end("hello,server");
});
```

查询结果，此时便将查询参数转换为了对象的形式

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231218113223881.png" alt="image-20231218113223881" style="zoom:50%;" />

方式2：

```javascript
const http = require("http");
const url = require("url");
const server = http.createServer((request, response) => {
  let url = new URL(request.url, "http://127.0.0.1:9000");
  console.log(url);
  response.end("hello,server");
});

server.listen(9000, () => {
  console.log("服务器已经启动了...");
});
```

结果为：

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231218113652582.png" alt="image-20231218113652582" style="zoom:50%;" />

```javascript
const http = require("http");
const url = require("url");
const server = http.createServer((request, response) => {
  let url = new URL(request.url, "http://127.0.0.1:9000");
  console.log(url.pathname);
  console.log(url.searchParams.get("key"));
  response.end("hello,server");
});

server.listen(9000, () => {
  console.log("服务器已经启动了...");
});
```

解析：

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231218113921979.png" alt="image-20231218113921979" style="zoom:50%;" />

### HTTP请求练习

### 设置HTTP响应报文

必须要有response.end()函数，且只能有一个

### 网页资源加载全过程

### HTTP响应练习拓展

将各种资源区分开时，每种资源都会对应发送一个请求，因此要正确处理响应内容：

```javascript
const server = http.createServer((request, response) => {
  const { pathname: path } = new URL(request.url, "http://127.0.0.1");
  if (path === "/") {
    const data = fs.readFileSync("./资源/06-index.html");
    response.write(data);
  } else if (path === "/index.js") {
    const data = fs.readFileSync("./资源/index.js");
    response.write(data);
  } else if (path === "/index.css") {
    const data = fs.readFileSync("./资源/index.css");
    response.write(data);
  } else {
    response.write(`<h2>404 Not Found</h2>`);
  }
  response.end();
});
```

### 静态资源与动态资源

静态资源：==内容长时间不发生改变==的资源，例如图片，视频，css文件，js文件，HTML文件，字体文件等；

动态资源：==内容经常更新的资源==

### 搭建静态资源服务

避免之前练习中的if-else情况判断，合理处理文件请求url与文件请求位置，从而统一响应

### 网站根目录/静态资源目录

HTTP服务在==哪个文件夹中寻找静态资源，哪个文件夹就是静态资源目录==，也称之为网站根目录

例如上个练习中的page文件夹即为静态资源目录

### 网页URL地址

#### 绝对路径：

| 形式                   | 特点                                                         |
| ---------------------- | ------------------------------------------------------------ |
| http://atguigu.com/web | 直接向目标资源发送请求，容易理解。外链多用此形式             |
| //atguigu.com.web      | 与页面URL的协议拼接形成完整URL再发送请求。大型网站用的比较多，如果http不对，状态码将提示重定向3xx，此时响应头中有新的locationm |
| /web                   | 与页面URL的==协议+主机名+端口==拼接形成完整url               |

==绝对路径使用较为广泛==：因为主机名有可能改变

#### 相对路径：

假如当前网页url为：http://www.atgyuigu.com/courses/h5.html

| 形式                   | 最终的url                                           |
| ---------------------- | --------------------------------------------------- |
| ./css/app.css          | http://www.atgyuigu.com/courses/h5.html/css/app.css |
| js/app.js：注意不带/   | http://www.atgyuigu.com/courses/h5.html/js/app.js   |
| ../img/logo.png        | http://www.atgyuigu.com/img/logo.png                |
| **../../mp4/show.mp4** | **http://www.atgyuigu.com/mp4/show.mp4**            |

==根目录下不能回退==，因此第四个需要注意

### 网页url的场景小结

包括但不限于如下场景：

- a标签href
- link标签 href
- script标签 src
- img标签 src
- form中的action
- AJAX请求中的URL

浏览器在解析html文件时，遇到外部标签，要与当前浏览器下的地址进行拼接

### 设置资源类型（mime类型）

媒体类型（通常称为Multipurpose Internet Mail Extensions或MIME类型）是一种标准，用来表示文档、文件或字节流的性质和格式。

mime类型结构：[type]/[subType]

HTTP服务可以设置响应头Content-Type来表明响应体的MIME类型，浏览器会根据该类型决定如何处理资源：

```
html:'text/html',
css:'text/css',
js:'text/javascript',
png:'image/png',
jpg:'image/jpeg',
gif:'image/gif',
mp4:'video/mp4',
mp3:'audio/mp3',
```

根据请求文件的后缀，设置响应内容类型：

```javascript
const http = require("http");
const fs = require("fs");
const path1 = require("path");
const mimes = {
  html: "text/html",
  css: "text/css",
  js: "text/javascript",
  png: "image/png",
  jpg: "image/jpeg",
  gif: "image/gif",
  mp4: "video/mp4",
  mp3: "audio/mp3",
};
const server = http.createServer((request, response) => {
  const { pathname: path } = new URL(request.url, "http://127.0.0.1");
  //拼接文件路径
  let root = __dirname + "/page";
  let filePath = root + path;
  fs.readFile(filePath, (err, data) => {
    if (err) {
      console.log(err);
      response.setHeader("content-type", 'text/html;charset="utf-8"');
      response.end("文件读取失败");
      return;
    }
    //1.获取文件后缀
    const ext = path1.extname(filePath).slice(1);
    //2.获取文件后缀对应的类型
    const type = mimes[ext];
    if (type) {
      //匹配到了type
      if (ext === "html") {
        response.setHeader("content-type", type + ';charset="utf-8"');
      } else {
        response.setHeader("content-type", type);
      }
    } else {
      response.setHeader("content-type", "application/octet-stream");
    }
    console.log(type);
    response.end(data);
  });
});

server.listen(9000, () => {
  console.log("服务器已启动...");
});
```

对于未知的资源类型，可以选择==application/octet-stream类型==，浏览器在遇到该类型的响应时，会对响应体内容进行独立存储，即我们常见的下载结果

设置内容类型不是必须的，因为浏览器会自动根据返回内容判断其类型，但是会让我们的开发更为规范

### 解决乱码问题

需要添加编码格式，不然中文会乱码：

但是在请求html文件时，不会乱码，因为html文档里有个meta标签，将charset设置为了utf-8：

```html
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <link rel="stylesheet" type="text/css" href="./index.css" />
  </head>
```

但是content-type的优先级高于meta标签；

在开发时，其实只要设置index.html标签的中文集，附加的css与js等资源将与网页的字符集一致

### 完善错误处理

```javascript
const server = http.createServer((request, response) => {
  const { pathname: path } = new URL(request.url, "http://127.0.0.1");
  //拼接文件路径
  let root = __dirname + "/page";
  let filePath = root + path;
  fs.readFile(filePath, (err, data) => {
    if (err) {
      console.log(err);
      response.setHeader("content-type", 'text/html;charset="utf-8"');
      switch (err.code) {
        default:
          response.statusCode = 500;
          response.end("<h1>Internal Server Error</h1>");
        case "ENOENT":
          response.statusCode = 404;
          response.statusMessage = "not found";
          response.end(`<h1>${response.statusMessage}</h1>`);
        case "EPERM":
          response.statusCode = 403;
          response.statusMessage = "Not forbidden";
      }
      return;
    }
    //1.获取文件后缀
    const ext = path1.extname(filePath).slice(1);
    //2.获取文件后缀对应的类型
    const type = mimes[ext];
    if (type) {
      //匹配到了type
      if (ext === "html") {
        response.setHeader("content-type", type + ';charset="utf-8"');
      } else {
        response.setHeader("content-type", type);
      }
    } else {
      response.setHeader("content-type", "application/octet-stream");
    }
    console.log(type);
    response.end(data);
  });
});
```

### GET和POST请求

#### 1.场景小结

GET请求的情况：

- 在地址栏输入url访问；
- 点击a链接；
- link标签引入css；
- script标签引入js；
- img标签引入图片；
- ajax中的get请求；
- form标签中的method为get

POST请求的情况：

- form标签中的method为post；
- ajax的post请求

#### 2.区别

1. 作用：get主要用来获取数据，post用来提交数据；
2. 参数位置：get参数将其附在url之后，post请求参数是将参数放入请求体中；
3. 安全性：post相比get更安全，因为get暴露参数；
4. get请求大小有限制，一般为2K，而post请求没有大小限制