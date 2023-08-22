# 手动部署

## 1.访问官网

https://replit.com/~

注册登录

## 2.创建Repl

![image-20230822141056911](./assets/image-20230822141056911.png)

## 3.清空

![image-20230822141234569](./assets/image-20230822141234569.png)

![image-20230822141259998](./assets/image-20230822141259998.png)

## 4.下载项目

![image-20230822141316946](./assets/image-20230822141316946.png)

在终端复制以下命令执行。

```Go
curl -fsSL fireboom.io/install.sh | bash -s fb-project -t fb-init-todo
```

![image-20230822141332305](./assets/image-20230822141332305.png)

## 5.飞布和Replit配置修改

执行完之后启动会报错，需要去修改相关配置。

![image-20230822141418209](./assets/image-20230822141418209.png)

在store/object/global_system_config.json中将将apiListenHost改为0.0.0.0

![image-20230822141435644](./assets/image-20230822141435644.png)

显示隐藏文件，目的是为了暴露端口

![image-20230822141454375](./assets/image-20230822141454375.png)

![image-20230822141514237](./assets/image-20230822141514237.png)

官方文档：https://docs.replit.com/programming-ide/configuring-repl

选择.replit文件，增加端口配置。

这里是将飞布的9123端口映射到5900，将9991端口映射到8099

```Go
[[ports]]
localPort = 9123
externalPort = 5900

[[ports]]
localPort = 9991 
externalPort = 8099
```

![image-20230822141539522](./assets/image-20230822141539522.png)

保存，重启项目

![image-20230822141555865](./assets/image-20230822141555865.png)

## 6.访问

9123对应5900

9991对应8099

点击，在域名后面分别加上5900或者8099

![image-20230822141615805](./assets/image-20230822141615805.png)

![image-20230822141630142](./assets/image-20230822141630142.png)

![image-20230822141644216](./assets/image-20230822141644216.png)

## 7.绑定git账号

```Go
git config --global user.name "xxx"
git config --global user.email "xxx@qq.com"

mkdir ~/.ssh
cd ~/.ssh
ssh-keygen -t rsa -C xxxx@qq.com
```

根据提示按回车，3次回车，生成私钥和公钥。私钥自己保留，公钥需要告诉我们要访问的ssh服务器，也就是git服务器。 生成的私钥和公钥： 私钥：.ssh/id-rsa 公钥：.ssh/id_rsa.pub

将公钥拷贝给git服务器

推送

```Go
git init 
git add .
git commit -m "xxx"
git remote add origin url
git push origin main
```

# 通过github仓库部署

## 1.在github新建飞布项目

## 2.从github导入

![image-20230822141701642](./assets/image-20230822141701642.png)

![image-20230822141715327](./assets/image-20230822141715327.png)

导入情况如图

![image-20230822141726520](./assets/image-20230822141726520.png)

## 3.修改相关配置

在update-fb.sh增加如下命令,目的是以开发模式运行飞布

![image-20230822141740043](./assets/image-20230822141740043.png)

新增端口配置，暴露端口

![image-20230822141753393](./assets/image-20230822141753393.png)

## 4.运行

在命令行执行sh update-fb.sh

![image-20230822141807264](./assets/image-20230822141807264.png)

## 5.访问

在域名后面添加5900，8099

![image-20230822141902511](./assets/image-20230822141902511.png)

![image-20230822141923055](./assets/image-20230822141923055.png)

# 钩子使用

在控制台界面启动钩子

![image-20230822142030085](./assets/image-20230822142030085.png)

在命令行依次执行以下命令，等待钩子启动

```Go
cd custom-go
go mod tidy
go run main.go
```

![image-20230822142054342](./assets/image-20230822142054342.png)

将api复制到浏览器运行

![image-20230822142113816](./assets/image-20230822142113816.png)

