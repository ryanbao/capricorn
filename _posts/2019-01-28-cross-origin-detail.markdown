---
layout: post
title: Sth About Cross Origin & Solution
date: 2019-01-28
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [FRONTEND,CORS]
---

#### 什么是跨域

跨域是指浏览器不能执行其他网站的脚本。是由浏览器的同源策略造成的，是浏览器对JavaScript实施的安全限制。同源策略是浏览器最核心也是最基本的安全功能，要求浏览器只被允许访问来自统一站点的资源而不是那些来自其它站点可能怀有恶意的资源；

所谓的同源是指：协议、域名和端口相同。

{% highlight ruby %}
URL: http[s]://domain:port/resources
{% endhighlight %}

同源策略限制了一下行为：

* Cookie LocalStorage 和 IndexDB无法跨域读取
* DOM 和 JS 对象无法获取 如：iframe 跨域的情况，不同域名的 iframe 是限制互相访问的。
* Ajax请求发送不出去 如：XMLHttpRequest同源策略，禁止使用 XHR 对象向不同源的服务器地址发起 HTTP 请求


#### 为什么浏览器会禁止跨域 - 安全问题

跨域的访问一般会带来很多安全性的问题。我们知道HTTP请求都是无状态的，用户在浏览器上访问很多情况下会和后台服务产生交互，为了标识用户我们一般都会在浏览器的Cookie中存放用户登录信息；如果允许跨域访问，那么恶意的网站只要通过一段脚本就可以获取用户的cookie，从而冒充用户去登录网站 造成安全问题，因为浏览器都会采用同源策略来避免跨域访问。


#### 跨域的解决方案

##### 1.JSONP

由于 `script` 标签不受浏览器同源策略的影响，允许跨域引用资源。因此可以通过动态创建 script 标签，然后利用 src 属性进行跨域，这也就是 JSONP 跨域的基本原理。

jsonp跨域其实也是JavaScript设计模式中的一种代理模式。在html页面中通过相应的标签从不同域名下加载静态资源文件是被浏览器允许的，所以我们可以通过这个“犯罪漏洞”来进行跨域。一般，我们可以动态的创建script标签，再去请求一个带参网址来实现跨域通信

// 原生的实现方式

{% highlight ruby %}
let script = document.createElement('script');
script.src = 'http://www.ryanbao.cn/xxx?param=value&callback=callback';
document.body.appendChild(script);
function callback(res) {
	console.log(res);
}
{% endhighlight %}

// jquery jsonp的实现方式

{% highlight ruby %}
$.ajax({
    url:'http://www.ryanbao.cn/xxx',
    type:'GET',
    dataType:'jsonp',//请求方式为jsonp
    jsonpCallback:'callback',
    data:{
        "param":"value"
    }
})
{% endhighlight %}

##### 优点

* 使用简便，没有兼容性问题，目前最流行的一种跨域方法。

##### 缺点

* 只支持 GET 请求。
* 由于是从其它域中加载代码执行，因此如果其他域不安全，很可能会在响应中夹带一些恶意代码。
* 要确定 JSONP 请求是否失败并不容易。虽然 HTML5 给 script 标签新增了一个 onerror 事件处理程序，但是存在兼容性问题。


##### 2.CORS

CORS（Cross-origin resource sharing，跨域资源共享）是一个 W3C 标准，定义了在必须访问跨域资源时，浏览器与服务器应该如何沟通。CORS 背后的基本思想，就是使用自定义的HTTP头部让浏览器与服务器进行沟通，从而决定请求或响应是应该成功，还是应该失败。CORS 需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE 浏览器不能低于 IE10。

整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信。

浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

同时满足以下两大条件就是简单请求

1.请求方法是 GET POST HEAD 三种方法之一

2.HTTP的头信息不超出以下几种字段：
* Accept
* Accept-Language
* Content-Language
* Last-Event-ID
* Content-Type：只限于三个值 application/x-www-form-urlencoded、multipart/form-data、text/plain

凡是不同时满足上面两个条件，就属于非简单请求

浏览器对这两种请求的处理，是不一样的。

##### 2.1 简单请求

在请求中需要附加一个额外的 `Origin`头部，其中包含请求页面的源信息（协议+域名+端口），以便服务器根据这个头部信息来决定是否给予响应。例如：`Origin: http://www.ryanbao.cn`

{% highlight ruby %}
GET /get/1 HTTP/1.1
Origin: http://www.ryanbao.cn
Host: api.ryanbao.cn
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0
...
{% endhighlight %}

Origin字段用来说明，本次请求来自哪个源。服务器根据这个值，决定是否同意这次请求。如果服务器认为这个请求可以接受，服务器返回的响应，会多出几个头信息字段：

{% highlight ruby %}
Access-Control-Allow-Origin: http://www.ryanbao.cn
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
{% endhighlight %}

上面的头信息之中，有三个与CORS请求相关的字段，都以Access-Control- 开头

* Access-Control-Allow-Origin : 该字段是必须的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求
* Access-Control-Allow-Credentials: 该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。
* Access-Control-Expose-Headers:该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。

withCredentials 属性

上面说到，CORS请求默认不发送Cookie和HTTP认证信息。如果要把Cookie发到服务器，一方面要服务器同意，指定Access-Control-Allow-Credentials字段。

另一方面，开发者必须在AJAX请求中打开withCredentials属性。

{% highlight ruby %}
$.ajax({
	...
	xhrFields: {
		withCredentials: true    // 前端设置是否带cookie
	},
	crossDomain: true,   		// 会让请求头中包含跨域的额外信息，但不会含cookie
	...
});
{% endhighlight %}

否则，即使服务器同意发送Cookie，浏览器也不会发送。或者，服务器要求设置Cookie，浏览器也不会处理。

但是，如果省略withCredentials设置，有的浏览器还是会一起发送Cookie。这时，可以显式关闭withCredentials。
需要注意的是，如果要发送Cookie，Access-Control-Allow-Origin就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的document.cookie也无法读取服务器域名下的Cookie。


如果Origin指定的源，不在许可范围内，服务器会返回一个正常的HTTP回应。浏览器发现，这个回应的头信息没有包含Access-Control-Allow-Origin字段，就知道出错了，从而不会解析返回结果，抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。

##### 2.2 非简单请求

非简单请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELETE，或者Content-Type字段的类型是application/json。

非简单请求的CORS请求，会在正式通信之前，增加一次HTTP查询请求，称为"预检"请求（preflight）。浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的XMLHttpRequest请求，否则就报错。

{% highlight ruby %}
var url = 'http://api.ryanbao.com/xxx';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send();
{% endhighlight %}

浏览器发现，这是一个非简单请求，就自动发出一个"预检"请求，要求服务器确认可以这样请求。下面是这个"预检"请求的HTTP头信息。

{% highlight ruby %}
OPTIONS /xxx HTTP/1.1
Origin: http://www.ryanbao.cn
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.ryanbao.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0
...
{% endhighlight %}

"预检"请求用的请求方法是OPTIONS，表示这个请求是用来询问的。头信息里面，关键字段是Origin，表示请求来自哪个源。除了Origin字段，"预检"请求的头信息包括两个特殊字段。如下：

* Origin：与简单的请求相同
* Access-Control-Request-Method: 该字段是必须的，用来列出浏览器的CORS请求会用到哪些HTTP方法，上例是PUT。
* Access-Control-Request-Headers: （可选）该字段是一个逗号分隔的字符串，指定浏览器CORS请求会额外发送的头信息字段，上例是X-Custom-Header

服务器收到"预检"请求以后，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段以后，确认允许跨源请求，就可以做出回应

{% highlight ruby %}
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2018 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://www.ryanbao.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
{% endhighlight %}

上面的HTTP回应中，关键的是Access-Control-Allow-Origin字段，表示http://www.ryanbao.cn

如果浏览器否定了"预检"请求，会返回一个正常的HTTP回应，但是没有任何CORS相关的头信息字段。这时，浏览器就会认定，服务器不同意预检请求，因此触发一个错误，被XMLHttpRequest对象的onerror回调函数捕获。控制台会打印出如下的报错信息。

服务器回应的其他CORS相关字段如下：

{% highlight ruby %}
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
{% endhighlight %}

* Access-Control-Allow-Methods：该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。
* Access-Control-Allow-Headers：如果浏览器请求包括Access-Control-Request-Headers字段，则Access-Control-Allow-Headers字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。
* Access-Control-Allow-Credentials： 该字段与简单请求时的含义相同。
* Access-Control-Max-Age： 该字段可选，用来指定本次预检请求的有效期，单位为秒。上面结果中，有效期是20天（1728000秒），即允许缓存该条回应1728000秒（即20天），在此期间，不用发出另一条预检请求

一旦服务器通过了"预检"请求，以后每次浏览器正常的CORS请求，就都跟简单请求一样，会有一个Origin头信息字段。服务器的回应，也都会有一个Access-Control-Allow-Origin头信息字段。

{% highlight ruby %}
PUT /xxx HTTP/1.1
Origin: http://www.ryanbao.cn
Host: api.ryanbao.cn
X-Custom-Header: value
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
{% endhighlight %}

浏览器的正常CORS请求。上面头信息的Origin字段是浏览器自动添加的。下面是服务器正常的回应。

{% highlight ruby %}
Access-Control-Allow-Origin: http://api.bob.com
Content-Type: text/html; charset=utf-8
{% endhighlight %}

Access-Control-Allow-Origin字段是每次回应都必定包含的

##### 优点

* CORS 通信与同源的 AJAX 通信没有差别，代码完全一样，容易维护。
* 支持所有类型的 HTTP 请求。

##### 缺点

* 存在兼容性问题，特别是 IE10 以下的浏览器。
* 第一次发送非简单请求时会多一次请求。

##### 3.其他方法





















