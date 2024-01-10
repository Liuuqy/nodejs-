## MongoDB

### 一、简介

#### 1.1MongoDB是什么

基于分布式文件存储的==数据库==。

#### 1.2数据库是什么



#### 1.3数据库的作用

管理数据，对数据进行增删改查

#### 1.4数据库管理数据特点

相比于纯文件管理数据、数据库管理数据有如下特点：

1.速度更快；

2.扩展性更强；

3.安全性更强；

#### 1.5为什么选择MongoDB

操作语法与js类似，容易上手，学习成本低

### 二、核心概念

#### 2.1 核心介绍

- 数据库：一个数据仓库；数据库下可有创建很大数据库，数据库中可以存放很多集合；
- 集合：集合类似于js中的数组，在集合中可以存放很多文档
- 文档：**文档是数据库中最小的单位，类似于js中的对象**

通过JSON文件来理解MongoDB中的概念：

- 一个JSON文件好比是一个数据库，一个MongoDB服务下可以有n个数据库；
- JSON文件中的一级属性的数组值好比是集合
- ==数组中的对象好比是文档；==
- 对象中的属性有时也称为表，==字段== 

### 四、命令行交互

#### 4.1 数据库命令

1.显示所有的数据库

```mongoDB
show dbs
```

2.切换到指定数据库，如果数据库不存在则会自动创建数据库

```
use databaseName
```

3.显示当前数据库

```
db
```

4.删除某个数据库

```
use 某个数据库
db.dropDatabase()
```

#### 4.2 集合命令

1.创建集合

```
db.createCollections('集合名称')
```

2.显示当前数据库中的所有集合

```
show collections
```

3.集合重命名

```
db.集合名字.renameCollection(newName)
```

4.删除集合

```
db.集合.drop()
```

#### 4.3 文档命令

1.插入文档

```
db.集合.insert(document)
```

2.查询文档

```
db.集合.find(查询条件)
> db.users.find()
{ "_id" : ObjectId("65829dc9d2e0787d6e5a7857"), "name" : "hcc", "age" : 18 }
{ "_id" : ObjectId("65829dded2e0787d6e5a7858"), "name" : "hnp", "age" : 19 }
{ "_id" : ObjectId("65829de8d2e0787d6e5a7859"), "age" : 30 }
//查询条件
> db.users.find({age:20})
{ "_id" : ObjectId("65829de8d2e0787d6e5a7859"), "name" : "zs", "age" : 20 }
```

3.更新文档

```
db.集合.update(查询条件,{$set:更新信息})
```

4.删除文档

```
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

- **query** :（可选）删除的文档的条件。
- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- **writeConcern** :（可选）抛出异常的级别。

### 五、Mongoose

#### 5.1 介绍

Mongoose是一个对象文档模型库：是一个工具包，

#### 5.2 作用

使用代码操作数据库

#### 5.3 Mongoose连接数据库

```javascript
//1.安装Mongoose
//2.导入Mongoose
const mongoose = require("mongoose");

//3.连接mongodb
//协议+主机端口号+数据库名称
mongoose.connect("mongodb://127.0.0.1:27017/users");

//4.设置回调,绑定事件
//连接成功
mongoose.connection.on("open", () => {
  console.log("数据库连接成功");
});
//连接失败
mongoose.connection.on("error", () => {
  console.log("连接失败");
});
//连接关闭
mongoose.connection.on("close", () => {
  console.log("连接关闭");
});

//关闭mangoDB的连接
setTimeout(() => {
  mongoose.disconnect();
}, 3000);
```

更新：

```javascript
//建议使用once,数据回调函数只执行一次
//比如当mongoDB突然断开又恢复时，on会再次执行，once不会再执行
mongoose.connection.once("open", () => {
  console.log("数据库连接成功");
});
```

#### 5.4 mongoose增加文档

```javascript
//1.安装Mongoose
//2.导入Mongoose
const mongoose = require("mongoose");

//3.连接mongodb
//协议+主机端口号+数据库名称
mongoose.connect("mongodb://127.0.0.1:27017/users");

mongoose.connection.once("open", () => {
  console.log("数据库连接成功");
  //设置集合中文档的属性以及属性的类型
  //5.创建文档的结构对象
  let UserSchema = new mongoose.Schema({
    name: String,
    age: Number,
    price: Number,
  });
  //6.创建模型对象：对文档操作的封装对象，即完成文档的增删改查，第一个参数为需要操作的集合名称，第二个参数为结构对象
  let UserModel = mongoose.model("users", UserSchema);

  //7.新增
  UserModel.create({
    name: "lqy",
    age: 23,
    price: 100,
  }).then((err, data) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log(data);
  });
});
//连接失败
mongoose.connection.on("error", () => {
  console.log("连接失败");
});
//连接关闭
mongoose.connection.on("close", () => {
  console.log("连接关闭");
});
```

#### 5.5 字段类型---文档属性值的类型

String，Number，Boolean，Array，Date，Buffer，

Mixed（任意类型，需要使用mongoose.Schema.Types.Mixed指定）

ObjectId(对象ID，需要使用mongoose.Schema.Types.ObjectId指定)

Decimal128(高精度数字，需要使用mongoose.Schema.Types.Decimal128指定)

#### 5.6 字段值验证

Mongoose有一些内建验证器，可以对字段值进行验证

##### 1.必填项

```javascript
let BooksSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true, //表明属性必须不为空
  },
  price: Number,
});
```

##### 2.默认值

```
let BooksSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true, //表明属性必须不为空
    default:'value',//默认值
  },
  price: Number,
});
```

##### 3.enum可迭代对象

表明了选择类型:

```javascript
let BooksSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true, //表明属性必须不为空
  },
  price: Number,
  style: {
    type: String,
    enum: ["言情", "城市", "鬼怪", "恐怖"],
  },
});
```

##### 4.唯一值

```javascript
let BooksSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true, //表明属性必须不为空
    unique: true, //表明该属性不能重复
  },
  price: Number,
  style: {
    type: String,
    enum: ["言情", "城市", "鬼怪", "恐怖"],
  },
});
```

 

### 条件控制

#### 1.运算符

- ＞ 使用 $gt（great than)
- < 使用$lt
- ≥ $gte
- !== $ne
- <= 使用 $lte
- === {price:20}

#### 2.逻辑运算

$or逻辑或：

```javascript
NovelModel.find({ $or: [{ price: 999 }, { price: 35 }] }).then((res, err) => {
    console.log(res);
});
```

$and逻辑与:

```javascript
NovelModel.find({
    $and: [{ price: { $gt: 35 } }, { price: { $lt: 9999 } }],
}).then((res, err) => {
    console.log(res);
});
```

#### 3.正则匹配

### Mongoose个性化读取

#### 1.字段筛选

仅展示部分属性，展示的属性为1

```javascript
  NovelModel.find()
    .select({ name: 1, price: 1, _id: 0 })
    .then((res, err) => {
      console.log(res);
      console.log(err);
    });
```

#### 2.数据排序

```javascript
NovelModel.find()
    .sort({ price: 1 })//price属性升序排列
    .then((res, err) => {
      console.log(res);
     });
```

#### 3.数据截断

输出前几位

```
limit(3)//输出前三位
```

输出特定几位到几位

```
skip(n).limit(m):输出n+1到n+1+m位
```

