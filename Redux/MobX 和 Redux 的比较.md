先要明白 mobx 和 redux 的定位是不同的。redux 管理的是 (STORE -> VIEW -> ACTION) 的整个闭环，而 mobx 只关心 STORE -> VIEW 的部分。

但作为两个目前最火的 React 应用框架库，人们习惯于把他们比较到一起。下面我们也来看下 mobx 和 redux 相比的优缺点。(据说每个列 3 点会让人更容易记住。。)

#优点

基于运行时的数据订阅

mobx 的数据依赖始终保持了最小，而且还是基于运行时。而如果用 redux，可能一不小心就多订阅或者少订阅了数据。所以为了达到高性能，我们需要借助 PureRenderMixin 以及 reselect 对 selector 做缓存。所以，如果。

OverSubscription 的例子：

非实时计算
view() {
  if (count === 0) {
    return a;
  } else {
    return b;
  }
}
基于 redux 的方案，我们必须同时监听 count, a 和 b 。在 counte === 0 的时候，b 如果修改了，也会触发 view 。而这个时候的 b 其实是无意义的。

粗粒度 subscription
view() {
  todos[0].title
}
基于 redux，我们通常会订阅 todos，这样 todos 的新增、删除都会触发 view 。其实这里真正需要监听的是 todos 第一个元素的 title 属性是否有修改。
通过 OOP 的方式组织领域模型 (domain model)

OOP 的方式在某些场景下会比较方便，尤其是容易抽取 domain model 的时候。进而由于 mobx 支持引用的方式引用数据，所以可以非常容易得形成模型图 (model graph )，这样可以更好地理解我们的应用。
修改数据方便自然

mobx 是基于原生的 JavaScript 对象、数组和 Class 实现的。所以修改数据不需要额外语法成本，也不需要始终返回一个新的数据，而是直接操作数据。
# 缺点

缺最佳实践和社区

mobx 比较新，遇到的问题可能社区都没有遇到过。并且，mobx 并没有很好的扩展/插件机制。
随意修改 store

我们都知道 redux 里唯一可以改数据的地方是 reducer，这样可以保证应用的安全稳定；而 mobx 可以随意修改数据，触发更新，给人一种不安全的感觉。

最新的 mobx 2.2 加入了 action 的支持。并且在开启 strict mode 之后，就只有 action 可以对数据进行修改，限制数据的修改入口。可以解决这个问题。
逻辑层的限制

如果更新逻辑不能很好地封装在 domain class 里，用 redux 会更合适。另外，mobx 缺类 redux-saga 的库，业务逻辑的整合不知道放哪合适。