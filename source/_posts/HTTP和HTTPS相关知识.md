---
title: HTTP和HTTPS相关知识
categories: 
- 计算机网络知识
tags: 
- 学习笔记
---
## 1. http和https的基本概念
http：*是一个客户端和服务端请求和应答的标准（TCP），用于从ＷＷＷ服务器传输超文本到本地浏览器的超文本传输协议。*
https：*是以安全为目标的HTTP通道，即HTTP下加入SSL层进行加密。其作用是：建立一个信息安全通道，来确保数据的传输，确保网站的真实性。*

## 2. http和https的区别以及优缺点
* http是超文本传输协议，信息是明文传输，https协议要比http协议安全。https是具有安全性的ssl加密传输协议，可防止数据在传输过程中被窃取、改变，确保数据的完整性。
* http默认端口为80，https的默认端口为443。
* http的连接很简单，是无状态的。https握手阶段比较费时，会使页面加载时间延长50%，增加10%～20%的耗电。
* https的缓存不如http高效，会增加数据开销。
* https协议需要ca证书，费用较高，功能越强大的证书费用越高。
* SSL证书需要绑定IP，不能在同一个IP上绑定多个域名，IPV4资源支持不了这种消耗。

## 3. https协议的工作原理
**客户端在使用https方式与web服务器通信时有以下几个步骤：**
1. 客户端使用https url访问服务器，则要求web服务器建立ssl连接。
2. web服务器接收到客户端的请求之后，会将网站的证书（证书中包含公钥），传输给客户端。
3. 客户端与web服务器端开始协商ssl连接的安全等级，也就是加密等级。
4. 客户端浏览器通过双方协商一致的安全等级，建立会话密钥，然后通过网站的公钥来加密会话密钥，并传送给网站。
5. web服务器通过自己的私钥解密出会话密钥。
6. web服务器通过会话密钥加密与客户端之间的通信。

## 4. http请求跨域问题
1. 跨域的原理
	**跨域**，是指浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的。
	**同源策略**，是浏览器对JavaScript实施的安全限制，只要<u>协议、域名、端口</u>有任何一个不同，都当作是不同的域。
	**跨域原理**，即是通过各种方式，避开浏览器的安全限制。
2. 解决方案
	* **JSONP**：
		ajax请求受同源策略的影响，不允许进行跨域请求，而script标签的src属性中的链接却可以访问跨域的js脚本，利用这个特性，服务端不再返回JSON格式的数据，而是返回一段调用某个函数的js代码，在src中实现了调用，这样实现了跨域。
		步骤：
			1. 去创建一个src标签
			2. 在script的src属性设置接口地址
			3. 接口参数，必须要带一个自定义的函数名，要不然后台无法返回数据
			4. 通过定义函数名去接受返回的数据
		``` javascript
			//动态创建 script
			var script = document.createElement('script');
		
			// 设置回调函数
			function getData(data) {
   			console.log(data);
			}
		
			//设置 script 的 src 属性，并设置请求地址
			script.src = 'http://localhost:3000/?callback=getData';
		
			// 让 script 生效
			document.body.appendChild(script);
		```
		JSONP缺点：
		JSONP只支持get，因为script标签只支持get请求，JSONP需要后端配合返回指定格式的数据。
	* **document.domian**: 基础域名相同，子域名不同
	* **window.name** 利用在一个浏览器窗口内，载入所有的域名都是共享一个window.name
	* **CORS** cross-origin-resource sharing 跨域资源共享 服务器设置对CORS的支持原理：服务器设置Access-Control-Allow-Origin HTTP请求头后，浏览器将允许跨域请求。
	* **proxy代理** 目前常用方式

## 5. Cookie、Session、Token
### Cookie
Cookie是一个非常具体的东西，指的就是浏览器里面能永久存储的一种数据，仅仅是浏览器实现的一种数据存储功能。
Cookie由服务器生成，发送给浏览器，浏览器把Cookie以kv的形式保存在某个目录下的文本文件内，下一次请求同一网站时会把Cookie发送给服务器。由于Cookie是存储在客户端的，所以浏览器加入了一些限制确保Cookie不会被恶意使用，同时不会占据太多磁盘空间，所以每个域的Cookie数量是有限的。
<img src="https://img-blog.csdnimg.cn/img_convert/db4ebdb74e6a767751f7d122844a7b0e.png"/>

### Session
服务器给每个客户端分配不同的"身份标识",然后客户端每次向服务器发请求的时候，都带上这个"身份标识",服务器就知道这个请求来自谁了。至于客户端怎么保存这个”身份标识“,可以有很多种方式，对于浏览器客户端，大家都默认采用cookie的方式。
服务器使用session把用户信息临时保存在了服务器上。这种用户信息存储方式相对cookie来说更加安全，可是session有一个缺陷，如果web服务器做了负载均衡，那么下一个操作请求到了另一台服务器的时候session会丢失。
浏览器第一次访问服务器会在服务器端生成一个session，有一个sessionid与它对应。它存储在服务器的内存中。
客户端只保存sessionid到cookie中，而不会保存session。session的销毁只能通过invalidata或超时，关闭浏览器不会关闭session。
<img src="https://img-blog.csdnimg.cn/img_convert/234a2fab9a546a7cdb6a5a115a5230a2.png"/>

### Token
基于Token的身份验证是无状态的，我们不将用户信息存在session或服务器中
1. 用户通过用户名和密码发送请求
2. 程序验证
3. 程序返回一个签名的token给客户端
4. 客户端存储token，并且用于每次发送请求
5. 服务端验证token并返回数据
<img src="https://img-blog.csdnimg.cn/img_convert/20413bdb78045e2e6c84c3f8da7b38f6.png"/>

每次请求都需要token。token应该在HTTP头部发送从而保证了Http请求无状态。我们同样通过设置服务器Access-Control-Allow-Origin: *,让服务器能接受到来自所有域的请求

**Token的优势**
* 无状态、可扩展
* 安全性
* 多平台跨域
### Session和Cookie的区别
数据存放位置不同：session数据是存在服务器中的，cookie数据存在浏览器当中。
安全程度不同：cookie放在浏览器中不是很安全，session放在服务器中相对安全。
性能使用程度不同：session放在服务器上，访问增多会占用服务器的性能，考虑到减轻服务器性能方面，应使用cookie。
数据存储大小不同：单个cookie保存的数据不能超过4K，session存储在服务端，根据服务器大小来决定。
### Token和Session的区别
token是开发定义的，session是http协议规定的
token不一定存储，session存在服务器中
token可以跨域，session不可以跨域，它是与域名绑定的