## fs模块（file system）

fs模块可以实现与硬盘的交互，例如文件的创建、删除、重命名、移动，还有文件内容的写入、读取，以及文件夹的相关操作。

### 写入

- fs.writeFile('文件地址','content','处理异常的回调函数')

### fs模块_同步与异步

```javascript
// 需求：新建一个文件，座右铭.txt，写入内容：三人行必有我师焉
//1.导入fs模块
const fs = require("fs");

//2.写入文件
fs.writeFile("座右铭.txt", "三人行，必有我师焉", (err) => {
  if (err) {
    console.log("写入失败");
    return;
  }
  console.log("写入成功");
});

console.log(1 + 1);
```

注意执行顺序：fs.writeFile为异步执行的函数

- 同步执行：

```javascript
//同步写入
fs.writeFileSync("./data.txt", "test");
```

### 追加写入

- fs.appendFile()
- fs.appendFileSync()

```javascript
const fs = require("fs");

fs.appendFile("座右铭.txt", "则其善者而从之", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("写入成功");
});

fs.appendFileSync("座右铭.txt", "\n温故而知新,可以为师矣");
```

- fs.writeFile实现追加写入

  ```javascript
  fs.writeFile("座右铭.txt", "texttexttext", { flag: "a" }, (err) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log("写入成功");
  });
  ```

### fs的流式写入

为什么需要流式写入：

程序打开一个文件是需要消耗资源的，流式写入可以**减少打开关闭文件的次数**。流式写入方式适用于==大文件写入或者频繁写入的场景==，writeFile适合写入频次较低的场景。

```javascript
const fs = require("fs");
//2.创建写入流对象
const ws = fs.createWriteStream("./观书有感.txt");
//3.write
ws.write("半亩方塘一鉴开\r\n");
ws.write("天光云影共徘徊\r\n");
//4.关闭通道
ws.close();
```

###  写入文件的场景

文件写入在计算机中是一个非常常见的操作，如：

- 下载文件
- 安装软件
- 保存程序日记，如Git
- 视频录制
- 编辑器保存文件

当需要持久化保存数据的时候，应该想到文件写入

### 文件读取

#### 1.异步读取

```
fs.readFile("./观书有感.txt", (err, data) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("读取成功:", data);
  console.log("内容为:", data.toString());
});
console.log("1");
```

#### 2.同步读取

```
const data = fs.readFileSync("./观书有感.txt");
console.log(data.toString());
```

#### 3.读取文件应用场景

- 程序运行
- 查看聊天记录
- 查看图片
- 查看视频
- 上传文件
- 编辑器打开文件

#### 4.文件流式读取

查看代码发现，==流式读取每次读取64KB==

##### 复制文件

原文件读取的数据放入副本文件：

```javascript
//复制文件video1.mp4
const fs = require("fs");
//方式1-readFile
let data = fs.readFileSync("../video1.mp4");
fs.writeFileSync("./video1copy1.mp4", data);

//方式2-流式写入与读取
const rs = fs.createReadStream("../video1.mp4");
const ws = fs.createWriteStream("./Video1copy2.mp4");
rs.on("data", (chunk) => {
  ws.write(chunk);
});
rs.on("close", () => {
  console.log("读取完成");
});
```

- 流式读取占用空间更小

### 文件操作

#### 1.文件重命名与移动

使用rename函数

```javascript
//复制文件video1.mp4
const fs = require("fs");
//方式1-readFile
let data = fs.readFileSync("../video1.mp4");
fs.writeFileSync("./video1copy1.mp4", data);

//方式2-流式写入与读取
const rs = fs.createReadStream("../video1.mp4");
const ws = fs.createWriteStream("./Video1copy2.mp4");
rs.on("data", (chunk) => {
  ws.write(chunk);
});
rs.on("close", () => {
  console.log("读取完成");
});
```

- 注意，rename函数在移动时不会创建文件夹

#### 2.文件删除

使用unlink函数与rm函数

```javascript
const fs = require("fs");

//方式1
fs.unlink("./观书有感.txt", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("删除成功");
});

//方式2
fs.rm("./观书有感.txt", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("操作成功");
});
```

### 文件夹操作

#### 1.创建文件夹

```javascript
const fs = require("fs");
//1.普通创建
fs.mkdir("./a", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("创建成功");
});
//2.嵌套创建
fs.mkdir("./a/b/c", { recursive: true }, (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("创建成功");
});
```

#### 2.读取文件夹

```javascript
//3.读取
fs.readdir("./资料", (err, files) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(files);
});
```

#### 3.删除文件夹

```javascript
//删除文件夹
//删除文件夹
fs.rm("./a/b/c", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("删除成功");
});

fs.rmdir("./a", { recursive: true }, (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("删除成功");
});
```

### 查看资源状态

1.stat方法

```javascript
const fs = require("fs");

//1.stat方法，status
fs.stat("./video1.mp4", (err, stats) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log(stats);
});
```

<img src="C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231217154751098.png" alt="image-20231217154751098" style="zoom:50%;" />

size:文件大小；birthtimeMs：文件创建时间；atimeMs：文件最后访问时间；mtimeMs：文件最后修改时间；

2.获取文件类型

```javascript
//1.
fs.stat("./video1.mp4", (err, stats) => {
  if (err) {
    console.log(err);
    return;
  }
  //console.log(stats);
  console.log(stats.isFile());
  console.log(stats.isDirectory());
});
```

