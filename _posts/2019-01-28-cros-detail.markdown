---
layout: post
title: CROS Detail
date: 2019-01-28
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [FRONTEND,CROS]
---

#####什么是跨域

跨域是指浏览器不能执行其他网站的脚本。是由浏览器的同源策略造成的，是浏览器对JavaScript实施的安全限制。

同源策略是浏览器最核心也是最基本的安全功能，要求浏览器只被允许访问来自统一站点的资源而不是那些来自其它站点可能怀有恶意的资源；

所谓的同源是指：协议、域名和端口相同。

{% highlight ruby %}
URL: http[s]://domain:port/resources
{% highlight%}

同源策略限制了一下行为：

* Cookie LocalStorage 和 IndexDB无法跨域读取
* DOM 和 JS 对象无法获取
* Ajax请求发送不出去: ex. XMLHttpRequest同源策略，禁止使用 XHR 对象向不同源的服务器地址发起 HTTP 请求