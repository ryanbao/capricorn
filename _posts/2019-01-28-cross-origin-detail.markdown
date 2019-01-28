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

* 在请求中需要附加一个额外的 `Origin`头部，其中包含请求页面的源信息（协议、域名和端口），以便服务器根据这个头部信息来决定是否给予响应。例如：`Origin: http://www.ryanbao.cn`

* 如果服务器认为这个请求可以接受，就在 Access-Control-Allow-Origin 头部中回发相同的源信息（如果是公共资源，可以回发 * ）。例如：`Access-Control-Allow-Origin：http://www.ryanbao.cn`

* 没有这个头部或者有这个头部但源信息不匹配，浏览器就会驳回请求。正常情况下，浏览器会处理请求。注意，请求和响应都不包含 cookie 信息。

* 如果需要包含 cookie 信息，ajax 请求需要设置 xhr 的属性 withCredentials 为 true，服务器需要设置响应头部 `Access-Control-Allow-Credentials: true`。

##### 2.2 非简单请求

浏览器在发送真正的请求之前，会先发送一个 Preflight 请求给服务器，这种请求使用 OPTIONS 方法，发送下列头部：

* Origin：与简单的请求相同

* Access-Control-Request-Method: 请求自身使用的方法

* Access-Control-Request-Headers: （可选）自定义的头部信息，多个头部以逗号分隔

例如：

{% highlight ruby %}
Origin: http://www.ryanbao.cn

Access-Control-Request-Method: POST

Access-Control-Request-Headers: NCZ
{% endhighlight %}

发送这个请求后，服务器可以决定是否允许这种类型的请求。服务器通过在响应中发送如下头部与浏览器进行沟通：

{% highlight ruby %}
Access-Control-Allow-Origin: http://www.ryanbao.cn   -- 与简单的请求相同

Access-Control-Allow-Methods: GET, POST   -- 允许的方法，多个方法以逗号分隔

Access-Control-Allow-Headers: NCZ  -- 允许的头部，多个方法以逗号分隔

Access-Control-Max-Age: 1728000 -- 应该将这个 Preflight 请求缓存多长时间（以秒表示）
{% endhighlight %}

一旦服务器通过 Preflight 请求允许该请求之后，以后每次浏览器正常的 CORS 请求，就都跟简单请求一样了。

##### 优点

* CORS 通信与同源的 AJAX 通信没有差别，代码完全一样，容易维护。
* 支持所有类型的 HTTP 请求。

##### 缺点

* 存在兼容性问题，特别是 IE10 以下的浏览器。
* 第一次发送非简单请求时会多一次请求。

##### 3.其他方法





















