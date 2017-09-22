# http之cookie & session
HTTP协议是无状态协议，所以服务器端为了记录用户的状态，就需要某些机制了
## cookie ##

1.  使用cookie来记录用户的状态

	过程：
	- 浏览器向服务器端发出请求，服务器获得请求后，生成了一个cookie对象
	- 服务器准备向浏览器发出一个http响应，并在响应头设置`set-cookie`
	- 它的值是之前设置的cookie对象，并设置:`authed:true`
	- 浏览器收到响应，发现`set-cookie`字段，所以就将该cookie存储在硬盘当中
	- 浏览器再次向服务器发出请求时，就会附上该用户对应cookie信息

2.  安全隐患问题
	cookie中有一个很敏感的信息就是authed:true;我们知道可以发送HTTP请求的不只是浏览器，很多HTTP客户端软件（包括curl、Node.js）都可以发送任意的HTTP请求，可以设置任何头字段。 假如我们直接设置Cookie字段为authed=true并发送该HTTP请求，服务器岂不是被欺骗了？**这种攻击非常容易，Cookie是可以被篡改的**

	而且cookie是**明文传输**的，主要服务器设置过一次authed:true;那么对方就很容易知道true的签名是什么，那么他就可以使用这个签名去欺骗服务器了，所以在cookie上不要放一些敏感的信息，不然的话，容易被篡改。

3.  cookie的大小
	一般只能设置4k

4.  属性
	- `Set-Cookie: name=Nicholas; expires=Sat, 02 May 2009 23:38:25 GMT`
	- domain: 指定了cookie将要被发送至哪个或哪些域中

5. 失效日期
	- 一般由服务器生成，可设置失效时间(expires)。
	- 如果在浏览器端生成，默认关闭浏览器失效。
	- 当超过cookie数量限制时，会自动删除旧的cookie，以为新建的cookie提供空间

## session ##

传递session过程
- 用户提交包含用户名和密码的表单，发送HTTP请求。
- 服务器验证用户发来的用户名密码。如果正确新建一个session对象，并用其唯一标识session，敏感数据比如`authed:true`也存放在其中
- 设置Cookie为`sessionId=xxxxxx|checksum`并发送HTTP响应， 仍然为每一项Cookie都设置**签名**
- 用户收到HTTP响应后，便看不到任何敏感数据了。在此后的请求中发送该Cookie给服务器
- 服务器收到此后的HTTP请求后，发现Cookie中有SessionID，进行放篡改验证
- 如果通过了验证，根据该ID从Redis中取出对应的用户对象， 查看该对象的状态并继续执行业务逻辑



