## 1.XHR 介绍
IE5 第一款引入XHR对象的浏览器,通过MSXML库中的ActiveX对象实现的

MSXML2.XMLHttp MSXML2.XMLHttp.3.0 MSXML2.XMLHttp.6.0

IE7+ Firefox Opera Chrome Safari 都支持原生的XHR对象
var xhr = new XMLHttpRequest();

## 2.XHR 用法
```js
var xhr = new XMLHttpRequest();
xhr.open('get','example.php',false);
xhr.send(null);
```

收到响应后 响应的数据会自动填充XHR对象的属性

responseText：作为响应主体被返回的文本
responseXML：如果响应的内容类型是 'text/html' 或 'application/xml',这个属性中将保存包含着响应数据的XML DOM文档
status：响应的HTTP状态
statusText：HTTP状态的说明

接收到响应后(同步请求)
检查status 属性 200成功 304请求资源并没有被修改，使用浏览器缓存版本


接收到响应后(异步请求)
检测 XHR 对象的 readyState 属性
0 未初始化 未调用open()方法
1 启动 已经调用 open()方法 未调用send()方法
2 发送 已经调用 send()方法 未收到响应
3 接收 已经接收到部分响应数据
4 完成 已经接收到全部响应数据

例子
```js
var xhr =  new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        if (xhr.status >= 200 && xhr.status < 300 || xhr.status ===304>) {
            alert(xhr.responseText)
        }else {
            alert ("Resquest was unsuccessful:" + xhr.status)
        }
    }
}
xhr.open("get","example.txt",true);
xhr.send(null);
```

## 3.HTTP头部信息
xhr对象 提供操作 请求头部 和响应头部 新的的方法
发送xhr请求的同时，还会发送下列头部信息
Accept： 浏览器能够处理的内容类型
Accept-Charset： 浏览器 能够显示的字符集
Accept-Encoding: 压缩编码
Accept-Language： 当前设置的语言
Connection： 浏览器鱼服务器之间连接的类型
Host： 发出请求的页面所在的域
Referer： 发出请求的页面的URI
User-Agent： 浏览器的用户代理字符串

```js
var xhr =  new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        if (xhr.status >= 200 && xhr.status < 300 || xhr.status ===304>) {
            alert(xhr.responseText)
        }else {
            alert ("Resquest was unsuccessful:" + xhr.status)
        }
    }
}
xhr.open("get","example.txt",true);
xhr.setRequestHeader('MyHeader','MyValue') //必须在open()方法之后 send()方法之前
xhr.send(null);
```
## 4.get请求 
查询字符串参数追加到URL的末尾 必须使用encodeURIComponent()进行编码 所以名-值对 必须有&分开


辅助向现有URL的末尾添加查询字符串参数

```js
function addURLParam(url,name,value) {
    url += (url.indexof('?') == -1 ? '?' : '&');
    url += encodeURIComponent(name) + '=' + encodeURIComponent(value);
    return url;
}
```

## 5.post 请求
XHR 模拟表单提交
1. 将Content-Type 头部信息 设置为 application/x-www-form-urlencoded
2. 使用serialize()函数 来创建字符串

```js
function submitData () {
    var xhr =  new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (xhr.readyState === 4) {
            if (xhr.status >= 200 && xhr.status < 300 || xhr.status ===304>) {
                alert(xhr.responseText)
            }else {
                alert ("Resquest was unsuccessful:" + xhr.status)
            }
        }
    }
    xhr.open("post","example.txt",true);
    xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded') //必须在open()方法之后 send()方法之前
    var form = document.getElementById('user-info');
    xhr.send(serialize(form));
}

```

## 6.XMLHttpRequest 2级
1. FormData 序列化表单 以及创建于表单格式相同的数据(用于通过XHR传输)
```js
var data = new FormData();
data.append('name','Nicholas');
```
2. 超时设定 timeout属性
```js
xhr.timeout = 1000;
xhr.ontimeout = function() {
    //code
}
```
3. overriderMimeType() 重写XHR 响应的MIME类型

## 7.进度事件(Progress Events)
loadstart 在接收到响应数据的第一个字节时触发
progress  在接受响应期间不断地触发  
error   在请求发错错误是触发
abort   在因为调用abort()方法而终止连接时触发
load    在接受到完整的响应数据时触发
loadend 在通信完成或者触发 error abort load 事件后触发

## 8.跨资源共享(CORS)
1. Access-Control-Allow-Origin： 域名
2. JSONP
3. Comet

