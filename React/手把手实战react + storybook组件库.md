# 引入

在UI组件库的开发过程中，如何方便的展示组件，测试组件，编写文档呢？storybook就提供了这样一种工具，利用它我们可以更方便地进行UI组件开发。最近一直在研究react，react和storybook的碰撞会是怎么样的呢。下面将从零开始，运用storybook手把手撸一个react组件。

# 快速搭建react环境
首先，基于create-react-app快速搭建react环境

storybook有单独的webpack配置，有单独的服务器。它的webpack配置类似于create-react-app。
```
npm i -g create-react-app
create-react-app storybook-demo
cd storybook-demo
yarn start
```
在命令行工具中运行上述命令，快速生成了一个已经配置好react环境的storybook-demo项目文件，进入该项目文件，通过yarn start启动webpack服务器，操作无误的话，浏览器将会自动打开如下页面：

## 安装storybook
搭配好react环境后，安装storybook插件
npm i --save-dev @storybook/react

## 配置npm scripts

之前已经提到过，storybook有单独的服务器，可以在npm scripts添加相关命令脚本，方便之后启动服务。

打开package.json，在scripts属性中添加storybook命令，如下：
{
  "scripts": {
    "storybook": "start-storybook -p 9001 -c .storybook"
  }
}
注意，脚本中-p 9001表示启动服务的端口号是9001，-c .storybook表示配置文件的目录，这里将配置文件设置为根目录下的.storybook文件夹。
## 创建配置文件config.js
在项目根目录中新建.storybook目录，并在里面添加config.js文件。
基本的配置主要是用来告诉storybook你的stories的存放位置。
我们将stories放在src目录下。
```
// config.js
import { configure } from '@storybook/react';

function loadStories() {
  require('../src/stories/index.js');
}

configure(loadStories, module);

// 
```
## 创建stories
接下来登场的就是stories，这是storybook的核心所在。
打开src目录，创建stories文件夹，同时可以创建components文件夹。
在components中，创建一个Button.js组件，在stories添加button.stories

```
// .storybook/config.js中修改路径
import { configure } from '@storybook/react';

function loadStories() {
  require('../src/stories/button.stories.js');
}

configure(loadStories, module);

// src/stories/button.stories.js
import React from 'react';
import { storiesOf } from '@storybook/react';
import {Button} from '../components/Button';

storiesOf('Button', module)
  .add('基本用法',() => (
    <Button>按钮</Button>
  ))
  
// src/components/Button.js
import React from 'react'

export class Button extends React.Component{
  constructor (props) {
    super(props)    
  }

  render () {
    return (
      <button style={{backgroundColor: '#fff', border: '1px solid #ccc'}}>{this.props.children}</button>
    )
  }
}
```
这里，在config.js中仅引入了button的story，如果组件比较多，有没有批量引入stories的方法呢？
可以使用require.context方法，此处将src/stories文件夹中以.stories.js为后缀的文件批量导入。
require.context是webpack的方法，具体参考require.context
```
import { configure } from '@storybook/react';

const req = require.context('../src/stories', true, /\.stories\.js$/)

function loadStories() {
  req.keys().forEach((filename) => req(filename))
}

configure(loadStories, module);
```
## 启动storybook服务
yarn storybook
在浏览器中打开http://localhost:9001，成功后会出现以下页面，证明你成功了！

添加额外组件信息
想要形成一个完整的组件文档，需要展示出组件的各种信息，上面展示的信息是远远不够的。为此，可以安装addon-info插件。插件更多信息参考addon-info
npm i -D @storybook/addon-info
复制代码在button.stories.js文件中引入withInfo方法，参数可以是字符串，支持markdown
```
// src/stories/button.stories.js
import { withInfo } from '@storybook/addon-info';

storiesOf('Button', module)
  .add('基本用法',
    withInfo(`
      description or documentation about my component, supports markdown
    
      ~~~js
      <Button>测试按钮</Button>
      ~~~
    
    `)(() =>
      <Button>测试按钮</Button>
    )
  )
  ```
此时浏览器页面更新如下。点击右侧的show info按钮，可以看到我们添加的组件信息。


使用markdown
可以将组件信息抽出来放在md文件中。直接上代码：
```
// src/stories/button.md
description or documentation about my component, supports markdown
<Button>测试按钮</Button>
复制代码// src/stories/button.stories.js
import { withInfo } from '@storybook/addon-info';
import button from './button.md'

storiesOf('Button', module)
  .add('基本用法',
    withInfo({
      markdown: button
    })(() =>
      <Button>测试按钮</Button>
    )
  )
  ```
组件信息单页显示
如何直接让组件信息在当前页面显示呢？在withInfo配置中添加inline:true
```
storiesOf('Button', module)
  .add('基本用法',
    withInfo({
      inline: true,
      markdown: button
    })(() =>
      <Button>测试按钮</Button>
    )
  )
```
