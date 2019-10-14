在dva中主要分3层，models,services,components,其中models是最重要概念，这里放的是各种数据，与数据交互的应该都是在这里。services是请求后台接口的方法。components是组件了。

services层：
```js
export function doit (body) {
    return request({
        method: "post",
        url: `${wechatApi}/doit`,
        data: JSON.stringify(body),
    })
}
```


这里就是请求后台接口的方法，其中这里的request是封装了axios的函数,所以它是返回的是一个promise对象，url就是要请求的地址，body就是请求参数了。

那个request如下
```js
import axios from "axios"

export default async function request (options) {
    let response
    try {
        response = await axios(options)
        return response
    } catch (err) {
        return response
    }
}
```
models层：
```js
export default {
  namespace: "test", //命名空间名字，必填  
  state: { num: 0 }，//state就是用来放初始值的
// 能改变界面的action应该放这里,这里按官方意思不应该做数据处理，只是用来return state 从而改变界面
reducers：{
    addNum ( // addNum可以理解为一个方法名 
    // 这里state就是上面初始的state，这里理解是旧state
    state, { payload: { num }}// num 是传过来的，名字随便起，不是state中的num，这接收一个action 　　　　　　
    )     　　　　
    { 
    //return新的state,这样页面就会更新 es6语法，就是把state全部展开，然后把num:num重新赋值，这样后面赋值的num就会覆盖前面的。也是es6语法，相同名字可以写成一个，所以上面接收处写了num
        return { ...state, num}
    }
},
,
// 与后台交互，处理数据逻辑的地方
effects:{
    * fetchNum({ payload2 }, { call, put,select }) {//fetchNum方法名，payload2是传来的参数，是个对象，如果没参数可以写成{_,{call,put,select}}
    const { data } = yield call(myService.doit, {anum:payload2.numCount}) // myService是引入service层那个js的一个名字，anum是后台要求传的参数，data就是后台返回来的数据
    //const m = yield select((state) => state.test.num) //select就是用来选择上面state里的，这里没用上

    yield put({
      type: "addNum",// 这就是reducer中addNum方法, put就是用来触发上面reducer的方法，payload里就是传过去的参数。 同时它也能触发同等级effects中其他方法。
      payload: {
        num: data, // 把后台返回的数据赋值给了num，假如那个reducer中方法是由这里effects去触发的，那个num名必须是这里名字num，如果reducer中方法不是这触发，那名字可随便起
      },
    })
  },
* fetchUser(_,{call,put}) {
    // XXXXXXX代码
  }
},
subscriptions:{
  // 订阅监听，比如我们监听路由，进入页面就如何，可以在这写
  setup({dispatch, history, query})
  {
    return history.listen(async ({pathname, search, query}) => {
      if (pathname === "/testdemo") {// 当进入testdemo这路由，就会触发fetchUser方法
        dispatch({type: "fetchUser"})
      }
    })

  }
}
}
```
components层：
```js
clickHandler = () => {
  dispatch({
    type: "test/fetchNum",// 这里就会触发models层里面effects中fetchNum方法（也可以直接触发reducer中方法，看具体情况） ,test就是models里的命名空间名字
    payload: {
      numCount: ++1,
    },
  })
}
```
 

所以整体流程是：

点击页面按钮，会触发clickHandler,——>触发models层effect的fetchNum——>触发services层doit，获取到后台返回数据——>触发models层的addNum，把返回数据传给addNum，再去更新models里的state,components应用了models层中的state的num的话，就会触发页面render方法重新渲染，界面就会更新。

render方法什么时候会触发

当state或props变化时就会触发render，我们一般在render里只获取props和state，尽量不做逻辑处理(数据逻辑处理基本在render上面的函数或者models中处理)。当父组件给子组件传递props时，子组件那个props最好不要在render里面做逻辑计算赋值，不然传递过去，子

组件有可能拿不到最新的值。比如传了个数组arr,arr在render里做了数据处理，赋值，render会运行多次(这里举例3次)所以结果可能是[1,2,3] [1,2,3] [1,2]，子组件拿到的值是[1,2,3]而不是最终的[1,2]，所以当你出现

子组件无法获取父组件传递过来最后正确的值，看看是不是值在render做了运算赋值，解决方法就是把数据逻辑放在models层处理，然后再返回，这样就没问题了。

 

页面要应用models层的数据要用connect
```js

import { Component } from "react"
import { connect } from "dva"


class TheDemo  extends Component {
    clickHandler = () =>{xxxx}
   render () {
    const {num} = this.props //获取下面的num
        return (
        <div>
            <button onClick={this.clickHandler}><button>
            <p>{num}</p>
        </div>
)
    }
}

//字面意思就是，把models的state变成组件的props
function mapStateToProps (state) {
    const { num} = state.test // test就是models命名空间名字 
    return {
        num, // 在这return,上面才能获取到
    }
}

export default connect(mapStateToProps)(TheDemo)    
```       