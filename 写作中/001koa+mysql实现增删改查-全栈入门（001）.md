# koa+mysql实现增删改查-全栈之路（001）

> Date: 2020-4-23

以前很少写文章，从今天开始我要挑战一下自己，连续输出100篇技术类文章。这100篇文章我尽量以实战案例为主。

如果你觉得本文还不错，记得关注或者给个 star，你们的赞和 star 是我编写更多更精彩文章的动力！
[GitHub 地址](https://github.com/shixinglong007/shixinglong007.github.io)

## 本文重点内容
* 从 0 到 1 集成 node + mysql + ejs 用户管理系统
* 上手 sequelize 不使用sql操作数据库
* 熟悉 MVC 开发模式

## 成品演示
- [在线地址](http://demo_01.catok.top/)
### 关键技术点

- 1.1 数据库操作
- 1.2 MVC 模式是什么？
> 1.1 数据库操作

```javascript
// 使用 sequelize 代理数据库操作
const { Sequelize, Model, DataTypes } = require('sequelize');
const config = require('./config')

// 配置数据库连接
const sequelise = new Sequelize(
    dbName,
    username, password,
    {
        host: host,
        dialect: 'mysql', // 配置方言
})
class User extends Model {}

// 创建表
User.init({
  username: DataTypes.STRING,
  birthday: DataTypes.DATE
}, { sequelize, modelName: 'user' });

sequelize.sync() // 生成数据表
  .then(() => User.create({ // 插入数据
    username: 'janedoe',
    birthday: new Date(1980, 6, 20)
  }))
  .then(jane => {
    console.log(jane.toJSON());
  });
```

> 1.2 MVC模式是什么？

    MVC即Model、View、Controller即模型、视图、控制器

    Module     - 对象和业务逻辑
    View       - 用户界面
    Controller - 用来调度 View 和 Model

## 开始撸代码

### 第一步 初始化目录
先来初始化下目录结构：

    $ mkdir demo_001 && cd demo_001
    $ npm init -y
    $ npm i -s nodemon better-npm-run
    $ npm i -s koa koa-views @koa/router koa-bodyparser
    $ npm i -s ejs sequelize mysql2

> 各个库的版本号为：
@koa/router: 9.0.0, better-npm-run: 0.1.1，ejs: 3.0.2，koa: 2.11.0，koa-views：6.2.1，sequelize：5.21.6，koa-bodyparser：4.3.0，koa-static：5.0.0，mysql2：2.1.0，nodemon：2.0.3

添加 npm scripts 到 package.json：

```Json
    "scripts": {
      "start": "npm run dev",
      "dev": "better-npm-run dev",
      "prd": "better-npm-run prd"
    },
    "betterScripts": {
      "dev": {
        "command": "nodemon app.js",
        "env": {
          "NODE_ENV": "development"
        }
      },
      "prd": {
        "env": {
          "NODE_ENV": "production"
        },
        "command": "pm2 start app.js -n demo_001"
      }
    },
```
### 第二步 实现 view 层

新建 app.js 
```javascript
// app.js 代码
const Koa = require('koa');
const views = require('koa-views');
const path = require('path');
const bodyparser = require('koa-bodyparser');

const app = new Koa();

app.keys = ['my keys'];

app.use(bodyparser());
app.use(views(path.join(__dirname, './views'), { extension: 'ejs' }));

app.listen(3000, () => {
    console.log('server is running', new Date());
});
```
让代码跑起来，之后修改代码不用频繁的重启服务。因为开发环境是用 nodemon 托管的

    $ npm start

新建 views 目录结构
```
  demo_001
  ├── router
  │   └── index.js
  ├── views
  │   ├── index.ejs
  │   ├── header.ejs
  │   ├── create.ejs
  │   └── edit.ejs
  └── app.json
  └── package.json
```
view 层核心代码：

```html
<!-- views/header.ejs -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>node + mysql 实现增删改查</title>
</head>
<body>
```
```html
<!-- views/index.ejs -->
<% include('./header.ejs') %>
    <h1>
        <%= title %>
        <small>实现增删改查</small>
    </h1>
    <a href="/user/create">添加用户</a>
    <style>
        table{
            border-color: #ccc;
        }
        table td, th{
            background: #fff;
        }
    </style>
    <table cellspacing="1"  cellpadding="15" bgcolor="#000"  >
        <thead>
            <tr>
                <th>username</th>
                <th>pwd</th>
                <th>phone</th>
                <th>age</th>
                <th>gender</th>
                <th>操作</th>
            </tr>
        </thead>
        <% for (const user of users) { %>
            <tr>
                <td><%= user.username %></td>
                <td><%= user.pwd %></td>
                <td><%= user.phone %></td>
                <td><%= user.age %></td>
                <td><%= user.gender %></td>
                <td>
                    <a href="/user/edit?id=<%= user.id %>">修改</a>
                    <a href="/user/del/<%= user.id %>">删除</a>
                </td>
            </tr>
        <% } %>
    </table>
</body>
</html>
```
```html
<!-- views/create.ejs -->
<% include('./header.ejs') %>
    <style>
        label{
            width: 80px;
            display: inline-block;
            text-align: right;
            padding-right: 10px;
        }
    </style>
    <h1>
        <%= title %> <small><a href="/">返回首页</a></small>
    </h1>
    <form action="/user/create" method="POST" >
        <fieldset>
            <label>username</label>
            <input value="" name="username" />
        </fieldset>
        <fieldset>
            <label>pwd</label>
            <input value="" name="pwd" />
        </fieldset>
        <fieldset>
            <label>phone</label>
            <input value="" name="phone" />
        </fieldset>
        <fieldset>
            <label>age</label>
            <input value="" name="age" />
        </fieldset>
        <fieldset>
            <label>gender</label>
            <input value="" name="gender" />
        </fieldset>
        <fieldset>
            <button type="submit">Submit</button>
        </fieldset>
    </form>
</body>
</html>
```
```html
<!-- views/edit.ejs -->
<% include('./header.ejs') %>
    <style>
        label{
            width: 80px;
            display: inline-block;
            text-align: right;
            padding-right: 10px;
        }
    </style>
    <h1>
        <%= title %> <small><a href="/">返回首页</a></small>
    </h1>
    <form action="/user/edit" method="POST" >
        <input value="<%= user.id %>" name="id" type="hidden" />
        <fieldset>
            <label>username</label>
            <input value="<%= user.username %>" name="username" />
        </fieldset>
        <fieldset>
            <label>pwd</label>
            <input value="<%= user.pwd %>" name="pwd" />
        </fieldset>
        <fieldset>
            <label>phone</label>
            <input value="<%= user.phone %>" name="phone" />
        </fieldset>
        <fieldset>
            <label>age</label>
            <input value="<%= user.age %>" name="age" />
        </fieldset>
        <fieldset>
            <label>gender</label>
            <input value="<%= user.gender %>" name="gender" />
        </fieldset>
        <fieldset>
            <button type="submit">Submit</button>
        </fieldset>
    </form>
</body>
</html>
```
路由部分核心代码：
```javascript
const Router = require('@koa/router');
const router = new Router()

// 首页，查询所有用户
router.get('/', async ctx => {
    let users = []
    console.log('查询所有用户')
    await ctx.render('index', { title: 'node + mysql ', users });
});
// 增加
router.get('/user/create', async ctx => {
    await ctx.render('create', { title: '添加用户', method: 'add' })
})
router.post('/user/create', async ctx => {
    console.log('添加用户：',ctx.request)
    ctx.redirect('/')
})
// 修改
router.get('/user/edit', async ctx => {
    const codition = { id: ctx.query.id }
    console.log('查询要修改的用户',codition)
    await ctx.render('edit', { title: '修改用户', method: 'edit', user: {} })
})
router.get('/user/edit', async ctx => {
    console.log('要修改的用户：', ctx.request)
    ctx.redirect('/')
})
// 删除
router.get('/user/del/:id', async ctx => {
    console.log('删除用户id，', ctx.params.id)
    ctx.redirect('/')
})

module.exports = router;
```
目前为止所有的路由已经准备好了，需要挂载到 koa 实例上

```javascript
// 修改 app.js
// 引入路由部分
const indexRouter = require('./router/index')
// 挂载到 koa 实例
app.use(indexRouter.routes(), indexRouter.allowedMethods());
```

到这一步我们已经把页面做好了

打开浏览器输入 http://localhost:3000

到此为止，页面已经可以访问了

![新增页面](http://xinglong.tech/access/001/001_微信图片_20200423173813.png)
![查询页面](http://xinglong.tech/access/001/001_微信截图_20200423173714.png)

### 第三步 实现 module 层

新建 module 目录结构
```
  demo_001
  ├── config
  │   ├── dev.js
  │   ├── prd.js
  │   └── index.js
  ├── modules
  │   └── user.js
  ├── router
  ├── views
  ├── db.js
  ├── app.json
  └── package.json
```

我们先要配置数据库连接：

```javascript
// config/index.js
if (process.env.NODE_ENV === 'production') {
    module.exports = require('./prd')
} else {
    module.exports = require('./dev')
}
```
dev 和 prd 分别对应不同环境的数据库连接。这里都写成你自己的数据库地址即可
```javascript
// config/prd.js, config/dev.js
module.exports = {
    db: {
        // host
        host: '127.0.0.1',
        // 数据库名
        dbName: 'xxxx',
        // 用户名
        username: 'xxxx',
        // 密码
        password: 'xxxx'
    }
}
```

连接数据库 db.js

```javascript
// db.js
const Sequelize = require('sequelize');
const config = require('./config')

const { dbName, username, password, host } = config.db;

const sequelise = new Sequelize(
    dbName,
    username, password,
    {
        host: host,
        dialect: 'mysql',
        // 配置连接池
        pool: {
            max: 5,
            min: 0,
            acquire: 30000,
            idle: 10000
        }
})

sequelise
    .authenticate()
    .then(() => {
        console.log('数据库连接成功')
    })
    .catch(err => {
        throw new Error('数据库连接失败', err)
    })

module.exports = sequelise
```
创建 module

```javascript
// modules/user.js
const Sequelize = require('sequelize')

const sequelize = require('../db');

const User = sequelize.define('user', {
    username: { type: Sequelize.STRING },
    pwd: { type: Sequelize.STRING, },
    phone: { type: Sequelize.STRING, },
    age: { type: Sequelize.STRING, },
    gender: { type: Sequelize.INTEGER, },
});

module.exports.add = async (data) => await User.create(data)
module.exports.del = async (id) => await User.destroy({ where: { id } })

module.exports.update = async (data) => {
    let newUser = {...data}
    delete newUser['id']
    return await User.update({ ...newUser }, {
        where: { id: data.id }
    })
}

module.exports.find = async (condition) => {
    if (Object.keys(condition).length) {
        return await User.findAll({ where: { ...condition } })
    } else {        
        return await User.findAll();
    }
}
```
### 第四步 控制层调度
现在要通 **修改** 过路由，把视图和 module 结合起来。
```javascript
// router/index.js
// 引入
const UserMudule = require('../modules/user')

// 修改路由：增加
router.post('/user/create', async (ctx) => {
    const {
        username, pwd, phone, age, gender
     } = ctx.request.body
     await UserMudule.add({
        username, pwd, phone, age, gender
     })
    ctx.redirect('/')
})
// 修改路由：首页，查询所有用户
router.get('/', async ctx => {
    let users = await UserMudule.find(ctx.query)
    await ctx.render('index', { title: 'node + mysql ', users });
});
```

    此时已经实现了增加和查询，你可以测试一下这部分功能。

接下来实现修改和删除的代码

```javascript
// router/index.js
// 修改路由：delete
router.get('/user/del/:id', async ctx => {
    await UserMudule.del(ctx.params.id)
    ctx.redirect('/')
})

// 修改路由：update
router.get('/user/edit', async ctx => {
    const codition = { id: ctx.query.id }
    // 修改前先查询出 User 对象
    let user = await UserMudule.find(codition)
    await ctx.render('edit', { title: '修改用户', method: 'edit', user: user[0] })
})

// 修改路由：update
router.post('/user/edit', async ctx => {
    const {
        username, pwd, phone, age, gender, id
    } = ctx.request.body
    //  接受参数，执行修改
    await UserMudule.update({
    id, username, pwd, phone, age, gender
    })
    ctx.redirect('/')
})
```

来，测试一下！

![新增和查询](https://xinglong.tech/access/001/001_添加，查询.gif)

![修改和删除](https://xinglong.tech/access/001/001_修改，删除.gif)
### 总结

  到此你已经掌握了简单的 nodejs 服务器开发，下一篇文章我继续带你一步步的**上线**一个 nodejs 项目

  所以，如果你看完真觉得不错，那就给个 star 吧！你的赞和 star 是我编写更多精彩文章的动力[GitHub 地址](https://github.com/shixinglong007/shixinglong007.github.io)

### 后记

这篇文章我花了6个小时，写代码，录gif 脖子和胳膊都酸了~~
希望小伙伴们给我一点点打赏，鼓励我写成更多干货文章

![微信](http://xinglong.tech/access/wechart.jpg)
![支付宝](http://xinglong.tech/access/zhifubao.jpg)

> 作者：[石兴龙](https://xinglong.tech/)<br/>
> 来源：[GitHub](https://github.com/shixinglong007/shixinglong007.github.io)<br/>
>  <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br/>
>  本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。