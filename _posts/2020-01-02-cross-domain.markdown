---
layout:     post
title:      "前端跨域"
subtitle:   ""
date:       2020-01-02 22:26:51
author:     "Z1hgq"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - JavaScript
    - 网络
    - 跨域
---

> “我们这个世界，从不会给一个伤心的落伍者颁发奖牌”

---

### 同源策略

#### 同源的定义

如果两个页面的协议，端口（如果有指定）和主机都相同，则两个页面具有相同的源。
源由下面三个部分组成
1. 协议
2. 端口
3. 主机

下表给出了相对http://store.company.com/dir/page.html同源检测的示例:

|                    URL                          | 结果 | 原因 |
| ----------------------------------------------- | --- | --- |
| http://store.company.com/dir2/other.html        | 成功 | 只有路径不同 |
| http://store.company.com/dir/inner/another.html | 成功 | 只有路径不同 |
| https://store.company.com/secure.html           | 失败 | 不同协议 ( https和http )|
| http://store.company.com:81/dir/etc.html        | 失败 | 不同端口 ( http:// 80是默认的)|
| http://news.company.com/dir/other.html          | 失败 | 不同域名 ( news和store )|

#### 同源策略的意义

**同源策略**限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离在恶意文件的重要安全机制


假设没有同源策略，那么我在A网站下的cookie就可以被任何一个网站拿到；那么这个网站的所有者，就可以使用我的cookie(也就是我的身份)在A网站下进行操作。
同源策略可以算是 web 前端安全的基石，如果缺少同源策略，浏览器也就没有了安全性可言。


我们平时使用xhr访问不同域名的数据时，浏览器就会报跨域错误，我们看不到响应数据
此时其实请求是已经到了服务器并且成功响应，只是因为浏览器因为同源策略将此次响应拦截了而已。

#### 限制范围

通常来说同源策略的限制范围有如下几个：

1. 无法共享和操作 cookie, localStorage, indexDB
2. 无法操作彼此的 dom 元素
3. 无法发送 ajax 请求
3. 无法通过 flash 发送 http 请求
4. 其他

但是同源策略是允许嵌入外部资源的。比如：
```js
1. <script></script>标签嵌入js脚本资源
2. <img />标签嵌入图片资源
3. <video></video>和<audio></audio>标签嵌入多媒体资源
4. <link />标签嵌入css或者其他类似资源
5. <object>, <embed> 和 <applet>
6. @font-face 引入的字体资源
7. Iframe嵌入网页
```
这些请求都会发送get请求，然后响应资源，我们也能在浏览器中看到它们，但是本站js脚本因为同源策略并不能读取这些资源。
这里放一个我个人的理解，同源策略的限制主要是限制在js脚本中操作不同域资源时，比如使用xhr请求一个图片资源、在canvas中使用不同域图片、操作iframe中不同域的dom等等。

---

### 跨域

有同源策略的限制就有跨域，因为我们不可能只在一个域交互，下面介绍几种现在比较常用的几种跨域方式：
- Jsonp
- Cors
- 代理
- Websocket

其他跨域方式这里就不做介绍，因为不算很常用，需要了解可以在网上查资料

---

### JSONP

Jsonp主要是通过script标签在加载资源时不受同源策略的限制且js脚本在加载完成后会自动执行的原理来实现的，这个需要服务器返回 callback(数据);的形式，如果window上也有一个callback的函数，这样脚本加载回来就会执行callback函数，我们就能拿到数据了。
```js
// 实现
(function (global) {
    var id = 0,
        container = document.getElementsByTagName("head")[0];

    function jsonp(options) {
        if(!options || !options.url) return;

        var scriptNode = document.createElement("script"),
            data = options.data || {},
            url = options.url,
            callback = options.callback,
            fnName = "jsonp" + id++;

        // 添加回调函数
        data["callback"] = fnName;

        // 拼接url
        var params = [];
        for (var key in data) {
            params.push(encodeURIComponent(key) + "=" + encodeURIComponent(data[key]));
        }
        url = url.indexOf("?") > 0 ? (url + "&") : (url + "?");
        url += params.join("&");
        scriptNode.src = url;

        // 传递的是一个匿名的回调函数，要执行的话，暴露为一个全局方法
        global[fnName] = function (ret) {
            callback && callback(ret);
            container.removeChild(scriptNode);
            delete global[fnName];
        }

        // 出错处理
        scriptNode.onerror = function () {
            callback && callback({error:"error"});
            container.removeChild(scriptNode);
            global[fnName] && delete global[fnName];
        }

        scriptNode.type = "text/javascript";
        container.appendChild(scriptNode)
    }

    global.jsonp = jsonp;

})(this);

// 使用示例
jsonp({
    url : "www.example.com",
    data : {id : 1},
    callback : function (ret) {
        console.log(ret);
    }
});

// 服务器端
var http = require('http');
var urllib = require('url');

var port = 8080;
var data = {'data':'world'};

http.createServer(function(req,res){
    var params = urllib.parse(req.url,true);
    if(params.query.callback){
        console.log(params.query.callback);
        //jsonp
        var str = params.query.callback + '(' + JSON.stringify(data) + ')';
        res.end(str);
    } else {
        res.end();
    }
    
}).listen(port,function(){
    console.log('jsonp server is on');
});
```
jsonp的缺点是只能支持get请求，其他请求不行，且需要服务器支持，成本较高。

---

### CORS

`CORS`（`Cross-Origin Resource Sharing`，跨源资源共享）定义了在必须访问跨源资源时，浏览器与服务器应该如何沟通。`CORS`背后的基本思想，就是使用自定义的`HTTP`头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功，还是应该失败。

`CORS`需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于IE10
。
整个`CORS`通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，`CORS`通信与同源的`AJAX`通信没有差别，代码完全一样。浏览器一旦发现`AJAX`请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。
因此，实现`CORS`通信的关键是服务器。只要服务器实现了`CORS`接口，就可以跨源通信。

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。
只要同时满足以下两大条件，就属于简单请求。

> 1. 请求方法是以下三种方法之一：
> - HEAD
> - GET
> - POST
> 2. HTTP的头信息不超出以下几种字段：
> - Accept
> - Accept-Language
> - Content-Language
> - Last-Event-ID
> - Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/> - form-data、text/plain

凡是不同时满足上面两个条件，就属于复杂请求。且浏览器对这两种请求的处理，是不一样的。

#### 简单请求

对于简单请求，浏览器直接发出CORS请求，在头信息之中，增加一个Origin字段标识当时请求的是来自哪个源。
```
GET /cors HTTP/1.1
Origin: http://api.bob.com
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
服务器接收到请求后判断该源是否在许可范围内，如果在就会在响应头中增加如下几个字段:
```
Access-Control-Allow-Origin: http://api.bob.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```
浏览器判断要Access-Control-Allow-Origin这个字段不存在或者不与当前源对应，就会抛出跨域错误，而且响应状态码还可能是200。否则就响应成功。

Access-Control-Allow-Credentials这个字段主要是表示是否允许给服务器发送Cookie，要生效的话需要将xhr的withCredentials字段设置为ture。

#### 复杂请求

对于复杂请求，浏览器在发出真正请求前会首先执行一次options请求，来判断服务器是否此次跨域，流程和上面的简单请求一样，不过多了一个允许跨域的方法判断。

在options请求通过时，以后的每个真正的请求就跟简单请求一样的流程了。如果不通过一样会抛出错误，被xhr捕获到。

Cors比jsonp更强大，只需要服务器配合，且能支持多种请求，开发者在使用时完全和平时一样，浏览器会自动处理这些流程。在jsonp慢慢被淘汰的情况下，cors是以后跨域的一种好的方式。

---

### 代理

在其他方式都不能使用的情况下我们还可以使用反向代理的方式来实现跨域

反向代理就是当服务器收到一个请求时，如果服务器没有此资源，就去其他服务器将资源获取过来(包括cookie和其他头信息)，然后在原封不动的返回给客户端。

我们使用反向代理就可以做到请求同一个源的接口，拿到不同源的数据了。


反向代理的方式有很多，服务器都可以做，最简单的是使用`nginx`，配置一下转发策略即可， `nginx`的用法可以去查看相关文档，这里不做介绍

我们还可以使用前端最熟悉的nodejs来做反向代理转发，这里介绍一个相关中间件，`express-http-proxy`和`koa-proxy`，他们的原理一样是通过提供的转发策略来做资源的转发。






