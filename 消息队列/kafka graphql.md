# 1.启动Kafka服务器

首先，必须启动kafka。请参阅[官方文档](https://kafka.apache.org/quickstart)了解更多信息。

Kafka使用[ZooKeeper](https://zookeeper.apache.org/)，所以如果你还没有ZooKeeper服务器，你需要先启动一个ZooKeeper服务器。

bin/zookeeper-server-start.sh config/zookeeper.properties
现在启动Kafka服务器:

```
bin/kafka-server-start.sh config/server.properties
```

#  2.创建主题

为了演示的目的，需要创建一些主题。那么，让我们创建两个名为“topic-a”和“topic-b”的主题，

只有一个分区和一个副本:

```
bin/kafka-topics.sh——create——boot -server localhost:9092——replication-factor 1——partitions 1——topic topic-a
bin/kafka-topics.sh——create——boot -server localhost:9092——replication-factor 1——partitions 1——topic topic-b
```

# 3.启动GraphQL

启动GraphQL服务器
配置非常简单:

```
 ./go-graphql-subscription-example --help
Usage:
  go-graphql-subscription-example [flags]

Flags:
  -h, --help             help for go-graphql-subscription-example
      --port uint16      The listening port (default 8000)
      --source string    The URI of the source to connect to
      --topics strings   The list of topics/stream names that subscribers can consume (default [foo])
```

运行将之前创建的两个主题公开给订阅者的应用程序:

./go-graphql-subscription-example --topics topic-a,topic-b

或者，如果先前已经构建了docker映像，则可以这样启动容器:

docker run -ti --rm -p 8000:8000 ccamel/go-graphql-subscription-example --topics topic-a,topic-b

# 4.订阅

该应用程序公开了一个图形端点，客户端可以通过该端点接收来自kafka主题的消息。

导航到“http://localhost:8000/graphiql”URL并向主题“topic-a”提交订阅。
```
subscription {
  event(on: "topic-a")
}
还可以指定要使用的偏移量id。负值有一个特殊的含义:

- ' -1 ':分区可用的最近偏移量(end)
- ' -2 ':分区最近可用的最小偏移量(开始)
subscription {
	event(on: "topic-a", at: -1)
}
此外，还可以指定筛选器表达式。然后使用的事件只与给定谓词匹配。
您可以参考[antonmedv/expr]了解用于编写谓词的语法概述。
event(
    on: "topic-a",
    at: -1,
    matching: "value > 8"
	)
}
```



# 5.消息推送

运行producer，然后在控制台输入一些消息发送给Kafka。注意，消息应该是JSON对象
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic topic-a
{ "message": "hello world !", "value": 14 }
消息将会显示在浏览器上。