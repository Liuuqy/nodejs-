### 相对路径问题

fs模块对资源进行操作时，路径的写法有两种

- 相对路径

  1. ./座右铭.txt 当前目录下的座右铭;
  2. 座右铭.txt 同1；
  3. ../座右铭.txt 上一级目录下的座右铭

- 绝对路径

  1. D:/ProgramFiles windows下的绝对路径

  2. /usr/bin linux系统下的绝对路径

### 相对路径注意点

**fs模块中的相对路径参照物：命令行的工作目录**

```javascript
const fs = require("fs");
fs.writeFile("./index.txt", "test", (err) => {
  if (err) {
    console.log(err);
    return;
  }
  console.log("操作成功");
});
```

- 在运行环境1：

  ![image-20231217160136637](C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231217160136637.png)

index.txt将保存在nodejs文件夹下

- 在运行环境2：

  ![image-20231217160232908](C:\Users\liuuq\AppData\Roaming\Typora\typora-user-images\image-20231217160232908.png)

index.txt将保存在代码文件夹下

解决方法：引入__dirname

### __dirname

__dirname:所在文件的所在目录的绝对路径