# kafka-learning
Messaging & Streaming

## Setup preparation
* stand alone
* jdk
* kafka 2.1.0 (with zookeeper)

## Install JDK

## Download binary package
Download the 2.1.0 release and un-tar it.
```
tar -xzf kafka_2.11-2.1.0.tgz
cd kafka_2.11-2.1.0
```
## Test with script
1. start zookeeper server
</br></br>windows:
```
> .\bin\windows\zookeeper-server-start.bat config/zookeeper.properties
```
2. start kafka server
</br></br>windows:
```
> .\bin\windows\kafka-server-start.bat config/server.properties
```
3. create & list topic
</br></br>windows:
```
> .\bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
> .\bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
```
4. create producer & consumer
</br></br>producer & consumer in separate shell and send message
</br></br>windows:
```
> .\bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test
This is a message
This is another message
```
```
> .\bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```


## Create producer & consumer in Java
