---
title: 浏览器缓存
date: 2018-07-04 15:33:22
tags: http
categories: http
---

浏览器缓存（Browser Caching）是为了节约网络的资源加速浏览，浏览器在用户磁盘上对最近请求过的文档进行存储，当访问者再次请求这个页面时，浏览器就可以从本地磁盘显示文档，这样就可以加速页面的阅览。
<!-- more -->

在浏览器第一次发起请求时，本地无缓存，向web服务器发送请求，服务器起端响应请求，浏览器端缓存。过程如下图所示：

![第一次请求过程](/image/browser-cache/browser-cache-1.png)

在第一次请求时，服务器会将页面最后修改时间通过Last-Modified标识由服务器发送给客户端，客户端记录修改时间；服务器还会生成一个Etag，并发送给客户端。

浏览器后续再次进行请求时：

![浏览器再次请求](/image/browser-cache/browser-cache-2.png)

# 缓存的优缺点

## 优点
1. 减少了冗余的数据传输，节省了网络带宽流量；
2. 减少了服务器的负担，大大提升了网站的性能；
3. 加快了客户端加载网页的速度，提高了页面的响应速度。

## 缺点
客户端和服务端交互的时候，服务端的数据虽然变了，但是页面缓存没有改变，对于相同的url，ajax提交过去以后，浏览器是从缓存中拿数据，这种情况肯定是不被允许的。

*****

# 缓存机制

浏览器缓存主要有两类：**强缓存**和**协商缓存**

## 强缓存
就是缓存中已经有了请求数据的时候，客户端直接从缓存中获取数据，只有当缓存中没有请求数据的时候，客户端才会从服务端拿取数据。
强缓存是利用http的返回头中的Expires或者Cache-Control两个字段来控制的，用来表示资源的缓存时间。

### Expires
该字段是http1.0时的规范，它的值为一个绝对时间的GMT格式的时间字符串，比如Expires:Mon,18 Oct 2066 23:59:59 GMT。这个时间代表着这个资源的失效时间，在此时间之前，即命中缓存。这种方式有一个明显的缺点，由于失效时间是一个绝对时间，所以当服务器与客户端时间偏差较大时，就会导致缓存混乱。

### Cache-Control
Cache-Control是http1.1时出现的header信息，主要是利用该字段的max-age值来进行判断，它是一个相对时间，例如Cache-Control:max-age=3600，代表着资源的有效期是3600秒。
Cache-Control有很多产物，不同的属性代表的意义不同。

* max-age=t：缓存内容在t秒后失效

* private： 只能被终端用户的浏览器缓存，不允许CDN等中继缓存服务器对其缓存

* public： 可以被所有的用户缓存，包括终端用户和CDN等中间代理服务器

* no-cache：需要使用协商缓存来验证缓存数据

* no-store：所有内容不使用缓存

## 协商缓存
协商缓存就是由服务器来确定缓存资源是否可用，所以客户端与服务器端要通过某种标识来进行通信，从而让服务器判断请求资源是否可以缓存访问，这主要涉及到下面两组header字段，这两组搭档都是成对出现的，即第一次请求的响应头带上某个字段（Last-Modified或者Etag），则后续请求则会带上对应的请求字段（If-Modified-Since或者If-None-Match），若响应头没有Last-Modified或者Etag字段，则请求头也不会有对应的字段。

下面是一些协商缓存的缓存方案：

### Last-Modified
#### Last-Modified
服务端在响应请求时，会返回资源的最后修改时间

#### If-Modified-Since
客户端再次请求服务端的时候，请求头会包含这个字段，后面跟着在缓存中获取的资源的最后修改时间。服务端收到请求发现此请求头中有If-Modified-Since字段，会与被请求资源的最后修改时间进行对比，如果一致则会返回304和响应报文头，浏览器从缓存中获取数据即可。从字面上看，就是说从某个时间节点开始看，是否被修改了，如果被修改了，就返回整个数据和200 OK，如果没有被修改，服务端只要返回响应头报文，304 Not Modified.

#### If-Unmodified-Since
和If-Modified-Since相反，就是说从某个时间点开始看，是否没有被修改.如果没有被修改，就返回整个数据和200 OK，如果被修改了，不传输和返回412 Precondition failed (预处理错误)

If-Modified-Since和If-Unmodified-Since区别就是一个是修改了返回数据一个是没修改返回数据。

Last-Modified也有缺点，就是说服务端的资源只是改了下修改时间，但是其实里面的内容并没有改变，会因为Last-Modified发生了改变而返回整个数据，为了解决这个问题，http1.1推出了Etag

### Etag
#### Etag
服务端响应请求时，通过此字段告诉客户端当前资源在服务端生成的唯一标识（生成规则由服务端决定）

#### If-None-Match
再次请求服务端的时候，客户端的请求报文头部会包含此字段，后面的值是从缓存中获取的标识，服务端接收到报文后发现If-None-Match则与被请求的资源的唯一标识对比。如果相同，说明资源不用修改，则响应header，客户端直接从缓存中获取数据，返回状态码304，如果不同，说明资源被改过，返回整个数据，200 OK。

但是实际应用中由于Etag的计算是使用算法计算出来的，而算法会占用服务端的资源，所有服务端的资源都是宝贵的，所以很少使用Etag。

现在顺便说一下不同的刷新的请求执行过程哈

* 浏览器直接输入url，回车
浏览器发现缓存中有这个文件了，不用继续请求了，直接去缓存中拿（最快）

* F5
告诉浏览器，去服务端看下文件是否过期了，于是浏览器发了一个请求带上If-Modified-Since

* Ctrl+F5
告诉浏览器，先把缓存删了，再去服务端请求完整的资源文件过来，于是浏览器就完成了强制更新的操作


这两种缓存机制可以同时存在，不过强制缓存的优先级高于协商缓存。

# 缓存解决方案
1. 在Ajax的 URL 参数后加上 "?sa=" + Math.random(); //当然这里参数 sa 可以任意取了
5. 第2种方法和第1种类似，在 URL 参数后加上 "?timestamp=" + new Date().getTime();
1. 在ajax发送请求前加上 xmlHttpRequest.setRequestHeader(“Cache-Control”,”no-cache”);
3. 在ajax发送请求前加上 xmlHttpRequest.setRequestHeader(“If-Modified-Since”,”0″);
2. 在服务端加 header(“Cache-Control: no-cache, must-revalidate”);
6. 用POST替代GET：不推荐
7. jQuery提供一个防止ajax使用缓存的方法：
```javascript
<script type="text/javascript" language="javascript">
     $.ajaxSetup ({
           cache: false //close AJAX cache
      });
</script>
```
8. 修改load 加载的url地址，如在url 多加个时间参数就可以：
```javasript
function loadEventInfoPage(eventId){

    $.ajaxSetup ({
       cache: true // AJAX cache  下面加上时间后load的页面中的js、css图片等都会重新加载，

         //加上这句action会重新加载，但是js、css、图片等会走缓存
    });
    $("#showEventInfo").load(ctx + "/custEvents/viewEvent.action",  {"complaint.Id":eventId, "tt":(new Date()).getTime()},function(){})
}
```
9. 设置html的缓存
```html
<META HTTP-EQUIV="Pragma" CONTENT="no-cache">
<META HTTP-EQUIV="Cache-Control" CONTENT="no-cache">
<META HTTP-EQUIV="Expires" CONTENT="0">
```

