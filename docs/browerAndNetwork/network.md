# 网络相关
## 手写ajax
### 手写 XMLHttpRequest 不借助任何库

  ```js
  var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // 这里的函数异步执行，可参考之前 JS 基础中的异步模块
    if (xhr.readyState == 4) {
        if (xhr.status == 200) {
            alert(xhr.responseText)
        }
    }
}
xhr.open("GET", "/api", false) //建立连接，参数一：发送方式，二：请求地址，三：是否异步，true为异步
xhr.send(null) //
xhr.send(data);        //发送
  ```
 jq
 
  ```js
$.ajax({
	   type: "POST",
	   url: "test.php",
	   data: "name=garfield&age=18",
	   success: function(data){
				console.log(data)
		  },
		  error:function(xhr){
		     console.log(xhr)
		  }
	});
  ``` 
### xhr.readyState的状态码
  1. 0 -代理被创建，但尚未调用 open() 方法
	2. 1 -open() 方法已经被调用
	3. 2 -send() 方法已经被调用，并且头部和状态已经可获得
	4. 3 -下载中， responseText 属性已经包含部分数据

---

## HTTP协议
>http是HyperText Transfer Protocol的缩写，也称为超文本传输协议，最初的版本只能用来传输html文件，现在则可以传输包括文字、图像、视频和二进制文件的所有内容
### HTTP协议的主要特点
1. **简单快速**，每个资源固定出来起来方便
2. **灵活**，每个资源头部数据类型
3. **无连接**：连接一次就会断掉，不会一直连接
4. **无状态**，客户端和服务端是两种身份，http帮助连接，下次连接不会记住状态，是谁连接的

### http协议报文组成部分
请求报文：请求行、请求头、空行（告诉服务器请求头部到此为止）、请求体

响应报文：状态行（200 ok）、响应头、空行、响应体

### HTTP方法
 GET（获取资源） POST（传输资源） PUT（更新资源）DELETE（删除资源）HEAD（获得报文首部）

1. Get 请求能缓存，Post 不能
2. Post 相对 Get 安全一点点，因为Get 请求都包含在 URL 里，且会被浏览器保存历史纪录，Post 不会，但是在抓包的情况下都是一样的。
3. Post 可以通过 request body来传输比 Get 更多的数据，Get 没有这个技术
4. URL有长度限制，会影响 Get 请求，但是这个长度限制是浏览器规定的，不是 RFC 规定的
5. Post 支持更多的编码类型且不对数据类型限制
6. 在express框架中，对于GET请求的参数'?xxxx=',使用req.query.xxxxx方法获得；对于POST请求的参数，使用req.body.xxxxx方法获得

### http常见状态码

**2XX 成功**  

**200 OK，表示从客户端发来的请求在服务器端被正确处理**
204 No content，表示请求成功，但响应报文不含实体的主体部分  
206 Partial Content，进行范围请求  

**3XX 重定向**  

**301 moved permanently，永久性重定向，表示资源已被分配了新的 URL**  
**302 found，临时性重定向，表示资源临时被分配了新的 URL**  
303 see other，表示资源存在着另一个 URL，应使用 GET 方法丁香获取资源  
304 not modified，表示服务器允许访问资源，但因发生请求未满足条件的情况  
307 temporary redirect，临时重定向，和302含义相同  

**4XX 客户端错误**  

**400 bad request，请求报文存在语法错误**
401 unauthorized，表示发送的请求需要有通过 HTTP 认证的认证信息  
403 forbidden，表示对请求资源的访问被服务器拒绝  
**404 not found，表示在服务器上没有找到请求的资源** 

**5XX 服务器错误**  

**500 internal sever error，表示服务器端在执行请求时发生了错误**  
**503 service unavailable，表明服务器暂时处于超负载或正在停机维护，无法处理请求**  

---

###　http持久连接
HTTP 协议采用“请求-应答”模式，当使用普通模式，即非 Keep-Alive 模式时，每个请求/应答客户和服务器都要新建一个连接，完成之后立即断开连接（HTTP协议为无连接的协议）；

当使用` Keep-Alive `模式（又称持久连接、连接重用）时，`Keep-Alive` 功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，`Keep-Alive` 功能避免了建立或者重新建立连接

---

### HTTP 管线化

默认情况下 HTTP 协议中每个传输层连接只能承载一个 HTTP 请求和响应，浏览器会在收到上一个请求的响应之后，再发送下一个请求。在使用持久连接的情况下，某个连接上消息的传递类似于请求1 -> 响应1 -> 请求2 -> 响应2 -> 请求3 -> 响应3。

HTTP Pipelining（管线化）是将多个 HTTP 请求整批提交的技术，在传送过程中不需等待服务端的回应。使用 HTTP Pipelining 技术之后，某个连接上的消息变成了类似这样请求1 -> 请求2 -> 请求3 -> 响应1 -> 响应2 -> 响应3。

注意下面几点：

1. 管线化机制通过持久连接（persistent connection）完成，仅 HTTP/1.1 支持此技术（HTTP/1.0不支持）
2. 只有 GET 和 HEAD 请求可以进行管线化，而 POST 则有所限制
3. 初次创建连接时不应启动管线机制，因为对方（服务器）不一定支持 HTTP/1.1 版本的协议
4. 管线化不会影响响应到来的顺序，如上面的例子所示，响应返回的顺序并未改变
5. HTTP /1.1 要求服务器端支持管线化，但并不要求服务器端也对响应进行管线化处理，只是要求对于管线化的请求不失败即可
6. 由于上面提到的服务器端问题，开启管线化很可能并不会带来大幅度的性能提升，而且很多服务器端和代理程序对管线化的支持并不好，因此现代浏览器如 Chrome 和 Firefox 默认并未开启管线化支持

---

### HTTP 缓存机制和原理
对于强制缓存，服务器通知浏览器一个缓存时间，在缓存时间内，下次请求，直接用缓存，不在时间内，执行比较缓存策略

对于比较缓存，将缓存信息中的Etag和Last-Modified通过请求发送给服务器，由服务器校验，返回304状态码时，浏览器直接使用缓存

---


### HTTP与HTTPS的区别
HTTP：是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于从WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使网络传输减少。

HTTPS：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。

HTTPS和HTTP的区别主要如下：

1、https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。

2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。

3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

4、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

---

## 跨域
>什么是跨域？
跨域是指一个域下的文档或脚本试图去请求另一个域下的资源
1. 资源跳转： A链接、重定向、表单提交
2. 资源嵌入： `<link>、<script>、<img>、<frame>`等dom标签，还有样式中background:url()、@font-face()等文件外链
3. 脚本请求： js发起的ajax请求、dom和js对象的跨域操作等
>什么是同源策略？

所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个ip地址，也非同源。如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。

同源策略限制以下几种行为：
1. Cookie、LocalStorage 和 IndexDB 无法读取
2. DOM 和 Js对象无法获得
3. AJAX 请求不能发送

### 跨域解决方案
1. 通过jsonp跨域
2. document.domain + iframe跨域
3. location.hash + iframe
4. window.name + iframe跨域
5. postMessage跨域
6. 跨域资源共享（CORS）
7. nginx代理跨域
8. nodejs中间件代理跨域
9. WebSocket协议跨域

#### 1.通过jsonp跨域 --- 缺点只能实现get一种请求。
通常为了减轻web服务器的负载，我们把js、css，img等静态资源分离到另一台独立域名的服务器上，在html页面中再通过相应的标签从不同域名下加载静态资源，而被浏览器允许，基于此原理，我们可以通过动态创建script，再请求一个带参网址实现跨域通信。
##### 1.原生实现
```js
 <script>
  var script = document.createElement('script');
  script.type = 'text/javascript';

  // 传参并指定回调执行函数为onBack
  script.src = 'http://www.domain2.com:8080/login?user=admin&callback=onBack';
  document.head.appendChild(script);

  // 回调执行函数
  function onBack(res) {
      alert(JSON.stringify(res));
  }
 </script>
```
服务端返回如下（返回时即执行全局函数）：
```js
onBack({"status": true, "user": "admin"})
```
##### 2.ajax
```js
$.ajax({
  url: 'http://www.domain2.com:8080/login',
  type: 'get',
  dataType: 'jsonp',  // 请求方式为jsonp
  jsonpCallback: "onBack",    // 自定义回调函数名
  data: {}
});
```
#### 3.vue.js
```js
this.$http.jsonp('http://www.domain2.com:8080/login', {
  params: {},
  jsonp: 'onBack'
}).then((res) => {
  console.log(res); 
})
```

---

#### 2.document.domain + iframe跨域
>此方案仅限主域相同，子域不同的跨域应用场景。

实现原理：两个页面都通过js强制设置`document.domain`为基础主域，就实现了同域。

父窗口：(http://www.domain.com/a.html))
```js
<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>
<script>
  document.domain = 'domain.com';
  var user = 'admin';
</script>
```
子窗口：(http://child.domain.com/b.html))
```js
<script>
  document.domain = 'domain.com';
  // 获取父窗口中变量
  alert('get js data from parent ---> ' + window.parent.user);
</script>
```

---

#### 3.location.hash + iframe跨域
 
 实现原理： a欲与b跨域相互通信，通过中间页c来实现。 三个页面，不同域之间利用`iframe`的`location.hash`传值，相同域之间直接js访问来通信。  

具体实现：A域：`a.html` -> B域：`b.html` -> A域：`c.html`，a与b不同域只能通过hash值单向通信，b与c也不同域也只能单向通信，但c与a同域，所以c可通过`parent.parent`访问a页面所有对象。  


1.）a.html：`(http://www.domain1.com/a.html))`
  ```js
<iframe id="iframe" src="http://www.domain2.com/b.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');

    // 向b.html传hash值
    setTimeout(function() {
        iframe.src = iframe.src + '#user=admin';
    }, 1000);

    // 开放给同域c.html的回调方法
    function onCallback(res) {
        alert('data from c.html ---> ' + res);
    }
</script>
   ``` 
2.）b.html：`(http://www.domain2.com/b.html))`
  ```js
<iframe id="iframe" src="http://www.domain1.com/c.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');

    // 监听a.html传来的hash值，再传给c.html
    window.onhashchange = function () {
        iframe.src = iframe.src + location.hash;
    };
</script>
   ``` 
3.）c.html：`(http://www.domain1.com/c.html))`
  ```js
<script>
    // 监听b.html传来的hash值
    window.onhashchange = function () {
        // 再通过操作同域a.html的js回调，将结果传回
        window.parent.parent.onCallback('hello: ' + location.hash.replace('#user=', ''));
    };
</script>
   ``` 
---

#### 4.window.name + iframe跨域
 
`window.name`属性的独特之处：`name`值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 `name` 值（2MB）  
1.）`a.html：(http://www.domain1.com/a.html))`
  ```js
var proxy = function(url, callback) {
    var state = 0;
    var iframe = document.createElement('iframe');

    // 加载跨域页面
    iframe.src = url;

    // onload事件会触发2次，第1次加载跨域页，并留存数据于window.name
    iframe.onload = function() {
        if (state === 1) {
            // 第2次onload(同域proxy页)成功后，读取同域window.name中数据
            callback(iframe.contentWindow.name);
            destoryFrame();

        } else if (state === 0) {
            // 第1次onload(跨域页)成功后，切换到同域代理页面
            iframe.contentWindow.location = 'http://www.domain1.com/proxy.html';
            state = 1;
        }
    };

    document.body.appendChild(iframe);

    // 获取数据以后销毁这个iframe，释放内存；这也保证了安全（不被其他域frame js访问）
    function destoryFrame() {
        iframe.contentWindow.document.write('');
        iframe.contentWindow.close();
        document.body.removeChild(iframe);
    }
};

// 请求跨域b页面数据
proxy('http://www.domain2.com/b.html', function(data){
    alert(data);
});
   ``` 
2.）proxy.html：`(http://www.domain1.com/proxy…)`
中间代理页，与a.html同域，内容为空即可。

3.）b.html：`(http://www.domain2.com/b.html))`
  ```js
<script>
    window.name = 'This is domain2 data!';
</script>
  ```
总结：通过`iframe`的`src`属性由外域转向本地域，跨域数据即由`iframe`的`window.name`从外域传递到本地域。这个就巧妙地绕过了浏览器的跨域访问限制，但同时它又是安全操作。

---

#### 5.postMessage跨域
`postMessage`是`HTML5 XMLHttpRequest Level 2中的API`，且是为数不多可以跨域操作的window属性之一，它可用于解决以下方面的问题：

    a.） 页面和其打开的新窗口的数据传递
    b.） 多窗口之间消息传递
    c.） 页面与嵌套的iframe消息传递
    d.） 上面三个场景的跨域数据传递

用法：`postMessage(data,origin)`方法接受两个参数  
data： `html5`规范支持任意基本类型或可复制的对象，但部分浏览器只支持字符串，所以传参时最好用`JSON.stringify()`序列化。  
origin： 协议+主机+端口号，也可以设置为`"*"`，表示可以传递给任意窗口，如果要指定和当前窗口同源的话设置为`"/"`。  

1.）a.html：`http://www.domain1.com/a.html))`
  ```js
<iframe id="iframe" src="http://www.domain2.com/b.html" style="display:none;"></iframe>
<script>       
    var iframe = document.getElementById('iframe');
    iframe.onload = function() {
        var data = {
            name: 'aym'
        };
        // 向domain2传送跨域数据
        iframe.contentWindow.postMessage(JSON.stringify(data), 'http://www.domain2.com');
    };

    // 接受domain2返回数据
    window.addEventListener('message', function(e) {
        alert('data from domain2 ---> ' + e.data);
    }, false);
</script>
  ```
2.）b.html：`(http://www.domain2.com/b.html))`
  ```js
<script>
    // 接收domain1的数据
    window.addEventListener('message', function(e) {
        alert('data from domain1 ---> ' + e.data);

        var data = JSON.parse(e.data);
        if (data) {
            data.number = 16;

            // 处理后再发回domain1
            window.parent.postMessage(JSON.stringify(data), 'http://www.domain1.com');
        }
    }, false);
</script>
  ```
---

#### 6.跨域资源共享（CORS）----主要设置这个withCredentials 
普通跨域请求：只服务端设置`Access-Control-Allow-Origin`即可，前端无须设置，若要带`cookie`请求：前后端都需要设置。

需注意的是：由于同源策略的限制，所读取的`cookie`为跨域请求接口所在域的`cookie`，而非当前页。如果想实现当前页cookie的写入，可参考下文：`七、nginx反向代理`中设置`proxy_cookie_domain` 和 `八、NodeJs中间件代理`中`cookieDomainRewrite`参数的设置。

目前，所有浏览器都支持该功能`(IE8+：IE8/9需要使用XDomainRequest对象来支持CORS）)`，`CORS`也已经成为主流的跨域解决方案。

1、 前端设置：  
1.）原生ajax

// 前端设置是否带cookie
xhr.withCredentials = true;
示例代码：
  ```js
var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容

// 前端设置是否带cookie
xhr.withCredentials = true;

xhr.open('post', 'http://www.domain2.com:8080/login', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('user=admin');

xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        alert(xhr.responseText);
    }
};
  ```
2.）jQuery ajax
  ```js
$.ajax({
    ...
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
   crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    ...
});
  ```
3.）vue框架  
在vue-resource封装的ajax组件中加入以下代码：

`Vue.http.options.credentials = true`  


2、 服务端设置：
若后端设置成功，前端浏览器控制台则不会出现跨域报错信息，反之，说明没设成功。

1.）Java后台：
  ```js
/*
 * 导入包：import javax.servlet.http.HttpServletResponse;
 * 接口参数中定义：HttpServletResponse response
 */
response.setHeader("Access-Control-Allow-Origin", "http://www.domain1.com");  // 若有端口需写全（协议+域名+端口）
response.setHeader("Access-Control-Allow-Credentials", "true");
  ```
2.）Nodejs后台示例：
  ```js
var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var postData = '';

    // 数据块接收中
    req.addListener('data', function(chunk) {
        postData += chunk;
    });

    // 数据接收完毕
    req.addListener('end', function() {
        postData = qs.parse(postData);

        // 跨域后台设置
        res.writeHead(200, {
            'Access-Control-Allow-Credentials': 'true',     // 后端允许发送Cookie
            'Access-Control-Allow-Origin': 'http://www.domain1.com',    // 允许访问的域（协议+域名+端口）
            'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'   // HttpOnly:脚本无法读取cookie
        });

        res.write(JSON.stringify(postData));
        res.end();
    });
});

server.listen('8080');
console.log('Server is running at port 8080...');
  ```

---

#### 7.nginx代理跨域
1、 `nginx`配置解决`iconfont`跨域  
浏览器跨域访问`js、css、img`等常规静态资源被同源策略许可，但`iconfont`字体文件`(eot|otf|ttf|woff|svg)`例外，此时可在nginx的静态资源服务器中加入以下配置。
  ```js
location / {
  add_header Access-Control-Allow-Origin *;
}
  ```
2、 nginx反向代理接口跨域  
跨域原理： 同源策略是浏览器的安全策略，不是`HTTP`协议的一部分。服务器端调用`HTTP`接口只是使用`HTTP`协议，不会执行JS脚本，不需要同源策略，也就不存在跨越问题。

实现思路：通过`nginx`配置一个代理服务器（域名与`domain1`相同，端口不同）做跳板机，反向代理访问`domain2`接口，并且可以顺便修改`cookie`中`domain`信息，方便当前域`cookie`写入，实现跨域登录。

`nginx`具体配置：
  ```js
// proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;

    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
  ```
1.) 前端代码示例：
  ```js
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问nginx中的代理服务器
xhr.open('get', 'http://www.domain1.com:81/?user=admin', true);
xhr.send();
  ```
2.) Nodejs后台示例：
  ```js
var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var params = qs.parse(req.url.substring(2));

    // 向前台写cookie
    res.writeHead(200, {
        'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'   // HttpOnly:脚本无法读取
    });

    res.write(JSON.stringify(params));
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');
  ```

---

#### 8.Nodejs中间件代理跨域
`node`中间件实现跨域代理，原理大致与`nginx`相同，都是通过启一个代理服务器，实现数据的转发，也可以通过设置`cookieDomainRewrite`参数修改响应头中`cookie`中域名，实现当前域的`cookie`写入，方便接口登录认证。

1、 非vue框架的跨域（2次跨域）
利用node + express + http-proxy-middleware搭建一个proxy服务器。

1.）前端代码示例：
  ```js
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问http-proxy-middleware代理服务器
xhr.open('get', 'http://www.domain1.com:3000/login?user=admin', true);
xhr.send();

  ```
2.）中间件服务器：
  ```js
var express = require('express');
var proxy = require('http-proxy-middleware');
var app = express();

app.use('/', proxy({
    // 代理跨域目标接口
    target: 'http://www.domain2.com:8080',
    changeOrigin: true,

    // 修改响应头信息，实现跨域并允许带cookie
    onProxyRes: function(proxyRes, req, res) {
        res.header('Access-Control-Allow-Origin', 'http://www.domain1.com');
        res.header('Access-Control-Allow-Credentials', 'true');
    },

    // 修改响应信息中的cookie域名
    cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
}));

app.listen(3000);
console.log('Proxy server is listen at port 3000...');
  ```
3.）Nodejs后台同（六：nginx）

2、 vue框架的跨域（1次跨域）  
利用`node + webpack + webpack-dev-server`代理接口跨域。在开发环境下，由于`vue`渲染服务和接口代理服务都是`webpack-dev-server`同一个，所以页面与代理接口之间不再跨域，无须设置`headers`跨域信息了。

webpack.config.js部分配置：
  ```js
module.exports = {
    entry: {},
    module: {},
    ...
    devServer: {
        historyApiFallback: true,
        proxy: [{
            context: '/login',
            target: 'http://www.domain2.com:8080',  // 代理跨域目标接口
            changeOrigin: true,
            cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
        }],
        noInfo: true
    }
}
  ```

---

#### 9. WebSocket协议跨域
`WebSocket protocol`是`HTML`5一种新的协议。它实现了浏览器与服务器全双工通信，同时允许跨域通讯，是`server push`技术的一种很好的实现。
原生`WebSocket API`使用起来不太方便，我们使用`Socket.io`，它很好地封装了`webSocket`接口，提供了更简单、灵活的接口，也对不支持`webSocket`的浏览器提供了向下兼容。

1.）前端代码：
  ```js
<div>user input：<input type="text"></div>
<script src="./socket.io.js"></script>
<script>
var socket = io('http://www.domain2.com:8080');

// 连接成功处理
socket.on('connect', function() {
    // 监听服务端消息
    socket.on('message', function(msg) {
        console.log('data from server: ---> ' + msg); 
    });

    // 监听服务端关闭
    socket.on('disconnect', function() { 
        console.log('Server socket has closed.'); 
    });
});

document.getElementsByTagName('input')[0].onblur = function() {
    socket.send(this.value);
};
</script>
  ```
2.）Nodejs socket后台：
  ```js
var http = require('http');
var socket = require('socket.io');

// 启http服务
var server = http.createServer(function(req, res) {
    res.writeHead(200, {
        'Content-type': 'text/html'
    });
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');

// 监听socket连接
socket.listen(server).on('connection', function(client) {
    // 接收信息
    client.on('message', function(msg) {
        client.send('hello：' + msg);
        console.log('data from client: ---> ' + msg);
    });

    // 断开处理
    client.on('disconnect', function() {
        console.log('Client socket has closed.'); 
    });
});

  ```
