# fb-oidc

Fireboom 提供的简化版OIDC

## 介绍

本项目对casdoor进行了精简，符合OIDC协议，实现了密码登录和短信登录

## 注意事项

fork 本项目后，注意在 `object.jwks.go` 中需要初始化 cert, 启动项目后在 `object.jwks.go` 程序初始化时会查找项目根目录下的两个文件（用于OIDC服务发现的private key 和 certificate），如果没有会在初始化时生成这两个文件

```
func init() {
	cert = new(Cert)
	permPath, _ := util.GetAbsolutePath("token_jwt_key.pem")
	keyPath, _ := util.GetAbsolutePath("token_jwt_key.key")

	pemData, err := getFileData(permPath)
	if err != nil {
		// 生成 cert
		certStr, keyStr := generateRsaKeys(4096, 20, "fbCert", "fireboom")
		cert.Certificate = []byte(certStr)
		cert.PrivateKey = []byte(keyStr)
		// 生成文件
		go generateCertFile(permPath, cert.Certificate)
		go generateCertFile(keyPath, cert.PrivateKey)
		log.Info("已生成cert文件...")
	} else {
		cert.Certificate = pemData
		key, err := getFileData(keyPath)
		if err != nil {
			log.Error(err.Error())
		}
		cert.PrivateKey = key
	}
}
```



### 更改 Mysql 数据源

执行sql包下的DDL.sql，创建数据库表，然后在conf文件夹下面将config.yaml文件里面的mysql的配置更改为自己的配置。

```
mysql:
  host: "localhost"
  port: 3306
  user: "root"
  password: "123456"
  dbname: "main"
  max_open_conns: 200
  max_idle_conns: 50
```

### 登录认证

我们的演示服务基于 Casdoor 进行了精简，自研部署了一套符合 OIDC 协议的登录认证服务，将其作为 OpenAPI数据源注册到飞步，支持手机短信登录和密码登录

OIDC 是一种用于身份验证和授权的开放式协议。它建立在OAuth 2.0基础上，并为第三方应用程序提供了一种方便的方法来验证用户身份并获取用户信息，例如名称、邮件地址等。OIDC还支持单点登录（SSO），以便用户只需在一个地方登录，就可以访问多个应用程序。

可能刚入门的同学对OIDC并不了解，在传统的单体服务架构中，我们对接口的安全控制只需要使用JWT进行签发token，在后端对需要安全拦截的接口进行token认证就可以了，在Java SpringBoot项目中可以很方便地使用一些框架或者类库（如Shiro/SpringSecurity等）实现这一点。但是在分布式架构中，我们的服务可能还要接入第三方服务商，比如使用微信、QQ登录，此时token认证商是三方的服务，这个时候就有可能需要我们提供token认证的方法给三方服务。cert通常代表着公私钥对中的私钥，用于对JWT进行签名，验证Token时使用公钥进行解密和验证。那么我们是需要把公钥暴露给三方服务的。在飞布控制台我们可以去按照一定的规范添加身份验证商。详细请见角色和权限控制


## 角色和权限控制

- 前端控制：

  用户登录时返回动态路由、用户信息、用户权限等数据，前端根据这些数据动态地生成菜单，保存用户数据。比如用户admin具有管理员和普通用户两种角色（注意：用户是可以拥有多种角色的），登录后根据用户角色查询所具有的权限列表，返回给前端，前端使用directive指令判断用户权限是否存在，从而决定是否显示操作对应的按钮或链接等。

- 身份验证：

  参考官方文档：https://ansons-organization.gitbook.io/product-manual/kai-fa-wen-dang/yan-zheng-he-shou-quan/shen-fen-yan-zheng

  根据 casdoor 提供的路由信息在飞布控制台添加并配置身份验证

  ![image-20230710164515669](https://zql-oss1.oss-cn-nanjing.aliyuncs.com/notes/image-20230710164515669.png)

  飞布根据 Issuer 生成服务发现地址，其中 http://127.0.0.1:10021/ 为 casdoor 服务地址，http://127.0.0.1:10021/.well-known/openid-configuration 为服务发现地址，返回数据中定义了包括但不限于：

  - 用户端点url：userinfo_endpoint
  - jwks访问url：jwks_uri

  飞布根据这些信息解析出`jwks_uri`后，就可以对请求进行`token`认证

  补充： 

  >JWKS (JSON Web Key Set) 代表了用于身份验证和授权的一组JSON格式的Web密钥。它是OAuth 2.0和OpenID Connect等身份验证和授权协议中的一部分。JWKS通常用于在安全的方式下公开API的公钥和其他安全参数。
  >
  >JWKS包含一个包含一组密钥的JSON对象，每个密钥都有一个唯一的标识符（kid），并提供其算法（alg）、密钥类型（kty）和公钥（用于验证签名）或私钥（用于生成签名）等信息。通常，JWKS中的密钥是使用非对称加密算法生成的，如RSA或ECDSA。
  >
  >在OAuth 2.0和OpenID Connect中，JWKS用于验证由身份提供者（如认证服务器）签名的访问令牌或ID令牌。客户端可以通过从指定的JWKS端点获取JWKS并验证签名，来确保令牌的身份验证和完整性。
  >
  >总结来说，JWKS是一个存储公钥和其他安全参数的JSON对象，用于身份验证和授权协议中的安全验证和签名操作。

  

  **如何通过自定义的身份验证提供商进行请求拦截校验呢？**

  飞布提供了全局的身份验证钩子，可以在钩子函数中进行操作

  ![image-20230711133357144](https://zql-oss1.oss-cn-nanjing.aliyuncs.com/notes/image-20230711133357144.png)

  这里的 `mutatingPostAuthentication` 钩子功能如下：

  用户身份验证后，会调用此函数，可以在此函数中手动控制用户身份验证结果，返回 ok 表示验证通过，deny 表示验证失败。 验证通过时，可以修改返回的user对象，如果不需要修改，则直接返回入参的user即可。

  其他钩子请参考官方文档：[fireboom.io](https://ansons-organization.gitbook.io/product-manual/kai-fa-wen-dang/gou-zi-ji-zhi/node-gou-zi#mutatingpostauthentication)

  在此钩子中，飞布对请求携带的Authorization请求头进行解析，通过前面我们配置的身份验证提供商获取用户信息，并将其传入钩子函数中，我们可以在这个地方查询用户的所属角色，填充用户的角色属性，这样就可以与飞布自带的RBAC指令进行串联，从而实现请求拦截校验的目的。

- 飞布RBAC指令

  在飞布中创建的每一个API都存储在store/list/FbOperation文件中，其中记录了每个API的信息，格式如下：

  ```
  {
  		"content": "mutation QueryRaw($query: String!) {\n      main_queryRaw(query: $query)\n}",
  		"createTime": "2023-06-25 05:44:48Z",
  		"deleteTime": "",
  		"enabled": true,
  		"id": 2019540587708416,
  		"illegal": false,
  		"isPublic": true,
  		"liveQuery": false,
  		"method": "POST",
  		"operationType": "mutations",
  		"remark": "",
  		"restUrl": "",
  		"roleType": "requireMatchAny",
  		"roles": "[\"admin\"]",
  		"title": "/Statistics/QueryRaw",
  		"updateTime": "2023-07-07 07:43:27Z"
  }
  ```

  其中`roles`和`roleType`控制了API访问所需要的角色，`requireMatchAny` 表示只要匹配任意一种角色就可以。

  **飞布底层支持有相关的几个接口：**

  | API                       | 说明                            | 参数                     |
  | ------------------------- | ------------------------------- | ------------------------ |
  | /api/v1/operateApi/opened | 获取通过飞布创建的所有开启的API | 无                       |
  | /api/v1/role/bindApi      | 为角色绑定API                   | roleCode, apis, allRoles |
  | /api/v1/role/apis         | 获取角色已绑定的API             | code, roleType           |

  注：roleType 为可选参数，并且为枚举类型

  | roleType        | 说明                   |
  | --------------- | ---------------------- |
  | requireMatchAll | 需要匹配所有角色       |
  | requireMatchAny | 需要匹配任意一个角色   |
  | denyMatchAll    | 匹配所有角色则拒绝访问 |
  | denyMatchAny    | 匹配任一角色则拒绝访问 |

  

