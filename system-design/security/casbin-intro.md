## [什么是Casbin?](https://casbin.org/docs/zh-CN/overview)
权限管理在几乎每个系统中都是必备的模块。如果项目开发每次都要实现一次权限管理，无疑会浪费开发时间，增加开发成本。因此，casbin库出现了。**casbin是一个强大、高效的访问控制库。支持常用的多种访问控制模型，如ACL/RBAC/ABAC等**。可以实现灵活的访问权限控制。同时，casbin支持多种编程语言，Go/Java/Node/PHP/Python/.NET/Rust。我们只需要一次学习，多处运用。

Casbin 可以：
1. 支持自定义请求的格式，默认的请求格式为<strong style="color:#DD5145">{subject, object, action}。</strong>
2. 具有访问控制模型model和策略policy两个核心概念。
3. 支持RBAC中的多层角色继承，不止主体可以有角色，资源也可以具有角色。
4. 支持内置的超级用户 例如：root 或 administrator。超级用户4可以执行任何操作而无需显式的权限声明。
5. 支持多种内置的操作符，如 keyMatch，方便对路径式的资源进行管理，如 /foo/bar 可以映射到 /foo*

Casbin 不能：
1. 身份认证 authentication（即验证用户的用户名和密码），Casbin 只负责访问控制。应该有其他专门的组件负责身份认证，然后由 Casbin 进行访问控制，二者是相互配合的关系。
2. 管理用户列表或角色列表。 Casbin 认为由项目自身来管理用户、角色列表更为合适， 用户通常有他们的密码，但是 Casbin 的设计思想并不是把它作为一个存储密码的容器。 而是存储RBAC方案中用户和角色之间的映射关系。

## 工作原理
在 Casbin 中, 访问控制模型被抽象为基于 PERM (Policy, Effect, Request, Matcher) 的一个文件。 因此，切换或升级项目的授权机制与修改配置一样简单。 您可以通过组合可用的模型来定制您自己的访问控制模型。 例如，您可以在一个model中结合RBAC角色和ABAC属性，并共享一组policy规则。

<strong style="color: #DD5145">PERM模式由四个基础（政策Policy、效果Effect、请求Request、匹配Matcher）组成，描述了资源与用户之间的关系。</strong>

### 请求
定义请求参数。基本请求是一个元组对象，至少需要**主体(访问实体)、对象(访问资源) 和动作(访问方式)**

例如，一个请求可能长这样： r={sub,obj,act}

它实际上定义了我们应该提供访问控制匹配功能的参数名称和顺序。

### 策略
定义访问策略模式。事实上，它是在政策规则文件中定义字段的名称和顺序。

例如： p={sub, obj, act} 或 p={sub, obj, act, eft}

注⚠️：如果未定义eft (policy result，只包含两种结果allow/deny)，则策略文件中的结果字段将不会被读取， 和匹配的策略结果将默认被允许。

### 匹配器
匹配请求和政策的规则。

例如： m = r.sub == p.sub && r.act == p.act && r.obj == p.obj 这个简单和常见的匹配规则意味着如果请求的参数(访问实体，访问资源和访问方式)匹配， 如果可以在策略中找到资源和方法，那么策略结果（p.eft）便会返回。 策略的结果将保存在 p.eft 中。

### 效果
它可以被理解为一种模型，在这种模型中，对匹配结果再次作出逻辑组合判断。

例如： e = some (where (p.eft == allow))

这句话意味着，如果匹配的策略结果有一些是允许的，那么最终结果为真。

让我们看看另一个示例： e = some (where (p.eft == allow)) && !some(where (p.eft == deny) 此示例组合的逻辑含义是：如果有符合允许结果的策略且没有符合拒绝结果的策略， 结果是为真。 换言之，当匹配策略均为允许（没有任何否认）是为真（更简单的是，既允许又同时否认，拒绝就具有优先地位)。

Policy effect中只有以下几种类型
![政策效果集](https://raw.githubusercontent.com/zmk-c/cloudImage/master/img/20221116204517.png)

Casbin中最基本、最简单的model是ACL。ACL中的model 配置为:

```casbin
# 在线编辑器
https://casbin.org/en/editor

# Request definition
[request_definition]
r = sub, obj, act

# Policy definition
[policy_definition]
p = sub, obj, act

# Policy effect
[policy_effect]
e = some(where (p.eft == allow))

# Matchers
[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

## 角色域
role_definiton 角色域
- g = _ , _ 表示以角色为基础
- g = _ , _ , _ 表示以域为基础（多商户模式）

Casbin中RABC（Role-Based Access Control，基于角色的访问控制）的model 配置为:
```casbin
# 在线编辑器
https://casbin.org/en/editor

# 请求入参（实体、资源、方法）
[request_definition]
r = sub, obj, act

# 策略（实体、资源、方法）
[policy_definition]
p = sub, obj, act

# 相比ACL 多了一个角色域
[role_definition]
g = _, _ # 这里的意思是g收两个参数 g=用户,角色

# 经过下面的匹配规则匹配后的bool值中是否有一条等于allow的
[policy_effect]
e = some(where (p.eft == allow))

# 这里的g 有点类似转换器的意思 r.sub被p.sub这个角色替换
[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

多商户模型的model 配置为：
```casbin
# 在线编辑器
https://casbin.org/en/editor

[request_definition]
r = sub, dom, obj, act

[policy_definition]
p = sub, dom, obj, act

[role_definition]
g = _, _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub, r.dom) && r.dom == p.dom && r.obj == p.obj && r.act == p.act
```

## casbin应用于golang中
第一种：使用配置文件的方式
其中，model.conf中的文件内容如下：
```conf
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```
policy.csv中的文件内容如下：
```csv
p,zhangsan,data1,read
p,zhangsan,data1,write
```
开始使用
```golang
# 引入casbin包
go get github.com/casbin/casbin/v2
```
```golang
package main

import (
	"fmt"
	"github.com/casbin/casbin/v2"
	"log"
)

func main() {
	// 1.创建模型和policy 使用配置文件的放松
	e, err := casbin.NewEnforcer("./model.conf", "./policy.csv")

	sub := "alice" // 想要访问资源的用户
	obj := "data1" // 将要被访问的资源
	act := "read"  // 用户对资源实施的操作

  // 2.进行检查
	ok, err := e.Enforce(sub, obj, act)

	if err != nil {
		// 处理错误
		log.Fatal("err=>", err)
		return
	}

	if ok {
		// 允许 alice 读取 data1
		fmt.Println("匹配成功")
	} else {
		// 拒绝请求，抛出异常
		fmt.Println("匹配失败")
	}
}
```
第二种：将模型和策略存在数据库中 采用gorm映射 需要使用适配器
```golang
package main

import (
	"fmt"
	"github.com/casbin/casbin/v2"
	gormadapter "github.com/casbin/gorm-adapter/v3"
	_ "github.com/go-sql-driver/mysql"
	"log"
)

func main() {
	// 将策略policy保存到数据库中
	a, _ := gormadapter.NewAdapter("mysql", "root:rootroot@tcp(127.0.0.1:3306)/casbin", true) 
	e, _ := casbin.NewEnforcer("./base/demo_casbin_2/model.conf", a)

	// 添加一个策略 重复策略只添加一次
	added, err := e.AddPolicy("zhangsan", "data1", "read")
	if err != nil {
		return
	}

	if added {
		fmt.Println("添加策略入库成功")
	}

	// Load the policy from DB.
	e.LoadPolicy()

	// Check the permission.
	ok, err := e.Enforce("alice", "data1", "read")
	if err != nil {
		// 处理错误
		log.Fatal("err=>", err)
		return
	}

	if ok {
		fmt.Println("匹配成功")
	} else {
		fmt.Println("匹配失败")
	}

	// Modify the policy. 即增删改查policy
	// e.AddPolicy(...)
	// e.RemovePolicy(...)

	// Save the policy back to DB.
	e.SavePolicy()
}

```

## 参考
- Casbin官方文档 - https://casbin.org/docs/zh-CN/overview