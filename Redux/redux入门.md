# 什么是Redux
Redux是一个流行的JavaScript框架，为应用程序提供一个可预测的状态容器。
## 数据流图
![数据流图](https://segmentfault.com/img/bVWi0h?w=687&h=330)

在Redux中，所有的数据（比如state）被保存在一个被称为store的容器中 → 在一个应用程序中只能有一个。store本质上是一个状态树，保存了所有对象的状态。任何UI组件都可以直接从store访问特定对象的状态。要通过本地或远程组件更改状态，需要分发一个action。分发在这里意味着将可执行信息发送到store。当一个store接收到一个action，它将把这个action代理给相关的reducer。reducer是一个纯函数，它可以查看之前的状态，执行一个action并且返回一个新的状态。

#配置Redux
配置Redux开发环境的最快方法是使用create-react-app工具。在开始之前，确保已经安装并更新了nodejs，npm和yarn。我们生成一个redux-shopping-cart项目并安装Redux：
```bash
npm install -g create-react-app
create-react-app redux-shopping-cart
cd redux-shopping-cart
yarn add redux 
npm install redux
```
```js
import { createStore } from "redux";

const reducer = function(state, action) {
  return state;
}
const store = createStore(reducer);
```

让我解释一下上面的代码：

首先，我们从redux包中引入createStore()方法。
我们创建了一个名为reducer的方法。第一个参数state是当前保存在store中的数据，第二个参数action是一个容器，用于：
type - 一个简单的字符串常量，例如ADD, UPDATE, DELETE等。
payload - 用于更新状态的数据。
我们创建一个Redux存储区，它只能使用reducer作为参数来构造。存储在Redux存储区中的数据可以被直接访问，但只能通过提供的reducer进行更新。


