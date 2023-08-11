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



