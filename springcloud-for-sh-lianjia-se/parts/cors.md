### Ajax跨域
Web浏览器对跨域（Cross Domain)请求有着严格的安全限制。  

下图是在网站http://stackoverflow.com 向 http://api.route.dooioo.org 发起的一个跨域Ajax请求：


![跨域请求拒绝]({{book.imagePath}}/parts/chapter1/images/crossdomain-denied.png)

跨域请求会额外发送一个Request Header: Origin，如果服务端检测到有此Header，那么就可以断定此请求是跨域的。
