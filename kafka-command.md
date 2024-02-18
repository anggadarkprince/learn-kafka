## Kafka
- Download kafka binary https://kafka.apache.org/downloads
- Extract downloaded file to a folder

## Configuration
- edit `config/kraft/server.properties`
- change location kafka data `log.dirs = data`
- generate server id `./bin/kafka-storage.sh random-uuid`
- format data directory `bin/kafka-storage.sh format --cluster-id [random-uuid]`

## Start & stop kafka
- run `./bin/kafka-server-start.sh config/kraft/server.properties`
- to stop `ctrl + c`

## Topic
- Create topic `./bin/kafka-topics.sh --bootstrap-server [localhost:9092] --create --topic [topic-name]`
- Show topic list `./bin/kafka-topics.sh --bootstrap-server [localhost:9092] --list`
- Delete topic `./bin/kafka-topics.sh --bootstrap-server [localhost:9092] --delete --topic [topic-name]`
- Describe topic `./bin/kafka-topics.sh --bootstrap-server [localhost:9092] --describe --topic [topic-name]`

## Producer
- Sender who sent data into kafka
- `./bin/kafka-console-producer.sh --bootstrap-server [localhost:9092] --topic [topic-name]`
- Input message and press enter to append and send
- Press `ctrl + c` to finish & quit the producer console

## Consumer
- Receiver who receive data from kafka
- `./bin/kafka-console-consumer.sh --bootstrap-server [localhost:9092] --topic [topic-name] --from-beginning`
- if `--from-beginning` option is not included then consumer only read the new incoming message only

## Consumer group
- If consumer group is not mentioned in command, kafka automatically create new consumer group (each Consumer)
- Kafka identify a Consumer as unique entity by `topic`, consumer `group name`, and `partition`, so it wil not send to the same consumer group twice
- If consumer group change or not mentioned, then kafka always send the same data for each Consumer
- Let's say we have a topic `Order` and consumed with `3 Servers` with same functionality, 
- then those servers should have same label (consumer group)
- Kafka will send once to 1 of 3 servers to procceed the message
- `./bin/kafka-console-consumer.sh --bootstrap-server [localhost:9092] --topic [topic-name] --from-beginning --group [group-name]`

## Offset
- Kafka has 2 mode when start consumer, read new incoming message only or from beginning (`--from-beginning` option)
- From beginning mean it will read from the last consumer stop (store the `offset`, continue reading message when it left, not very first data when producer start)
- Consumer store the `offset` with the `group-name`, if consumer start with `--from-beginning` option with different `group-name` then it will read from very first message when producer publish the message or `offset 0`, if it start with `--from-begining` after some period of time the consumer off, then it continue when it last stop.
- Check kafka offset `./bin/kafka-consumer-groups.sh --bootstrap-server [localhost:9092] --all-groups --all-topics --describe`

## Partition
- Split queue from publisher into multiple Partitions
- If we have multiple Partitions then a message will be queued in one of the Partitions, not duplicate to all or some of them
- 1 Partition will be consumed by 1 Consumer
- 1 Consumer can consume multiple Partitions
- Kafka try to rebalance between total Partitions and Consumers
- If there is 3 Partitions and 1 Consumer, all message will be consumed to the only Consumer, if we have 2 Consumers, 1 of them consume from 2 Partitions, if there are 3 Consumers, then each Partition will be handled by 1 Consumer, if we have 4 Consumers the last one consumer will be not received any messages.
- Create topic with partition `./bin/kafka-topics.sh --bootstrap-server [localhost:9092] --create --topic [topic-name] --partitions [total]`
- Create topic with partition `./bin/kafka-topics.sh --bootstrap-server [localhost:9092] --alter --topic [topic-name] --partitions [total]`

## Routing
- By default kafka will send message to partition 0 if we do not include `key` (default: null) as routing parameter to calculate which partition it will be queued
- Kafka route the message by `hash(key) % total-partitions`
- `./bin/kafka-console-producer.sh --bootstrap-server [localhost:9092] --topic [topic-name] --property "parse.key=true" --property "key.separator=:" --property "print.key=true"`
