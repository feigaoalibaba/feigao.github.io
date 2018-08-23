---
layout: post
title: HttpOnly Cookie的作用
tags:
- 防跨站脚本攻击
- 防XSS攻击
- HttpOnly
- Cookie
---

# 前言
   
   做WEB开发，经常要跟cookie打交道，经常会遭受黑客的XSS，说一下HttpOnly Cookie的作用
   
   要理解HttpOnly的作用，要先弄懂XSS攻击，即跨站脚本攻击，大伙可以Google一下看看XSS到底是什么，来自wikipedia的解释：
   
   跨网站脚本（Cross-site scripting，通常简称为XSS或跨站脚本或跨站脚本攻击）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意使用者将代码注入到网页上，其他使用者在观看网页时就会受到影响。这类攻击通常包含了HTML以及使用者端脚本语言。
   
# 问题
   举个简单栗子：某网站提供了一个留言页面，留言内容是个富文本编辑器（也就意味着可以提交html代码给server），小黑同学深情款款得提交了一个振聋发聩的意见，但是内容中包含了一段恶意javascript脚本：
```javascript
 <script>evil_script()</script>
```
   而服务端没有做任何处理。由于这个留言页面可以看到别人提交的留言内容，于是后来的留言者就可以看到小黑的留言内容，就会运行小黑的那段javascript脚本，假如这个脚本的功能是窃取用户的cookie发给小黑自己的服务器，而这个无良的网站，竟然把用户的登录名和密码都保存在了cookie中，于是一起很严重的安全事件诞生！

   要解决这个问题，可以从两方面着手：
   
   - 服务端对提交上来的留言内容做过滤，把恶意代码过滤掉
      
   - 让恶意javascript代码读取不到我们种的cookie
   
# HttpOnly

   HttpOnly的cookie就是从第二点解决方案着手的，cookie是通过http response header种到浏览器的，我们来看看其定义和设置cookie的语法：
   
## 定义
   HttpOnly是包含在Set-Cookie HTTP响应头文件中的附加标志。生成cookie时使用HttpOnly标志有助于降低客户端脚本访问受保护cookie的风险（如果浏览器支持）。
   如果某一个Cookie 选项被设置成 HttpOnly = true 的话，那此Cookie 只能通过服务器端修改，Js 是操作不了的.
   当cookie的HTTPOnly属性被设为True时（默认为false），document对象中就找不到该cookie了。自然js无法从document获取到cookie了。这样哪怕网站存在XSS漏洞，也能在大部分情况下杜绝cookie被劫持。

## 设置cookie的语法
```text
 Set-Cookie: <name>=<value>[; <Max-Age>=<age>][; expires=<date>][; domain=<domain_name>][; path=<some_path>][; secure][; HttpOnly]
```
   是一个name=value的KV，然后是一些属性，比如失效时间，作用的domain和path，最后还有两个标志位，可以设置为secure和HttpOnly，所以，我们只要加一个;HttpOnly的标志即可，比如：

```text
 Set-Cookie: USER=Ulric-UlricPass; expires=Wednesday, 09-Nov-99 23:12:40 GMT; HttpOnly 
```
   这样一来，javascript就读取不到这个cookie信息了，不过在与服务端交互的时候，Http Request包中仍然会带上这个cookie信息，即我们的正常交互不受影响。

# 应用
   所有的cookie都应该加上HTTPonly属性么？
   网站中存储sessionID的cookie一定要设置。这也是AWVS为何命名该漏洞为Session Cookie without Secure flag set的原因。一般网站应用也不会在js里操作这些敏感Cookie的，对于一些需要在应用程序中用JS操作的cookie我们就不予设置。这样就保障了Cookie信息的安全，同时的保证了网站的正常业务。
   
# 小结
   HttpOnly 是Web应用中保护cookie安全的一种方法，一般跟cookie的其它属性一起使用 如：domain、expire等。
   HttpOnly的作用是只有在通过http协议才能访问cookie数据,其它方式不可以(JS写法 document.cookie),正常的http请求(包括ajax)都可以携带cookie发送到服务器。
   
   
   