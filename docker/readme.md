当前镜像支持开发模式和生产模式。

- 开发模式内置了golang和nodejs环境；
- 生产模式提供了飞布运行所必须得环境；

# 使用方法

（1）拉取：docker pull fireboomapi/fireboom_server:latest 

（2）运行：

以开发模式启动，不可用于正式环境！

```
docker run -it -v 你的目录:需要挂载的目录
							 -p 9123:9123 -p 9991:9991 \
							 fireboomapi/fireboom_server:latest dev 
```

以生产模式启动

```
docker run -it -v 你的目录:需要挂载的目录
							 -p 9123:9123 -p 9991:9991 \
							 fireboom_server:latest start 
```

飞布的代码位于fbserver目录下面，可以进行挂载的目录：custom-go、 log、 template、upload、exported、 store。可以根据实际需要进行挂载

（3）访问地址：[http://localhost:9123](http://localhost:9123/)（端口9123是飞布控制台的端口，端口9991是飞布处理所有api请求的端口）

---

# Go流程

访问你自己设置的映射端口，浏览器访问如图。容器里面自带了todo数据，可以快速进行操作。

![image-20230727090041905](./assets/image-20230727090041905.png)

这里演示如何添加添加数据源，飞布支持多种数据

![image-20230726154019279](./assets/image-20230726154019279.png)

这里我选择添加MySQL，填写相关参数,点击测试，保存。

![image-20230726154223564](./assets/image-20230726154223564.png)

然后打开数据源。连接可能会有点慢，需要等待一下

![image-20230726155555952](./assets/image-20230726155555952.png)

在Api管理下面新建目录，新建api，刷新一下界面，api操作就会出现。执行相关操作。

![image-20230726160356786](./assets/image-20230726160356786.png)

开启钩子，这里我以golang为例子。容器启动的飞布钩子模版里面是空的，根据需要可以下载对应的模版。

![image-20230726163012348](./assets/image-20230726163012348.png)

这里我用golang打开容器挂载的目录，就可以看见新增的模版了

![image-20230726163921254](./assets/image-20230726163921254.png)

在左侧钩子列表里面打开想要的钩子，如图

![image-20230726164134402](./assets/image-20230726164134402.png)

ide里面对应的钩子如图，可以在你想要的钩子里面编写代码

![image-20230726164259971](./assets/image-20230726164259971.png)

编写的代码如图

![image-20230726164523750](./assets/image-20230726164523750.png)

在终端查看容器id并且进入容器，默认目录是fbserver,fbserver目录下面是飞布相关代码

![image-20230727090425710](./assets/image-20230727090425710.png)

进入custom-go，拉取依赖go mod tidy，启动。在飞布控制台刷新钩子状态，变成已启动，复制链接，到浏览器去执行

![image-20230726170833683](./assets/image-20230726170833683.png)

![image-20230726170851123](./assets/image-20230726170851123.png)



查看终端，发现有日志里面有对应的执行。其他语言是相同的流程。

![image-20230727091054323](./assets/image-20230727091054323.png)

 

# Node流程

首先先下载node模版，可以参考上面go的流程

![image-20230727091338259](./assets/image-20230727091338259.png)

和上面相同的流程，可以查询到值

![image-20230727091429236](./assets/image-20230727091429236.png)

开启钩子

![image-20230727091621585](./assets/image-20230727091621585.png)

进入容器，进入到custom-ts目录下

![image-20230727091713171](./assets/image-20230727091713171.png)

在钩子里面修改代码，使用ide打开挂载的目录，也可以在飞布控制台修改。这里演示使用飞布控制台修改

![image-20230727172233382](./assets/image-20230727172233382.png)使用npm i安装依赖，使用npm run dev启动，查看钩子状态为已启动

![image-20230727173001467](./assets/image-20230727173001467.png)

复制链接到浏览器执行会才会触发钩子

![image-20230727173040026](./assets/image-20230727173040026.png)

![image-20230727172917414](./assets/image-20230727172917414.png)
