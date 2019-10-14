#Ant Design of React

## 安装： $ npm install antd

```
import {　DatePicker　}　from 'antd';
ReactDOM.render(<DatePicker/>,mountNode);
```

## 引入样式：
import 'antd/dist/antd.css';  // or 'antd/dist/antd.less'
按需加载可通过此写法 import DatePicker from 'antd/lib/date-picker' 或使用插件 babel-plugin-antd（此插件支持 js 和 css 同时按需加载）。

## Button 

通过设置 Button 的属性来产生不同的按钮样式，推荐顺序为：type -> shape -> size -> loading -> disabled

按钮的属性说明如下：
属性	说明	类型	默认值
type		设置按钮类型，可选值为 primary ghost 或者不设	string	-
htmlType	设置 button 原生的 type 值，可选值请参考 HTML 标准	string	button
icon		设置按钮的图标类型	string	-
shape		设置按钮形状，可选值为 circle circle-outline 或者不设	string	-
size		设置按钮大小，可选值为 small large 或者不设	string	default
loading		设置按钮载入状态	boolean	false
onClick		click 事件的 handler	function	-

<Button>Hello world!</Button> 最终会被渲染为 <button>Hello world!</button>，并且除了上表中的属性，其它属性都会直接传到 <button></button>。

##Icon

图标的命名规范#

我们为每个图标赋予了语义化的命名，命名规则如下:
实心和描线图标保持同名，用 -o 来区分，比如 question-circle(实心) 和 question-circle-o(描线)；
命名顺序：[icon名]-[形状可选]-[描线与否]-[方向可选]。
如何使用#

使用 <Icon /> 标签声明组件，指定图标对应的 type 属性，示例代码如下:
<Icon type="link" />
最终会渲染为：
<i class="anticon anticon-${type}"></i>

#Layout栅格

布局的栅格化系统，我们是基于行（row）和列（col）来定义信息区块的外部框架，以保证页面的每个区域能够稳健地排布起来。下面简单介绍一下它的工作原理：

通过row在水平方向建立一组column（简写col）

你的内容应当放置于col内，并且，只有col可以作为row的直接元素

栅格系统中的列是指1到24的值来表示其跨越的范围。例如，三个等宽的列可以使用.ant-col-8来创建

如果一个row中的col总和超过 24，那么多余的col会作为一个整体另起一行排列


Flex 布局

我们的栅格化系统支持 Flex 布局，允许子元素在父节点内的水平对齐方式 - 居左、居中、居右、等宽排列、分散排列。子元素与子元素之间，支持顶部对齐、垂直居中对齐、底部对齐的方式。同时，支持使用 order 来定义元素的排列顺序。
Flex 布局是基于 24 栅格来定义每一个『盒子』的宽度，但排版则不拘泥于栅格。

基础布局
从堆叠到水平排列。
使用单一的一组 Row 和 Col 栅格组件，就可以创建一个基本的栅格系统，所有列（Col）必须放在 Row 内。

区块间隔
栅格常常需要和间隔进行配合，你可以使用 Row 的 gutter 属性，我们推荐使用 (16+8n)px 作为栅格间隔。

左右偏移
列偏移。
使用 offset 可以将列向右侧偏。例如，offset={4} 将元素向右侧偏移了 4 个列（column）的宽度。

布局排序
列排序。
通过使用 push 和 pull 类就可以很容易的改变列（column）的顺序。

Flex 布局
Flex 布局基础。
使用 row-flex 定义 flex 布局，其子元素根据不同的值 start,center,end,space-between,space-around，分别定义其在父节点里面的排版方式。

Flex 对齐
Flex 子元素垂直对齐。

响应式布局
参照 Bootstrap 的 响应式设计，预设四个响应尺寸：xs sm md lg。

其他属性的响应式
span pull push offset order 属性可以通过内嵌到 xs sm md lg 属性中来使用。
其中 xs={6} 相当于 xs={{ span: 6 }}。

Row#

成员	说明	类型	默认值
gutter	栅格间隔	number	0
type	布局模式，可选 flex，现代浏览器下有效	string	
align	flex 布局下的垂直对齐方式：top middle bottom	string	top
justify	flex 布局下的水平排列方式：start end center space-around space-between	string	start

Col#

成员	说明	类型	默认值
span	栅格占位格数，为 0 时相当于 display: none	number	-
order	栅格顺序，flex 布局模式下有效	number	0
offset	栅格左侧的间隔格数，间隔内不可以有栅格	number	0
push	栅格向右移动格数	number	0
pull	栅格向左移动格数	number	0
xs	<768px 响应式栅格，可为栅格数或一个包含其他属性的对象	number or object	-
sm	≥768px 响应式栅格，可为栅格数或一个包含其他属性的对象	number or object	-
md	≥992px 响应式栅格，可为栅格数或一个包含其他属性的对象	number or object	-
lg	≥1200px 响应式栅格，可为栅格数或一个包含其他属性的对象	number or object	-