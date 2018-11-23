# kafka-learning
Messaging & Streaming

## Setup preparation
* stand alone
* jdk
* kafka 2.1.0 (with zookeeper)

## Install JDK

## Download binary package
Download the 2.1.0 release and un-tar it.
```s
tar -xzf kafka_2.11-2.1.0.tgz
cd kafka_2.11-2.1.0
```
## Test with script
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
3. create & list topic
</br></br>windows:
```s
> .\bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
> .\bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
```
4. create producer & consumer
</br></br>producer & consumer in separate shell and send message
</br></br>windows:
```s
> .\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test
This is a message
This is another message
```
```s
> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```

## Create producer & consumer in Java
easy with Maven
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.1.0</version>
</dependency>
```
### producer
```java
... //properties construction
try {
    producer = new KafkaProducer<String, String>(properties);
    for (int i = 0; i < 100; i++) {
        String msg = "Message " + i;
        //send message
        producer.send(new ProducerRecord<String, String>("test", msg));
        System.out.println("Sent:" + msg);
    }
}
```
### consumer
```java
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<String, String>(properties);
kafkaConsumer.subscribe(Arrays.asList("test"));//这里改topic
while (true) {
    ConsumerRecords<String, String> records = kafkaConsumer.poll(100);//timeout
    for (ConsumerRecord<String, String> record : records) {
        Thread.currentThread().sleep(1000);//simulate processing
        System.out.printf("offset = %d, value = %s", record.offset(), record.value());
        System.out.println();
    }
}
```
consumer的 `sleep()` 模拟处理过程，在consumer处理数据时，producer依然可以不间断的向 kafka server 生产 message ，对于 consumer 该过程就是一种 stream processing，对与不停产生的数据流进行连续的处理。