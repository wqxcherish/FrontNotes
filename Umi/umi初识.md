# 初识Umi

## 环境搭建 
```bash
$ node -v   # node 版本是 8.10 或以上
$ yarn global add umi
$ umi -v   #全局安装 umi，并确保版本是 2.0.0 或以上
```


## 简单Demo
```bash
$ yarn create umi                            #初始化umi项目 选择ant-design-pro
$ yarn install                               #安装项目依赖模块
$ umi block add ant-design-pro/UserLogin     #添加登陆模块 
$ umi dev                                    #启动项目 http://localhost:8000/user 
$ umi build                                  #部署项目
```

##[什么是Umi](https://umijs.org/zh/guide/)

umi，中文可发音为乌米，是一个可插拔的企业级 react 应用框架。umi 以路由为基础的，支持类 next.js 的约定式路由，以及各种进阶的路由功能，并以此进行功能扩展，比如支持路由级的按需加载。然后配以完善的插件体系，覆盖从源码到构建产物的每个生命周期，支持各种功能扩展和业务需求。

个人理解: umi的约定式路由 在 规划模块以及页面时, 对于开发者而言非常的简便 路由级的按需加载 也提高了项目整体运行的速度 美中不足的是当 执行umi build指令时  项目难以部分更新和维护

### 架构

![umi架构图](https://gw.alipayobjects.com/zos/rmsportal/zvfEXesXdgTzWYZCuHLe.png)


### 目录及约定

在文件和目录的组织上，umi 更倾向于选择约定的方式。

一个复杂应用的目录结构如下：
```JavaScript
.
├── dist/                          // 默认输出路径，可通过配置 outputPath 修改。
├── mock/                          // mock 文件所在目录，基于 express
├── config/
    ├── config.js                  // umi 配置，同 .umirc.js，二选一
└── src/                           // 源码目录，可选
    ├── layouts/index.js           // 全局布局
    ├── pages/                     // 页面目录，里面的文件即路由
        ├── .umi/                  // dev 临时目录，需添加到 .gitignore
        ├── .umi-production/       // build 临时目录，会自动删除
        ├── document.ejs           // HTML 模板
        ├── 404.js                 // 404 页面
        ├── page1.js               // 页面 1，任意命名，导出 react 组件
        ├── page1.test.js          // 用例文件，umi test 会匹配所有 .test.js 和 .e2e.js 结尾的文件
        └── page2.js               // 页面 2，任意命名
    ├── global.css                 // 约定的全局样式文件，自动引入，也可以用 global.less
    ├── global.js                  // 可以在这里加入 polyfill
    ├── app.js                     // 运行时配置文件
├── .umirc.js                      // umi 配置，同 config/config.js，二选一
├── .env                           // 环境变量
└── package.json
```


### 路由
#### 约定式路由
```js
//1. 基础路由
//假设 pages 目录结构如下：
+ pages/
  + users/
    - index.js
    - list.js
  - index.js
//那么，umi 会自动生成路由配置如下：
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/', component: './pages/users/index.js' },
  { path: '/users/list', component: './pages/users/list.js' },
]
//2.动态路由
//umi 里约定，带 $ 前缀的目录或文件为动态路由。
//比如以下目录结构：
+ pages/
  + $post/
    - index.js
    - comments.js
  + users/
    $id.js
  - index.js
//会生成路由配置如下：
[
  { path: '/', component: './pages/index.js' },
  { path: '/users/:id', component: './pages/users/$id.js' },
  { path: '/:post/', component: './pages/$post/index.js' },
  { path: '/:post/comments', component: './pages/$post/comments.js' },
]
```

#### 配置式路由
如果你倾向于使用配置式的路由，可以配置 routes ，此配置项存在时则不会对 src/pages 目录做约定式的解析。
```js
export default {
  routes: [
    { path: '/', component: './a' },
    { path: '/list', component: './b', Routes: ['./routes/PrivateRoute.js'] },
    { path: '/users', component: './users/_layout',
      routes: [
        { path: '/users/detail', component: './users/detail' },
        { path: '/users/:id', component: './users/id' }
      ]
    },
  ],
};
```
注意：
component 是相对于 src/pages 目录的

### 在页面间跳转
在 umi 里，页面之间跳转有两种方式：声明式和命令式。

#### 声明式

基于 umi/link，通常作为 React 组件使用。
```js
import Link from 'umi/link';

export default () => (
  <Link to="/list">Go to list page</Link>
);
```
#### 命令式
基于 umi/router，通常在事件处理中被调用。
```js
import router from 'umi/router';

function goToListPage() {
  router.push('/list');
}
```



### HTML 模板
#### 修改默认模板
新建 src/pages/document.ejs，umi 约定如果这个文件存在，会作为默认模板，
内容上需要保证有 
```html
<div id="root"></div>
```
比如：
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Your App</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

#### 配置模板
模板里可通过 context 来获取到 umi 提供的变量，context 包含：

route，路由对象，包含 path、component 等
config，用户配置信息
publicPath2.1.2+，webpack 的 output.publicPath 配置
env，环境变量，值为 development 或 production
其他在路由上通过 context 扩展的配置信息
模板基于 ejs 渲染，可以参考 https://github.com/mde/ejs 查看具体使用。

比如输出变量，
```html
<link rel="icon" type="image/x-icon" href="<%= context.publicPath %>favicon.png" />
```
比如条件判断，
```html
<% if(context.env === 'production') { %>
  <h2>生产环境</h2>
<% } else {%>
  <h2>开发环境</h2>
<% } %>
```
#### 针对特定页面指定模板
WARNING
此功能需开启 exportStatic 配置，否则只会输出一个 html 文件。

TIP
优先级是：路由的 document 属性 > src/pages/document.ejs > umi 内置模板

配置路由的 document 属性。

比如约定式路由可通过注释扩展 document 属性，路径从项目根目录开始找，

/**
 * document: ./src/documents/404.ejs
 */
然后这个路由就会以 ./src/documents/404.ejs 为模板输出 HTML。


