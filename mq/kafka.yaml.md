
```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://10.182.173.4:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```


```shell
$ docker-compose up -d
$ docker exec -it kafka /bin/bash
bash-4.4# kafka-topics.sh --create --topic test --zookeeper zookeeper:2181 \
--replication-factor 1 --partitions 1
```
查看刚刚创建的topic信息：
```shell
bash-4.4# kafka-topics --zookeeper zookeeper:2181 --describe --topic test
```
打开生产者发送若干条消息：
```shell
kafka-console-producer --broker-list kafka:9092 --topic test
```
当需要为消息指定 key 时，可使用如下命令：
```shell
kafka-console-producer --broker-list kafka:9092 --topic test --property parse.key=true
```

开发消费者接收消息：
```shell
kafka-console-consumer --bootstrap-server kafka:9092 --from-beginning --topic test
```



异步通信原理
观察者模式