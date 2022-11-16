## 什么是 JWT?

JWT （JSON Web Token） 是目前最流行的跨域认证解决方案，是一种基于 Token 的认证授权机制。 从 JWT 的全称可以看出，JWT 本身也是 Token，一种规范化之后的 JSON 结构的 Token。

JWT 自身包含了身份验证所需要的所有信息，因此，我们的服务器不需要存储 Session 信息。这显然增加了系统的可用性和伸缩性，大大减轻了服务端的压力。

可以看出，**JWT 更符合设计 RESTful API 时的「Stateless（无状态）」原则** 。

并且， 使用 JWT 认证可以有效避免 CSRF 攻击，因为 JWT 一般是存在在 localStorage 中，使用 JWT 进行身份验证的过程中是不会涉及到 Cookie 的。

我在 [JWT 优缺点分析](./advantages&disadvantages-of-jwt.md)这篇文章中有详细介绍到使用 JWT 做身份认证的优势和劣势。

下面是 [RFC 7519](https://tools.ietf.org/html/rfc7519) 对 JWT 做的较为正式的定义。

> JSON Web Token (JWT) is a compact, URL-safe means of representing claims to be transferred between two parties. The claims in a JWT are encoded as a JSON object that is used as the payload of a JSON Web Signature (JWS) structure or as the plaintext of a JSON Web Encryption (JWE) structure, enabling the claims to be digitally signed or integrity protected with a Message Authentication Code (MAC) and/or encrypted. ——[JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)

## JWT 由哪些部分组成？

![此图片来源于：https://supertokens.com/blog/oauth-vs-jwt](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/javaguide/system-design/jwt/jwt-composition.png)

JWT 本质上就是一组字串，通过（`.`）切分成三个为 Base64 编码的部分：

- **Header** : 描述 JWT 的元数据，定义了生成签名的算法以及 `Token` 的类型。
- **Payload** : 用来存放实际需要传递的数据
- **Signature（签名）** ：服务器通过 Payload、Header 和一个密钥(Secret)使用 Header 里面指定的签名算法（默认是 HMAC SHA256）生成。

JWT 通常是这样的：`xxxxx.yyyyy.zzzzz`。

示例：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

你可以在 [jwt.io](https://jwt.io/) 这个网站上对其 JWT 进行解码，解码之后得到的就是 Header、Payload、Signature 这三部分。

Header 和 Payload 都是 JSON 格式的数据，Signature 由 Payload、Header 和 Secret(密钥)通过特定的计算公式和加密算法得到。

![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/javaguide/system-design/jwt/jwt.io.png)

### Header

Header 通常由两部分组成：

- `typ`（Type）：令牌类型，也就是 JWT。
- `alg`（Algorithm） ：签名算法，比如 HS256。

示例：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

JSON 形式的 Header 被转换成 Base64 编码，成为 JWT 的第一部分。

### Payload

Payload 也是 JSON 格式数据，其中包含了 Claims(声明，包含 JWT 的相关信息)。

Claims 分为三种类型：

- **Registered Claims（注册声明）** ：预定义的一些声明，建议使用，但不是强制性的。
- **Public Claims（公有声明）** ：JWT 签发方可以自定义的声明，但是为了避免冲突，应该在 [IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml) 中定义它们。
- **Private Claims（私有声明）** ：JWT 签发方因为项目需要而自定义的声明，更符合实际项目场景使用。

下面是一些常见的注册声明：

- `iss`（issuer）：JWT 签发方。
- `iat`（issued at time）：JWT 签发时间。
- `sub`（subject）：JWT 主题。
- `aud`（audience）：JWT 接收方。
- `exp`（expiration time）：JWT 的过期时间。
- `nbf`（not before time）：JWT 生效时间，早于该定义的时间的 JWT 不能被接受处理。
- `jti`（JWT ID）：JWT 唯一标识。

示例：

```json
{
  "uid": "ff1212f5-d8d1-4496-bf41-d2dda73de19a",
  "sub": "1234567890",
  "name": "John Doe",
  "exp": 15323232,
  "iat": 1516239022,
  "scope": ["admin", "user"]
}
```

Payload 部分默认是不加密的，**一定不要将隐私信息存放在 Payload 当中！！！**

JSON 形式的 Payload 被转换成 Base64 编码，成为 JWT 的第二部分。

### Signature

Signature 部分是对前两部分的签名，作用是防止 JWT（主要是 payload） 被篡改。

这个签名的生成需要用到：

- Header + Payload。
- 存放在服务端的密钥(一定不要泄露出去)。
- 签名算法。

签名的计算公式如下：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（`.`）分隔，这个字符串就是 JWT 。

## golang中是如何使用JWT的？
### JWT加密
``` golang
package main

import (
	"fmt"
	"github.com/dgrijalva/jwt-go" // 需要导入这个包
	"log"
	"time"
)

/**
jwt是一种后端不做存储的前端身份验证工具
分为三部分：Header Payload Signature
*/

type MyClaim struct {
	UserName string `json:"user_name"`
	jwt.StandardClaims
}

var (
	TokenSecret = []byte("wo_shi_zmk") // token加密的key
)

func main() {
	/*jwt加密部分*/

	// Create the Claims
	claims := MyClaim{
		UserName: "Krade",
		StandardClaims: jwt.StandardClaims{
			ExpiresAt: time.Now().Add(time.Hour * 2).Unix(), // 过期时间 常见的字段 这里设置为两小时后
			Issuer:    "zmk",                                // 签发者
		},
	}

	// 带有参数的生成 通常使用NewWithClaims(加密算法，结构体MapClaims) 因为我们可以通过匿名结构体来实现Claims接口 从而可以携带自己的参数
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

	//  签名Signature       头部Header                Payload
	fmt.Println("token中的内容为=>", token) // { 0x14000128180 map[alg:HS256 typ:JWT] {Krade { 15000  0 zmk 0 }}  false}

	// 对token传给前端时进行加密 这里的signedString就可以给前端使用了 前端存在浏览器缓存中 请求后端时放在头部携带中丢回来
	signedString, err := token.SignedString(TokenSecret)
	if err != nil {
		log.Fatalf("加密失败[%v]", err)
		return
	}
	fmt.Println("加密输出的内容为", signedString) // eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJLcmFkZSIsImV4cCI6MTY2ODUyNzcxMiwiaXNzIjoiem1rIn0.0H2T_iQqN8Xzh7jWIm8kbATnbDvaMqCd4EnTWcyid0k
}

```

### JWT解析
```golang
package main

import (
	"fmt"
	"github.com/dgrijalva/jwt-go"
	"log"
)

type MyClaim struct {
	UserName string `json:"user_name"`
	jwt.StandardClaims
}

var (
	TokenSecret = []byte("wo_shi_zmk") // token加密的key
)

func main() {
	/*jwt解密部分*/

	// 待解密的字符串
	tokenString := "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJLcmFkZSIsImV4cCI6MTY2ODUyNzcxMiwiaXNzIjoiem1rIn0.0H2T_iQqN8Xzh7jWIm8kbATnbDvaMqCd4EnTWcyid0k"

	// 解析jwt
	// 先使用指定key 将字符串解密成Claim结构体
	token, err := jwt.ParseWithClaims(tokenString, &MyClaim{}, func(token *jwt.Token) (interface{}, error) {
		return TokenSecret, nil
	})

	if err != nil {
		log.Fatalln("解析失败", err)
		return
	}

	fmt.Printf("解析出来的内容为:%v\n 后端需要使用的是jwt的claim:%v\n", token, token.Claims) // &{eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJLcmFkZSIsImV4cCI6MTY2ODUyNzcxMiwiaXNzIjoiem1rIn0.0H2T_iQqN8Xzh7jWIm8kbATnbDvaMqCd4EnTWcyid0k 0x1400012[alg:HS256 typ:JWT] 0x1400015c000 0H2T_iQqN8Xzh7jWIm8kbATnbDvaMqCd4EnTWcyid0k true}

	// 对claim做个断言 token.Valid经过验证
	if claim, ok := token.Claims.(*MyClaim); ok && token.Valid {
		fmt.Println("UserName=>", claim.UserName)
	}

}

```

## 如何基于 JWT 进行身份验证？

在基于 JWT 进行身份验证的的应用程序中，服务器通过 Payload、Header 和 Secret(密钥)创建 JWT 并将 JWT 发送给客户端。客户端接收到 JWT 之后，会将其保存在 Cookie 或者 localStorage 里面，以后客户端发出的所有请求都会携带这个令牌。

![JWT身份验证示意图](./images/jwt.svg)

简化后的步骤如下：

1. 用户向服务器发送用户名、密码以及验证码用于登陆系统。
2. 如果用户用户名、密码以及验证码校验正确的话，服务端会返回已经签名的 Token，也就是 JWT。
3. 用户以后每次向后端发请求都在 Header 中带上这个 JWT 。
4. 服务端检查 JWT 并从中获取用户相关信息。

两点建议：

1. 建议将 JWT 存放在 localStorage 中，放在 Cookie 中会有 CSRF 风险。
2. 请求服务端并携带 JWT 的常见做法是将其放在 HTTP Header 的 `Authorization` 字段中（`Authorization: Bearer Token`）

## 如何防止 JWT 被篡改？

有了签名之后，即使 JWT 被泄露或者解惑，黑客也没办法同时篡改 Signature 、Header 、Payload。

这是为什么呢？因为服务端拿到 JWT 之后，会解析出其中包含的 Header、Payload 以及 Signature 。服务端会根据 Header、Payload、密钥再次生成一个 Signature。拿新生成的 Signature 和 JWT 中的 Signature 作对比，如果一样就说明 Header 和 Payload 没有被修改。

不过，如果服务端的秘钥也被泄露的话，黑客就可以同时篡改 Signature 、Header 、Payload 了。黑客直接修改了 Header 和 Payload 之后，再重新生成一个 Signature 就可以了。

**密钥一定保管好，一定不要泄露出去。JWT 安全的核心在于签名，签名安全的核心在密钥。**

## 如何加强 JWT 的安全性？

1. 使用安全系数高的加密算法。
2. 使用成熟的开源库，没必要造轮子。
3. JWT 存放在 localStorage 中而不是 Cookie 中，避免 CSRF 风险。
4. 一定不要将隐私信息存放在 Payload 当中。
5. 密钥一定保管好，一定不要泄露出去。JWT 安全的核心在于签名，签名安全的核心在密钥。
6. Payload 要加入 `exp` （JWT 的过期时间），永久有效的 JWT 不合理。并且，JWT 的过期时间不易过长。
7. ......