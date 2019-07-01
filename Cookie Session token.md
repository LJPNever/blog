---
title: Cookie Session token
date: 2018-11-28 09:19:58
tags:
---

### Cookie Session token

<!--more-->

​

####  一 Cookie 技术

Cookie 技术是客户端技术，程序把每个用户的数据以cookie的形式写给用户的浏览器，当用户再次访问服务器中的web资源时，就会带着数据返回，这样，web资源处理的就是用户自己的数据了

所以，cookie技术就是把用户的数据保存到浏览器（客户端）

**常用api**：

```JAVA
Cookie(Name,Value)：构造Cookie对象
  
void setPath(String url):设置cookie有效访问路径
void setMaxAge(int expiry):设置cookie的有效时间
enpiry可以为正整数，负整数和零
正整数：表示Cookie数据保存浏览器的缓存到硬盘中，数值表示保存的时间
负整数：表示Cookie数据保存到浏览器的内存中，浏览器关闭Cookie就丢失了
零：表示删除同名的Cookie数据
void setValue(String newvalue):设置cookie的值
void response.addCookie(Cookie cookie):发送cookie到浏览器保存
Cookie[] request.getCookies():服务器端接受cookie 
```

**原理**：

1. 服务器创建Cookie对象，把会话数据存储到Cookie对象中

   new Cookie("name","value");

2. 服务器发送cookie信息到浏览器

   response.addCookie(cookie)；实际上是隐藏发送一个set-cookie名称的响应头

3. 浏览器得到服务器的cookie并保存在浏览器端

4. 浏览器在下次访问服务器时，会带着cookie信息，包含在http的请求头

5. 服务器接收来自浏览器带来信息

   request.getCookies();

Cookie数据类型只能保存非中文字符串类型的。可以保存多个Cookie，但是浏览器一般只允许存放300个Cookie，每个站点最多存放20个Cookie，每个Cookie的大小限制为4KB



#### 二 Session 技术

Session是服务器端技术，利用这个技术服务器运行时可以为每一个用户的数据创建一个独享的Session对象，当用户访问服务器的Web资源时，可以把各自的数据放在Session中，当再次访问服务器中其他的WEB资源时，其他的web资源可以在从用户的Session中取出用户数据

**常用api** :

Session是HttpSession的类，用于保存会话数据（会话：客户端和服务器之间的数据传输）

```java
HttpSession session=request.getSession():创建一个Session对象
session.setAttribute("key","value") :保存会话数据
session.getAttribute("key"):取出会话数据
void setMaxInactiveInterval(int interval):设置session的有效时间,默认30分钟自动回收Session对象
String getId():得到session编号
void invalidate():销毁sesson对象
```

**原理 ：** 

 1.第一次访问创建Session对象，给Session对象分配一个唯一的ID，叫JSESSIONID。

2.把JSESSIONID作为Cookie的值发送给浏览器保存

```java
Cookie cookie = new Cookie("JSESSIONID", sessionID);
response.addCookie(cookie);
```

3.第二次访问的时候，浏览器带着JSESSIONID的cookie访问服务器
4.服务器得到JSESSIONID，在服务器的内存中搜索是否存放对应编号的session对象
5.如果找到对应编号的session对象，直接返回该对象
6.如果找不到对应编号session对象，创建新的session对象，继续走1的流程

即通过JSESSION的cookie值在服务器找session对象

#### Token

对Token认证机制有5点直接注意的地方：

- 一个Token就是一些信息的集合；
- 在Token中包含足够多的信息，以便在后续请求中减少查询数据库的几率；
- 服务端需要对cookie和HTTP Authrorization Header进行Token信息的检查；
- 基于上一点，你可以用一套token认证代码来面对浏览器类客户端和非浏览器类客户端；
- 因为token是被签名的，所以我们可以认为一个可以解码认证通过的token是由我们系统发放的，其中带的信息是合法有效的；

**原理**：

当用户登录验证成功后，通过获取token签名生成密钥信息，并传送给客户端，客户端接收到token后存储起来，客户端每次请求服务器资源时需要带着签发的Token,服务器验证请求中的token信息，如果成功，则返回数据。

**token技术的优点** ： 

- 支持跨域访问: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.

- 无状态(也称：服务端可扩展行):Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.

- 去耦: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.

- 因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。

- 性能: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.

  ​