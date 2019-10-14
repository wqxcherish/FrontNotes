## 1.加载与执行

多数浏览器 使用单一进程来处理 **用户界面(UI)** 刷新 和 **JavaScript脚本** 执行
1. 推荐将 所有的 `<script>`标签尽可能放到`<body>` 标签的底部,以尽量减少对整个页面下载的影响.
2. 减少 页面中外链脚本文件的数量 合并JavaScript文件


#### 1.1 无阻塞的脚本
在页面加载完成后 才加载JavaScript代码. 即 在window对象的load事件触发后再下载脚本

#### 1.2 延迟脚本  
HTML4为`<script>`标签定义了一个扩展属性:defer
```js
<script type="text/javascript" src="file1.js" defer></script> 
```
带有defer属性的`<script>`标签可以放置在文档的任何位置.对应的JavaScript文件将在页面解析到`<script>`标签时 开始下载,但并不会执行,直到DOM加载完成(onload事件被触发前) 当一个带有defer属性的JavaScript文件下载时,它并不会阻塞浏览器的其他进程,因此这类文件可以与页面中的其他资源并行下载.

#### 1.3 动态脚本元素
可以用JavaScript动态创建HTML中的几乎所有内容

```js
var script = document.createElement("script");
script.type = "text/javascript";
script.src = "file1.js";
document.getElementByTagName("head")[0].appendChild(script);
```

这个新创建的`<script>` 元素加载了file1.js文件. 文件在该元素被添加到页面时开始下载.
这种技术的重点在于: 无论在何时启动下载,文件的下载和执行过程不会阻塞页面其他进程.


使用动态脚本节点下载文件时,返回的代码通常会立刻执行(除了Firefox 和 Opera,它们会等待此前所有动态脚本节点执行完毕) 当脚本"只执行"时,这种机制运作正常. 但是当代码只包含供页面其他脚本调用的接口时,就会有问题. 在这种情况下,你必须跟踪并确保脚本下载完成且准备就绪.这可以用动态`<scirpt>`节点触发的事件来实现.

Firefox Opera Chrome 和 Safari 以上版本 会在 `<script>`元素接收完成时触发一个load事件

IE 则支持 readystatechange事件

#### 1.4 动态加载JavaScript文件 
通用方法
```js
function loadScript(url,callback) {
    var script = document.createElement("script")
    script.type = "text/javascript";
    if (script.readyState) {
        script.onreadystatechange = function() {
            if (scirpt.readyState == 'loaded' || script.readyState == 'complete'){
                scirpt.onreadystatechange = null;
                callback();
            }
        }

    }else {
        script.onload = function() {
            callback();
        }
    }
    script.src = url;
    document.getElementsByTagName("head")[0].appendChild(script);
}

```

#### 1.5 XMLHttpRequest 脚本注入

另一种 无阻塞加载脚本的方法 是 使用XMLHttpRequest 对象 获取脚本并注入页面中

先创建一个xhr对象,然后用它下载JavaScript文件,最后通过创建动态`<script>`元素将代码注入页面中.
```js
var xhr = new XMLHttpRequest();
xhr.open("get","file1.js",true);
xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
        if (xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
            var script = document.createElement("script");
            script.type = "text/javascript";
            script.text = xhr.responseText;
            document.body.appendChild(script);
        }
    }
};
xhr.send(null);

```

#### 1.6 推荐的无阻塞模式

先添加动态加载所需的代码,然后加载初始化页面所需的剩下的代码

```html
<script type="text/javascript" src="loader.js"></script>
<script type="text/javascript">
    loadScript("the-rest.js",function() {
        Application.init();
    })
</script>
```

#### 1.7 总结 
减少JavaScript对性能的影响:
1. `<body>` 闭合标签之前,将所有的`<script>`标签放到页面底部. 这能确保在脚本执行前页面已经完成了渲染.
2. 合并脚本 页面中的`<script>`标签越少,加载也就越快,响应也跟迅速. 无论外链文件 还是内嵌脚本都是如此

3. 有多种无阻塞下载JavaScript的方法:
- 使用`<script>` 标签的defer属性
- 使用动态创建的 `<script>`元素来下载并执行代码
- 使用xhr对象下载JavaScript代码并注入页面中


## 2.数据存取

字面量 本地变量 数组元素 对象成员

#### 2.1 作用域链 和 标识符解析

每一个JavaScript函数 都表示为一个对象 ,Function对象的实例,拥有可以编程访问的熟悉,和一系列不能通过代码访问而仅提供JavaScript引擎存取的内部属性

内部属性 `[[Scope]]` 包含了一个函数被创建的作用域中对象的集合. 这个集合 被称为函数的作用域链,它决定哪些数据能被函数访问.

函数作用域中的每个对象 被称为一个可变对象,每个可变对象都以键值对的形式存在
当一个函数创建后 ,它的作用域链会被创建此函数的作用域中可访问的数据对象所填充


例子:
```js
function add(num1,num2) {
    var sum = num1 + num2;
    return sum;
}


```
当函数 add() 创建时, 它的作用域链中插入了一个对象变量,这个全局对象代表着 所有在全局范围内定义的变量.

函数add的作用域将会在 执行时用到

var total = add(5,10);

执行此函数 时 会创建一个成为执行环境的内部对象.一个执行环境定义了一个函数执行时的环境.
函数每次执行时对应的执行环境都是独一无二的 当函数执行完毕,执行环境就被销毁

每个执行环境 都有自己的作用域链, 用于解析标识符. 当执行环境被创建时,它的作用域链初始化为当前运行环境 `[[Scope]]` 属性中的对象. 这些值按照它们出现在函数中的顺序,被复制到执行环境的作用域链中. 这个过程一旦完成,一个被称为"活动对象"的新对象就为执行环境创建好了. 活动对象作为函数运行时的变量对象,包含了所有局部变量,命名参数,参数集合,以及this.然后此对象被推入作用域链的最前端. 当执行环境被销毁,活动对象也随之销毁.

在函数执行过程中,每遇到一个变量,都会经历一次标识符解析过程以 决定从哪里获取或存储数据.该过程搜索执行环境的作用域链,查找同名的标识符.搜索过程从作用域链 头部开始,也就是当前运行函数的活动对象.如果找到,就使用这个标识符对应的变量;如果没找到,继续搜索作用域链中的下一个对象.搜索过程会持续进行,直到找到标识符,若无法搜索到匹配的对象,那么标识符将被视为是未定义的.

在函数执行过程中,每个标识符都要经历 这样的搜索过程,影响了性能


#### 2.2 标识符解析的性能

在执行环境的作用域链中,一个标识符所在的位置越深,它的读写速度也就越慢.

提高性能的方法:
1. 先将全局变量的引用存储在一个局部变量中,然后使用这个局部变量代替全局变量


改变作用域链 
with语句 用来给对象的所有属性创建一个变量
try-catch语句  当try代码块中发生错误,执行过程会自动跳转到catch子句,然后把异常对象推入一个变量对象 并置于作用域的首位.在catch代码块内部,函数所有局部变量将会放在第二个作用域链对象中


#### 2.3 动态作用域

with语句 try-catch语句的catch子句 eval()函数 

#### 2.4 闭包

允许函数访问局部作用域之外的数据.

```js
function assignEvents() {
    var id = 'xdi9592';
    document.getElementById("save-btn").onclick = function(event) {
        saveDocument(id);
    }
}
```
assignEvents()函数 给一个DOM元素设置 事件处理函数.这个事件处理函数就是一个闭包,它在assignEvents() 执行时创建,并且能访问所属作用域的id变量.为了让这个闭包访问id,必须创建一个特定的作用域链

当assignEvents()函数执行时,一个包含了变量id 以及 其他数据的活动对象被创建. 它成为执行环境作用域链中的第一个对象,而全局对象紧随其后. 当闭包被创建时,它的`[[Scope]]` 属性被初始化为这些对象

由于闭包的`[[Scope]`属性 包含了 与执行环境作用域链相同的对象的引用,因此会产生副作用

通常来说 函数的活动对象会随着执行环境一同销毁 但引入闭包时,由于引用仍然存在于闭包的`[[Scope]`属性中,因此激活对象无法被销毁.这意味着 脚本中的闭包与非闭包函数 相比,需要更多的内存开销.

当闭包代码执行时,会创建一个执行环境,它的作用域链与属性`[[Scope]]`中所引用的两个相同作用域链对象一起被初始化,然后一个活动对象为闭包自身所创建
 
#### 2.5 原型

JavaScript中的对象是基于 **原型** 的.
原型是其他对象的基础,它定义并实现了 一个新创建的对象所必须包含的成员列表,原型对象为所有对象实例所共享

对象通过一个内部属性绑定到它的原型. 在Firefox Safari Chrome 浏览器中,这个属性 `_protp_ `对开发者可见,而其他的浏览器却不允许脚本访问此属性. 一旦你创建一个内置对象的实例,它们就会自动拥有一个`Object`实例作为原型

对象可以有两种成员类型: `实例成员` 和 `原型成员` 
**实例成员直接存在于对象实例中,原型成员则从对象原型继承而来**

解析对象成员 的过程与解析变量十分相似 当book.toString()被调用时,**会从对象实例开始**,搜索名为 'toSring'的成员.一旦book没有名为toString的成员 那么会**继续搜索其原型对象**,知道toString()方法被找到并且执行.

搜索实例成员比从字面量 或局部变量中读取数据代价更高,再加上遍历原型链带来的开销,这让性能问题更为严重

通常来说 在函数中如果要多次读取同一个对象属性,最佳做法是**将属性值保存到局部变量中**. 局部变量能用来替代属性 以避免多次查找带来的性能开销.特别是在处理嵌套对象成员时,这样做会明显提升执行速度

#### 2.6 小结

在JavaScript中,数据存储的位置 会对代码整体性能产生重大的影响. 数据存储共有4种方法: 字面量 变量 数组项 对象成员

- 访问字面量和局部变量的速度最快 相反,访问数据元素 和 对象成员相对较慢
- 由于局部变量存在于 作用域链的起始位置,因此访问局部变量比 访问跨作用域变量更快. 变量在作用域链中的位置越深,访问所需的时间就越长. 由于全局变量总处在作用域链的最末端 因此访问的速度也是最慢的

- 避免使用with语句,因为它会改变执行环境作用域链.同样, try-catch语句中的catch子句也有同样的影响

- 嵌套的对象成员会明显影响性能,尽量少用

- 属性或方法在原型链中的位置越深,访问它的速度也越慢

- 通常来说 ,你可以通过把常用的对象成员, 数组元素 ,跨域变量 保存在局部变量中来 改善JavaScript性能,因为局部变量访问速度更快

## 3. DOM编程

对HTML元素 集合循环操作

```js
function innerHTMLLoop() {
    for (var count = 0; count < 1500; count ++) {
        document.getElementById('here').innerHTML += 'a';
    }
}

//改进版

function innerHTMLLoop2() {
    var content = '';
    for (var count = 0; counr < 1500; count ++) {
        count += 'a';
    }
    document.getElementById('here').innerHTML += content;
}

```
如果在一个对性能有着苛刻要求的操作中更新一大段HTML 推荐使用innerHTML,速度较快. 但对大多数日常操作而言,并没有太大区别.

HTML集合 一直与文档保持着连接,每次你需要最新的信息时,都会重复执行查询的过程,哪怕只是获取集合里的元素个数(即访问集合的length属性)

一般来说 对于任何类型的DOM访问,需要多次访问同一个DOM属性或方法需要多次访问时,最好使用一个局部变量缓存此成员
当遍历一个集合时,第一优化原则是把集合存储在局部变量中,并把length缓存在循环外部,然后,使用局部变量替代这些需要多次读取的元素


```JavaScript

//较慢
function collectionGlobal() {
    var coll = document.getElementByTagName('div'),
        len = coll.length,
        name = '';
    for (var count = 0; count < len; count ++) {
        name = document.getElementByTagName('div')[count].nodeName;
        name =  document.getElementByTagName('div')[count].nodeType;
        name =  document.getElementByTagName('div')[count].tagName;
    }
    return name;
}
//较快
function collectionGlobal() {
    var coll = document.getElementByTagName('div'),
        len = coll.length,
        name = '';
    for (var count = 0; count < len; count ++) {
        name = coll[count].nodeName;
        name =  coll[count].nodeType;
        name =  coll[count].tagName;
    }
    return name;
}
//最快
function collectionGlobal() {
    var coll = document.getElementByTagName('div'),
        len = coll.length,
        name = '',
        el = null;
    for (var count = 0; count < len; count ++) {
        el = coll[count];
        name = el.nodeName;
        name =  el.nodeType;
        name =  el.tagName;
    }
    return name;
}

```
获取DOM 元素 

通常 你需要从某一个DOM元素开始,操作周围的元素,或者递归查找所有的子节点.你可以使用childNodes得到元素集合,或者用
nextSibling 来获取每个相邻元素

```js
function testNextSibling() {
    var el = document.getElementById('mydiv'),
        ch = el.firstChild,
        name = '';
        do {
            name = ch.nodeName;
        } while(ch = ch.nextSibling);
        return name;
}

function testChildNode() {
    var el = document.getElementById('mydiv'),
        ch = el.childNodes,
        len = ch.length,
        name = '';
    for (var count = 0; count < len;count ++) {
        name = ch[count].nodeName;
    }
    return name;
}

```

重绘与重排

浏览器下载完 页面中的所有组件  HTML标记 JavaScript CSS 图片 之后会解析生成两个内部数据结构 DOM树 渲染树
DOM树种 的每一个需要显示的节点在渲染树中 至少存在一个对应的节点(隐藏的DOM元素在渲染树中没有对应的节点)

渲染树中的节点 被称为 '帧' 或 '盒'

理解页面元素为一个 具有 内边距 (padding), 外边距 (margins), 边框 (borders) 位置(position) 的盒子

一旦 DOm 和 渲染树构建完成,浏览器就开始显示(绘制'paint')页面元素

当Dom 的变化 影响了元素的几何属性(宽和高) 比如改变边框宽度 或 给段落增加文字,导致行数增加 浏览器需要重新计算元素的几何属性,同样其他元素的几何属性和位置 也会因此受到影响. 浏览器 会使渲染树中受到影响的部分失效,并冲销构造渲染树. 这个过程称为 重排(reflow) 完成重排后 浏览器会重新绘制受影响的部分到屏幕中,该过程位 重绘(repaint)


当页面布局和 几何属性改变时 需要重排

- 添加或删除可见的DOM元素
- 元素位置改变
- 元素尺寸改变 (包括 外边距 内边距 边框厚度 宽度 高度等)
- 内容改变 例如 文本改变或图片被另一个不同尺寸的图片替代
- 页面渲染器初始化
- 浏览器窗口尺寸改变

最小化 重绘 和 重排

合并多次对DOM和样式的修改 
- cssText 属性 : el.style.cssText = 'border-left: 1px; border-right: 2px; padding: 5px;';
修改css的class名称

批量修改DOM 

当你需要对 DOM元素 进行一系列操作时,可以通过以下步骤来减少重绘 和 重排的次数:

- 使元素脱离文档流
- 对其应用多重改变
- 把元素带回文档中


使DOM 脱离文档

隐藏元素 应用修改 重新显示

使用文档片段 在当前DOM之外构建一个子树,再把它拷贝回文档

将原始元素 拷贝到一个脱离文档的节点中,修改副本,完成后再替换原始元素

例子

```html
<ul>
    <li><a href="http://phpied.com">php1</a></li>
    <li><a href="http://phpied.com">php2</a></li>
</ul>
```
```js
var data = [
    {
        "name": 'baidu',
        "url": ''
    },
    {
        "name": 'sougou',
        "url": ''
    }
]
function appendDataToElement(appendToElement,data) {
    var a, li;
    for (var i = 0; max = data.length; i < max; i++ ) {
        a = document.createElement('a');
        a.href = data[i].url;
        a.appendChild(document.createTextNode(data[i].name));
        li = document.createElement('li');
        li.appendChild(a);
        appendToElement.appendChild(li);
    }
}

//方案一

var url = document.getElementById('myList');
appendDataToElement(ul,data);

//方案二

var url = document.getElementById('myList');
ul.style.display = 'none';
appendDataToElement(ul,data);
ul.style.display = 'block';

//方案三
var fragment = document.createDocumentFragment();
appendDataToElement(fragment,data);
document.getElementById('mylist').appendChild(fragment)

//方案四

var old = document.getElementById('mylist');
var clone = old.cloneNode(true);
appendDataToElement(clone,data);
old.parentNode.replaceChild(clone,old);
```
缓存布局信息

当你查询 布局信息时,比如获取偏移量(offsets) 滚动位置(scroll values) 或计算出的样式值(computedstyle values)时,浏览器 为了返回最新值,会刷新队列并应用所有变更.

最好的做法 就是尽量减少布局信息的获取次数,获取后把它赋值给局部变量,然后再操作局部变量

例子

把myElement 元素 沿对角线移动,每移动一个像素,从100像素 X 100像素的位置开始,到 500像素 X 500像素的位置 结束

```js
//效率低下
myElement.style.left = 1 + myElement.offsetLeft + 'px';
myElement.style.top = 1 + myElement.offsetTop + 'px';

if (myElement.offsetLeft >= 500) {
    stopAnimaion();
}
var currentLeft = myElement.offsetLeft
var currentTop = myElement.offsetTop

currentLeft ++
currentTop ++
myElement.style.left = currentLeft + 'px';
myElement.style.top = currentTop + 'px';

if (myElement.offsetLeft >= 500) {
    stopAnimaion();
}

```
让元素 脱离动画流

使用以下步骤可以避免页面中的大部分重排

1. 使用觉得位置定位 页面上的动画元素,使其脱离文档流
2. 让元素动起来. 当它扩大时,会临时覆盖部分页面.但这只是页面的一个小区域的重绘过程,不会产生重排并重绘页面的大部分内容
3. 当动画结束时 恢复定位,从而只会下移一次文档的其他元素


事件委托

事件逐层冒泡并能被父级元素捕获. 使用事件代理,只需给外层元素绑定一个处理器,就可以处理在其子元素上触发的所有事件


```js
document.getElementById('menu').onclick = function(e) {
    e = e || window.event;
    var target = e.target || e.srcElement;
    var pageid,hrefparts;
    if (target.nodeName ! == 'A') {
        return;
    }
    hrefparts = target.href.split('/');
    pageid = hrefparts[hrefparts.length - 1];
    pageid = pageid.replace('.html','');

    //更新页面

    ajaxRequest('xhr.php?page=' + id, updatePageContents);

    //浏览器组织默认行为并取消冒泡

    if(typeof e.preventDefault === 'function') {
        e.preventDefault();
        e.stopPropagation();
    } else {
        e.returnValue = false;
        e.cancelBubble = true;
    }
}

```
跨浏览器 兼容的部分包括: 
- 访问事件对象,并判断事件源

- 取消文档树中的冒泡
- 阻止默认动作

小结

为了减少DOM编程带来的性能损失,请记住以下几点:
最小化DOM访问次数,尽可能在JavaScript端处理
如果需要多次访问某个DOM节点,请使用局部变量存储它的引用
小心处理HTML集合,因为它实时连系着底层文档 把集合的长度缓存到一个变量中,并在迭代中使用它. 如果需要经常操作集合,建议把它拷贝到一个数组中
使用速度更快的API 比如 querySelectorAll() 和 firstElementChild
要留意重绘和重排 批量修改样式 离线操作DOM树,使用缓存,并减少访问布局信息的次数

动画中使用绝对定位,使用拖放代理
使用事件委托来减少事件处理器的数量

## 4.算法和流程控制

for-in 循环

可以枚举任何对象的属性名

for(var prop in object) {
    //循环主体
}

循环体每次运行时,prop变量被赋值为 object的一个属性名 (字符串),直到所有属性遍历完成才返回.所返回的属性包括对象实例属性 以及从原型链中继承而来的属性

优化if - else

最小化到达正确分支前所需判断的条件数量

最简单的优化方法是确保最可能出现的条件放在首位

if - else 中的条件语句 应该总是按照最大概率到最小概率的顺序排列,以确保运行速度最快

另一种减少条件判断次数的方法 是 把if - else 组织成一系列嵌套的if - else 语句


JavaScript中可以使用数组和普通对象来构建查找表,通过查找表访问数据 比用 if -else 或 switch 快很多,特别是在条件语句数量很大的时候

```js
var results = [result0,result1,result2,result3,result4,result5,result6,result7,result8,result9]

return results[value];

```
当你使用查找表时,必须完全抛弃条件判断语句. 这个过程变成数据项查询 或者 对象成员查询

查找表的一个主要优点是 : 不用书写任何条件判断语句,即便候选值数量增加时,也几乎不会产生额外的性能开销


递归函数的潜在问题 是 终止条件不明确或缺少终止条件会导致函数长时间运行,并使得用户界面处于假死状态.而且,递归函数还可能遇到浏览器的 '调用栈大小限制'

JavaScript 引擎支持的递归数量 与 JavaScript调用栈大小直接相关.只有IE例外,它的调用栈与系统空闲内存有关,而其他所有浏览器都有固定数量的调用栈限制.

递归模式

1. factorial()函数为代表的直接递归模式,即函数调用自身

2. 隐伏模式 包含两个函数: 两个函数相互调用,形成一个无限循环.


迭代
任何递归能实现的算法 同样可以用迭代来实现.

合并排序算法 是最常见的递归实现算法

```js
function merge(left,right) {
    var result = [];
    while (left.length > 0 && right.length > 0) {
        if (left[0] < right[0]) {
            result.push(left.shift());
        } else {
            result.push(right.shift());
        }
    }
    return result.concat(left).concat(right);
}

function mergeSort(items) {
    if (items.length == 1) {
        return items;
    }
    var middle = Math.floor(items.length / 2),
        left = items.slice(0,middle),
        right = items.slice(middle);
    return merge(mergeSort(left),mergeSort(right));
}

```
```js
function mergeSort(items) {
    if (items.length == 1) {
        return items;
    }
    var work = [];
    for (var i = 0, len = items.length; i < len; i++) {
        work.push([items[i]]);
    }
    work.push([]);
    for (var lim = len; lim > 1; lim = (lim+1)/2) {
        for (var j = 0,k = 0; k < lim; j++ ,k+=2) {
            work[j] = merge(work[k],work[k+1]);
        }
        work[j] = [];
    }
    return work[0]
}


```
Memoization 

减少工作量 就是最好的性能优化技术.代码要处理的事越少,它的运行速度就越快.

Memoization

缓存前一个计算结果 供后续计算机使用.


小结
for while 和 do-while 循环性能特性相当

避免使用 for- in 循环 除非你需要遍历一个属性数量未知的对象

改善循环性能的最佳方式是减少每次迭代的运算量和减少循环迭代次数

通常来说,switch 总是比 if -else快 ,但并不总是 最佳解决方案

在判断条件较多时,使用查找表比if-else 和 switch 更快

浏览器的调用栈 大小限制了递归算法 在JavaScript中的应用 栈溢出错误会导致其他代码中断运行

如果你遇到栈溢出错误 ,可将方法改为迭代算法,或使用Memoization 来避免重复计算

## 5.字符串和正则表达式

优化字符串连接
```js
str = 'a' + 'b' + 'c';

str = 'a';
str += 'b';
str += 'c'; 

str = ['a','b','c'].join(""),

str = 'a'
str = str.concat('b','c')
```

join

concat 

正则表达式

去除字符串首尾空白


小结

密集的字符串操作和草率地编写正则表达式可能产生严重的性能障碍 
- 当连接数量巨大 或 尺寸巨大的字符串时,数据项合并是唯一在IE7及更早版本中性能合理的方法
- 如果不考虑IE7及更早版本的性能,数组项合并是最慢的字符串连接方法之一.推荐使用简单的 + 和 +=操作费替代,避免不必要的中间字符串
- 回溯既是正则表达式匹配功能的基本组成部分,也是正则表达式的低效之源
- 回溯失控发生在正则表达式本应快速匹配的地方,但因为某些特殊的字符串匹配动作 导致运行缓慢甚至浏览器崩溃.避免这个问题的方法: 使相邻的字元互斥,避免嵌套量词对同一个字符串的相同部分多次匹配,通过重复利用预查的原子组去除不必要的回溯
- 提高正则表达式效率的各种技术手段会有助于正则表达式更快地匹配,并在非匹配位置上花更少的时间
- 正则表达式并不总是完成工作的最佳工具,尤其当你只搜索字面字符串的时候
- 尽管有许多方法可以去除字符串的首尾空白,但使用两个简单的正则表达式(一个用来去除头部空白,另一个用来去除尾部空白) 来处理大量字符串内容能提供一个简洁而跨浏览器的方法.
从字符串末尾开始循环向前搜索第一个非空白字符,或者将此技术同正则表达式结合起来,会提供一个更好的替代方案,它很少受到字符串长度影响


## 6.快速响应的用户界面

用来执行JavaScript和更新用户界面的线程 通常被称为 浏览器UI线程

UI线程的工作基于一个简单的队列系统,任务会被保存到队列中直到进程空闲.一旦空闲,队列中的下个任务就被重新提取出来并运行.