# kafka-steam-demo setup
这个demo是 Apache kafka提供的wordcount 的demo，统计topic streams-plaintext-input中的单词个数，并输出到topic streams-wordcount-output中
## Setup preparation

* kafka 2.1.0 (with zookeeper)


## Start demo
1. start zookeeper server
</br></br>windows:
```s
> .\bin\windows\zookeeper-server-start.bat config/zookeeper.properties
```
2. start kafka server
</br></br>windows:
```s
> .\bin\windows\kafka-server-start.bat config/server.properties
```
3. create input&output topic
</br></br>windows:用\bin\windows下的bat文件代替sh文件
```s
> bin/kafka-topics.sh --create \
    --zookeeper localhost:2181 \
    --replication-factor 1 \
    --partitions 1 \
    --topic streams-plaintext-input
Created topic "streams-plaintext-input".
```
```s
> bin/kafka-topics.sh --create \
    --zookeeper localhost:2181 \
    --replication-factor 1 \
    --partitions 1 \
    --topic streams-wordcount-output \
    --config cleanup.policy=compact
Created topic "streams-wordcount-output".
```

4. start demo
在intellij idea 中运行 src/java 中的kafka_stream_demo 的main函数，这样之后，每当streams-plaintext-input中被写入，就会被我们的demo截获处理


5. 在console中向streams-plaintext-input的topic中写入单词

```s
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic streams-plaintext-input
> all streams lead to kafka
```
之后，在console中列出steams-wordcount-output这个topic中的记录
```s
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
    --topic streams-wordcount-output \
    --from-beginning \
    --formatter kafka.tools.DefaultMessageFormatter \
    --property print.key=true \
    --property print.value=true \
    --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
    --property value.deserializer=org.apache.kafka.common.serialization.LongDeserializer
 
all     1
streams 1
lead    1
to      1
kafka   1
```

之后，可以在input topic的console中实时写入，并看到output的console中实时更新。
