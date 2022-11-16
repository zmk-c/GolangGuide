## 认证 (Authentication) 和授权 (Authorization)的区别是什么？

这是一个绝大多数人都会混淆的问题。首先先从读音上来认识这两个名词，很多人都会把它俩的读音搞混，所以我建议你先先去查一查这两个单词到底该怎么读，他们的具体含义是什么。

说简单点就是：

- **认证 (Authentication)：** 你是谁
- **授权 (Authorization)：** 你有权限干什么

稍微正式点的说法就是 ：

- **Authentication（认证）** 是验证您的身份的凭据（例如用户名/用户 ID 和密码），通过这个凭据，系统得以知道你就是你，也就是说系统存在你这个用户。所以，Authentication 被称为身份/用户验证。
- **Authorization（授权）** 发生在 **Authentication（认证）** 之后。授权嘛，光看意思大家应该就明白，它主要掌管我们访问系统的权限。比如有些特定资源只能具有特定权限的人才能访问比如 admin，有些对系统资源操作比如删除、添加、更新只能特定人才具有

认证 ：

![](https://img-blog.csdnimg.cn/20210604160908352.png)

授权：

![](https://img-blog.csdnimg.cn/20210604161032412.png)

这两个一般在我们的系统中被结合在一起使用，目的就是为了保护我们系统的安全性。

## RBAC 模型了解吗？

系统权限控制最常采用的访问控制模型就是 **RBAC 模型** 。

**什么是 RBAC 呢？**

RBAC 即<strong style="color: #DD5145">基于角色的权限访问控制（Role-Based Access Control）</strong>。这是一种通过角色关联权限，角色同时又关联用户的授权的方式。

简单地说：一个用户可以拥有若干角色，每一个角色又可以被分配若干权限，这样就构造成“用户-角色-权限” 的授权模型。在这种模型中，用户与角色、角色与权限之间构成了多对多的关系，如下图：
![RBAC 权限模型示意图](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20221116110042.png)

**在 RBAC 中，权限与角色相关联，用户通过成为适当角色的成员而得到这些角色的权限。这就极大地简化了权限的管理。**

本系统的权限设计相关的表如下（一共 5 张表，2 张用户建立表之间的联系）：

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/2020-11/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BE%E8%AE%A1-%E6%9D%83%E9%99%90.png)

通过这个权限模型，我们可以创建不同的角色并为不同的角色分配不同的权限范围（菜单）。

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/books%E6%9D%83%E9%99%90%E7%AE%A1%E7%90%86%E6%A8%A1%E5%9D%97.png)

通常来说，如果系统对于权限控制要求比较严格的话，一般都会选择使用 RBAC 模型来做权限控制。

## 什么是 Cookie ? Cookie 的作用是什么?

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/system-design/security/cookie-sessionId.png)

`Cookie` 和 `Session` 都是用来跟踪浏览器用户身份的会话方式，但是两者的应用场景不太一样。

维基百科是这样定义 `Cookie` 的：

> `Cookies` 是某些网站为了辨别用户身份而储存在用户本地终端上的数据（通常经过加密）。

简单来说： **`Cookie` 存放在客户端，一般用来保存用户信息**。

下面是 `Cookie` 的一些应用案例：

1. 我们在 `Cookie` 中保存已经登录过的用户信息，下次访问网站的时候页面可以自动帮你登录的一些基本信息给填了。除此之外，`Cookie` 还能保存用户首选项，主题和其他设置信息。
2. 使用 `Cookie` 保存 `SessionId` 或者 `Token` ，向后端发送请求的时候带上 `Cookie`，这样后端就能取到 `Session` 或者 `Token` 了。这样就能记录用户当前的状态了，因为 HTTP 协议是无状态的。
3. `Cookie` 还可以用来记录和分析用户行为。举个简单的例子你在网上购物的时候，因为 HTTP 协议是没有状态的，如果服务器想要获取你在某个页面的停留状态或者看了哪些商品，一种常用的实现方式就是将这些信息存放在 `Cookie`
4. ......

**原理**：客户端首次向服务器发送请求时，服务器会为其分配cookie并存放到响应消息的特定字段中，客户端收到响应消息会解析并存储该字段，在下次再发送请求时就会在请求头部带上这个字段，因此服务器就能区分不同的客户端，以及客户端配置的各种偏好要求

## Cookie 和 Session 有什么区别？

**`Session` 的主要作用就是通过服务端记录用户的状态。** 典型的场景是购物车，当你要添加商品到购物车的时候，系统不知道是哪个用户操作的，因为 HTTP 协议是无状态的。服务端给特定的用户创建特定的 `Session` 之后就可以标识这个用户并且跟踪这个用户了。

`Cookie` 数据保存在客户端(浏览器端)，`Session` 数据保存在服务器端。相对来说 `Session` 安全性更高。如果使用 `Cookie` 的一些敏感信息不要写入 `Cookie` 中，最好能将 `Cookie` 信息加密然后使用到的时候再去服务器端解密。

**那么，如何使用 `Session` 进行身份验证？**

## 如何使用 Session-Cookie 方案进行身份验证？

很多时候我们都是通过 `Session ID` 来实现特定的用户，`Session ID` 一般会选择存放在 Redis 中。举个例子：

1. 用户成功登陆系统，然后返回给客户端具有 `SessionID` 的 `Cookie` 。
2. 当用户向后端发起请求的时候会把 `Session ID` 带上，这样后端就知道你的身份状态了。 

关于这种认证方式更详细的过程如下：
![session-cookie](./images/session-cookie.svg)

1. 用户使用账号密码请求登录；
2. 服务器验证用户身份后，创建session并分配唯一标识session ID，记录用户的认证状态，并将session ID放在响应头中返回给客户端（这一步中，可以使用服务端私钥对其进行签名，非必须）；
3. 客户端解析响应头，将服务端分配的session ID保存在本地cookie中，下一次请求时会在请求头中附带cookie信息；
4. 服务端接收请求时解析头部的session ID，与本地保存的session进行对比校验请求的合法性。

使用 `Session` 的时候需要注意下面几个点：

- 依赖 `Session` 的关键业务一定要确保客户端开启了 `Cookie`。
- 注意 `Session` 的过期时间。

## 多服务器节点下 Session-Cookie 方案如何做？

Session-Cookie 方案在单体环境是一个非常好的身份认证方案。但是，当服务器水平拓展成多节点时，Session-Cookie 方案就要面临挑战了。

举个例子：假如我们部署了两份相同的服务 A，B，用户第一次登陆的时候 ，Nginx 通过负载均衡机制将用户请求转发到 A 服务器，此时用户的 Session 信息保存在 A 服务器。结果，用户第二次访问的时候 Nginx 将请求路由到 B 服务器，由于 B 服务器没有保存 用户的 Session 信息，导致用户需要重新进行登陆。

**我们应该如何避免上面这种情况的出现呢？**

有几个方案可供大家参考：

1. 某个用户的所有请求都通过特性的哈希策略分配给同一个服务器处理。这样的话，每个服务器都保存了一部分用户的 Session 信息。服务器宕机，其保存的所有 Session 信息就完全丢失了。
2. 每一个服务器保存的 Session 信息都是互相同步的，也就是说每一个服务器都保存了全量的 Session 信息。每当一个服务器的 Session 信息发生变化，我们就将其同步到其他服务器。这种方案成本太大，并且，节点越多时，同步成本也越高。
3. 单独使用一个所有服务器都能访问到的数据节点（比如缓存）来存放 Session 信息。为了保证高可用，数据节点尽量要避免是单点。

## 如果没有 Cookie 的话 Session 还能用吗？

这是一道经典的面试题！

一般是通过 `Cookie` 来保存 `Session ID` ，假如你使用了 `Cookie` 保存 `Session ID` 的方案的话， 如果客户端禁用了 `Cookie`，那么 `Session` 就无法正常工作。

但是，并不是没有 `Cookie` 之后就不能用 `Session` 了，比如你可以将 `SessionID` 放在请求的 `url` 里面`https://golangguide.cn/?Session_id=xxx` 。这种方案的话可行，但是安全性和用户体验感降低。当然，为了你也可以对 `Session ID` 进行一次加密之后再传入后端。

## 为什么 Cookie 无法防止 CSRF 攻击，而 Token 可以？

**CSRF(Cross Site Request Forgery)** 一般被翻译为 **跨站请求伪造** 。那么什么是 **跨站请求伪造** 呢？说简单用你的身份去发送一些对你不友好的请求。举个简单的例子：

小壮登录了某网上银行，他来到了网上银行的帖子区，看到一个帖子下面有一个链接写着“科学理财，年盈利率过万”，小壮好奇的点开了这个链接，结果发现自己的账户少了 10000 元。这是这么回事呢？原来黑客在链接中藏了一个请求，这个请求直接利用小壮的身份给银行发送了一个转账请求,也就是通过你的 Cookie 向银行发出请求。

```html
<a src=http://www.mybank.com/Transfer?bankId=11&money=10000>科学理财，年盈利率过万</>
```

上面也提到过，进行 `Session` 认证的时候，我们一般使用 `Cookie` 来存储 `SessionId`,当我们登陆后后端生成一个 `SessionId` 放在 Cookie 中返回给客户端，服务端通过 Redis 或者其他存储工具记录保存着这个 `SessionId`，客户端登录以后每次请求都会带上这个 `SessionId`，服务端通过这个 `SessionId` 来标示你这个人。如果别人通过 `Cookie` 拿到了 `SessionId` 后就可以代替你的身份访问系统了。

`Session` 认证中 `Cookie` 中的 `SessionId` 是由浏览器发送到服务端的，借助这个特性，攻击者就可以通过让用户误点攻击链接，达到攻击效果。

但是，我们使用 `Token` 的话就不会存在这个问题，在我们登录成功获得 `Token` 之后，一般会选择存放在 `localStorage` （浏览器本地存储）中。然后我们在前端通过某些方式会给每个发到后端的请求加上这个 `Token`,这样就不会出现 CSRF 漏洞的问题。因为，即使有个你点击了非法链接发送了请求到服务端，这个非法请求是不会携带 `Token` 的，所以这个请求将是非法的。

![](https://img-blog.csdnimg.cn/20210615161108272.png)

需要注意的是：不论是 `Cookie` 还是 `Token` 都无法避免 **跨站脚本攻击（Cross Site Scripting）XSS** 。

> 跨站脚本攻击（Cross Site Scripting）缩写为 CSS 但这会与层叠样式表（Cascading Style Sheets，CSS）的缩写混淆。因此，有人将跨站脚本攻击缩写为 XSS。

XSS 中攻击者会用各种方式将恶意代码注入到其他用户的页面中。就可以通过脚本盗用信息比如 `Cookie` 。

推荐阅读：[如何防止 CSRF 攻击？—美团技术团队](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)

## 什么是 JWT?JWT 由哪些部分组成？

[JWT基础概念详解及使用](./jwt-intro.md)

## 如何基于 JWT 进行身份验证？ 如何防止 JWT 被篡改？

[JWT基础概念详解及使用](./jwt-intro.md)

## 什么是 SSO? 

SSO(Single Sign On)即单点登录说的是用户登陆多个子系统的其中一个就有权访问与其相关的其他系统。举个例子我们在登陆了京东金融之后，我们同时也成功登陆京东的京东超市、京东国际、京东生鲜等子系统。

![SSO 示意图](./images/sso.svg)



## SSO 有什么好处？

- **用户角度** :用户能够做到一次登录多次使用，无需记录多套用户名和密码，省心。
- **系统管理员角度** : 管理员只需维护好一个统一的账号中心就可以了，方便。
- **新系统开发角度:** 新系统开发时只需直接对接统一的账号中心即可，简化开发流程，省时。

## 如何设计实现一个 SSO 系统?

[SSO单点登录详解](./sso-intro.md)

## 什么是 OAuth 2.0？

OAuth 是一个行业的标准授权协议，主要用来授权第三方应用获取有限的权限。而 OAuth 2.0 是对 OAuth 1.0 的完全重新设计，OAuth 2.0 更快，更容易实现，OAuth 1.0 已经被废弃。详情请见：[rfc6749](https://tools.ietf.org/html/rfc6749)。

实际上它就是一种授权机制，它的最终目的是为第三方应用颁发一个有时效性的令牌 Token，使得第三方应用能够通过该令牌获取相关的资源。

OAuth 2.0 比较常用的场景就是第三方登录，当你的网站接入了第三方登录的时候一般就是使用的 OAuth 2.0 协议。

另外，现在 OAuth 2.0 也常见于支付场景（微信支付、支付宝支付）和开发平台（微信开放平台、阿里开放平台等等）。

下图是 [Slack OAuth 2.0 第三方登录](https://api.slack.com/legacy/oauth)的示意图：

![](https://img-blog.csdnimg.cn/20210615151716340.png)

**推荐阅读：**

- [OAuth 2.0 的一个简单解释](http://www.ruanyifeng.com/blog/2019/04/oauth_design.html)
- [10 分钟理解什么是 OAuth 2.0 协议](https://deepzz.com/post/what-is-oauth2-protocol.html)
- [OAuth 2.0 的四种方式](http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)
- [GitHub OAuth 第三方登录示例教程](http://www.ruanyifeng.com/blog/2019/04/github-oauth.html)

## 参考

- https://blog.csdn.net/weixin_35016347/article/details/108776512
- Introduction to JSON Web Tokens：https://jwt.io/introduction
- JSON Web Token Claims：https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims
