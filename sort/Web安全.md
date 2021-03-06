### 1.常见的网站漏洞有哪些？

有跨站脚本攻击（XSS）、跨站请求伪造（CSRF）、点击劫持、SQL注入、DDOS攻击、DNS劫持

### 2.简要介绍一下XSS以及XSS如何防御

跨站脚本攻击是说攻击者通过注入恶意的脚本，在用户浏览网页的时候进行攻击，比如获取cookie或者其他用户身份信息。 可以分为存储型和反射型，

- 存储型是攻击者输入一些数据并且存储到了数据库中，其他浏览者看到的时候进行攻击，
- 反射型的话.不存储在数据库中，往往表现为将攻击代码放在URL地址的请求参数中。
- 防御:

1. **转义字符** 对于用户的输入应该是永远不信任的。最普遍的做法就是转义输入输出的内容，对于引号、尖括号、斜杠进行转义。
2. **CSP** CSP 本质上就是建立白名单，开发者明确告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截是由浏览器自己实现的。我们可以通过这种方式来尽量减少 XSS 攻击。通常可以通过两种方式来开启 CSP：1.设置 HTTP Header 中的 Content-Security-Policy；2.设置 meta 标签的方式 。
3. 输入内容长度控制
4. HTTP-only Cookie: 禁止 JavaScript 读取某些敏感 Cookie，攻击者完成 XSS 注入后也无法窃取此 Cookie。
5. 验证码：防止脚本冒充用户提交危险操作。

### 3.简要介绍一下CSRF(跨站请求伪造)以及如何防御

攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的。

- 防御方法：

1. **同源检测**：验证 HTTP Referer 或者 Origin 字段；它记录了该 HTTP 请求的来源地址。
2. **在请求地址中添加 token 并验证；** 服务器下发一个随机 Token，每次发起请求时将 Token 携带上，服务器验证 Token 是否有效。
3. 在 HTTP 头中自定义属性并验证。把token放到HTTP头的自定义属性里面
4. **不让第三方网站访问到用户 Cookie。** 可以对 Cookie 设置 **SameSite** 属性。该属性表示 Cookie 不随着跨域请求发送，可以很大程度减少 CSRF 的攻击，但是该属性目前并不是所有浏览器都兼容。

### 4. DNS劫持与DNS污染

- **DNS劫持** DNS劫持就是通过劫持了DNS服务器，通过某些手段取得某域名的解析记录控制权，进而修改此域名的解析结果，导致对该域名的访问由原IP地址转入到修改后的指定IP。DNS劫持通过篡改DNS服务器上的数据返回给用户一个错误的查询结果来实现的。
- **DNS污染** DNS污染，指的是用户访问一个地址，国内的服务器(非DNS)监控到用户访问的已经被标记地址时，服务器伪装成DNS服务器向用户发回错误的地址的行为。范例，访问Youtube、Facebook之类网站等出现的状况。

### 5.简要介绍一下SQL注入？

- 所谓SQL注入，就是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令.
- 防范方法：

1. 永远不要信任用户的输入。对用户的输入进行校验，可以通过正则表达式，或限制长度；对单引号和双"-"进行转换等。
2. 永远不要使用动态拼装sql，可以使用参数化的sql或者直接使用存储过程进行数据查询存取。
3. 永远不要使用管理员权限的数据库连接，为每个应用使用单独的权限有限的数据库连接。
4. 不要把机密信息直接存放，加密或者hash掉密码和敏感的信息。
5. 应用的异常信息应该给出尽可能少的提示，最好使用自定义的错误信息对原始错误信息进行包装
6. sql注入的检测方法一般采取辅助软件或网站平台来检测，软件一般采用sql注入检测工具jsky,网站平台就有亿思网站安全平台检测工具。

### 6.什么是点击劫持？如何防范点击劫持？

- 点击劫持是一种视觉欺骗的攻击手段。攻击者将需要攻击的网站通过 iframe 嵌套的方式嵌入自己的网页中，并将 iframe 设置为透明，在页面中透出一个按钮诱导用户点击。
- 对于这种攻击方式，推荐防御方法有两种：

1. **X-FRAME-OPTIONS** 是一个 HTTP 响应头，在现代浏览器有一个很好的支持。这个 HTTP 响应头 就是为了防御用 iframe 嵌套的点击劫持攻击。该响应头有三个值可选，分别是1. DENY，表示页面不允许通过 iframe 的方式展示；2. SAMEORIGIN，表示页面可以在相同域名下通过 iframe 的方式展示； 3. ALLOW-FROM，表示页面可以在指定来源的 iframe 中展示。
2. **JS 防御** 当通过 iframe 的方式加载页面时，攻击者的网页直接不显示所有内容了.

```
<head>
  <style id="click-jack">
    html {
      display: none !important;
    }
  </style>
</head>
<body>
  <script>
    if (self == top) {
      var style = document.getElementById('click-jack')
      document.body.removeChild(style)
    } else {
      top.location = self.location
    }
  </script>
</body>
```

### 7.了解CSP吗，介绍一下？

内容安全策略 (CSP, Content Security Policy) 是一个附加的安全层，用于帮助检测和缓解某些类型的攻击，包括跨站脚本 (XSS) 和数据注入等攻击。 这些攻击可用于实现从数据窃取到网站破坏或作为恶意软件分发版本等用途。

### 8. cookie有哪些属性？

- name： 字段为一个cookie的名称。
- value： 字段为一个cookie的值。
- domain： 字段为可以访问此cookie的域名。
- path： 字段为可以访问此cookie的页面路径。
- **expires/Max-Age：** 字段为此cookie超时时间。若设置其值为一个时间，那么当到达此时间后，此cookie失效。不设置的话默认值是Session，意思是cookie会和session一起失效。当浏览器关闭(不是浏览器标签页，而是整个浏览器) 后，此cookie失效。
- size： 此cookie大小。
- **httpOnly：** 此属性可以规定只有通过HTTP（s）请求时，才会带上该 Cookie。可以防止通过JavaScript脚本获取cookie信息，有效的防止XSS攻击。
- **secure：** 此属性规定cookie只能在https协议下才能够发送到服务器。如果当前采用的是http协议，那么浏览器在接受服务器发来的cookie的时候会自动忽略Secure属性。可以在一定程度上防止信息在传递的过程中被监听捕获后信息泄漏。
- **sameSite：** 准确的说 SameSite 这个属性有两个可选值，分别是 Strict 和 Lax 。其中 Strict 为严格模式，另一个域发起的任何请求都不会携带该类型的 cookie，能够完美的阻止 CSRF 攻击，但是也可能带来了少许不便之处，例如通过一个导航网站的超链接打开另一个域的网页会因为没有携带 cookie 而导致没有登录等问题。因此 Lax 相对于 Strict 模式来说，放宽了一些。简单来说就是，用 「安全」的 HTTP 方法（GET、HEAD、OPTIONS 和 TRACE）改变了当前页面或者打开了新页面时，可以携带该类型的 cookie。



1. [什么叫点击劫持？对这种攻击有什么解决办法？](https://github.com/pwstrick/daily/issues/441)
2. [XSS是什么？对这种攻击有哪些防范办法？](https://github.com/pwstrick/daily/issues/442)
3. [请简单解释一下CSRF的攻击原理和防御手段。](https://github.com/pwstrick/daily/issues/443)
4. [什么是重放攻击？解决方案有哪些？](https://github.com/pwstrick/daily/issues/764)
5. [简要介绍一下DNS劫持和DNS污染？](https://github.com/pwstrick/daily/issues/848)
6. [简要介绍一下SQL注入？](https://github.com/pwstrick/daily/issues/849)
7. [请讲一下内容安全策略( CSP ) 。](https://github.com/pwstrick/daily/issues/850)
8. [如何阻止用户访问我网站的非公开部分？](https://github.com/pwstrick/daily/issues/861)
9. [如何预防中间人攻击？](https://github.com/pwstrick/daily/issues/865)
10. [如何应对浏览劫持？](https://github.com/pwstrick/daily/issues/866)