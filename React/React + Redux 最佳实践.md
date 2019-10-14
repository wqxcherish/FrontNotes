#React + Redux 最佳实践
![avatar](https://camo.githubusercontent.com/21740ab2fdb2ba1504678bfddf39ab9943adfa39/68747470733a2f2f6f732e616c697061796f626a656374732e636f6d2f726d73706f7274616c2f506b4a564957464a62705a63776d532e706e67)

## URL > Data

需求: routing
react-router + react-router-redux: 前者是业界标准，后者可以同步 route 信息到 state，这样你可以在 view 根据 route 信息调整展现，以及通过 action 来修改 route。

## Data

需求: 为 redux 提供数据源，修改容易。

方案

plain object: 配合 combineReducer 已经可以满足需求。

同时在组织 Store 的时候，层次不要太深，尽量保持在 2 - 3 层。如果层次深，可以考虑用 updeep 来辅助修改数据。

## Data > View

需求: 数据的过滤和筛选。

方案

reselect: store 的 select 方案，用于提取数据的筛选逻辑，让 Component 保持简单。选 reselct 看重的是 可组合特性 和 缓存机制。

## Action <> Store，业务逻辑处理

需求:  统一处理业务逻辑，尤其是异步的处理。

方案

redux-saga: 用于管理 action，处理异步逻辑。可测试、可 mock、声明式的指令。

可选

redux-loop: 适用于相对简单点的场景，可以组合异步和同步的 action 。但他有个问题是改写了 combineReducer，会导致一些意想不到的兼容问题，比如我在特定场景下用不了 redux-devtool 。

redux-thunk, redux-promise 等: 相对原始的异步方案，适用于更简单的场景。在 action 需要组合、取消等操作时，会不好处理。

### saga 入门

在 saga 之前，你可能会在 action creator 里处理业务逻辑，虽然能跑通，但是难以测试。比如：
```js
// action creator with thunking
function createRequest () {
  return (dispatch, getState) => {
    dispatch({ type: 'REQUEST_STUFF' });
    someApiCall(function(response) {
      // some processing
      dispatch({ type: 'RECEIVE_STUFF' });
    });
  };
}
```
然后组件里可能这样：
```js
function onHandlePress () {
  this.props.dispatch({ type: 'SHOW_WAITING_MODAL' });
  this.props.dispatch(createRequest());
}
```
这样通过 redux state 和 reducer 把所有的事情串联到起来。

但问题是：

Code is everywhere.
通过 saga，你只需要触发一个 action 。
```js
function onHandlePress () {
  // createRequest 触发 action `BEGIN_REQUEST`
  this.props.dispatch(createRequest());
}
```
然后所有后续的操作都通过 saga 来管理。
```js
function *hello() {
  // 等待 action `BEGIN_REQUEST`
  yield take('BEGIN_REQUEST');
  // dispatch action `SHOW_WAITING_MODAL`
  yield put({ type: 'SHOW_WAITING_MODAL' });
  // 发布异步请求
  const response = yield call(myApiFunctionThatWrapsFetch);
  // dispatch action `PRELOAD_IMAGES`, 附上 response 信息
  yield put({ type: 'PRELOAD_IMAGES', response.images });
  // dispatch action `HIDE_WAITING_MODAL`
  yield put({ type: 'HIDE_WAITING_MODAL' });
}
```
