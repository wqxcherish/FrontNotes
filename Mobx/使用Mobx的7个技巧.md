# 正确理解概念

1. observable： 可被追踪变化的数据
2. observer： 响应observable 数据更新的组件
3. computed values: 可根据observable 数据计算返回值（此处的值也可理解为是observable）的函数
4. reactions: 监听observable 数据变化被触发执行的不同类型的函数

# 使用正确的reactions

1. autorun: 提供的函数总是立即被触发一次, 依赖关系改变时会再次被触发。依赖关系指的是在autorun函数中出现过的observable数据。
2. when: 你可以设置断言，当断言生效时响应函数会执行，响应仅会被触发一次
3. reaction: 与 autorun 类似，函数不会立即执行

# 为reactions命名

为reactions命名的好处是当程序出错时，可以快速定位出错位置。

# 使用严格模式

默认情况下，MobX允许你随便的更新observable数据。在大型应用中，若随意的变更数据会使程序状态无法追踪，不可预测。
为此，MobX 提供了严格模式，强迫我们只可以在 action 中更新observable数据。
