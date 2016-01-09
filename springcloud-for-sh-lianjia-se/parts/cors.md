### Ajax跨域
Web浏览器对跨域（Cross Domain)请求有着严格的安全限制。  

下图是在网站stackoverflow.com 向我们的API网关api.route.dooioo.org 发起的一个跨域Ajax请求：

> $.get("http://api.route.dooioo.org/loupan/server/v1/citys")  

<br>


![跨域请求拒绝]({{book.imagePath}}/parts/chapter1/images/crossdomain-denied.png)

跨域请求会发送一个Request Header: Origin，如果服务端检测到有此Header，那么就可以断定此请求是跨域的。Origin为请求发起方的主机名。
![跨域请求头部]({{book.imagePath}}/parts/chapter1/images/crossdomian-headers.png)


