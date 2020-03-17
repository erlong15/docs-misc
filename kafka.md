# What is topic? What is partition? What is replication factor?
- A topic is a category or feed name to which records are published. Topics in Kafka are always multi-subscriber; that is, a topic can have zero, one, or many consumers that subscribe to the data written to it.
- Partition is a shard of topic. Each partition is an ordered, immutable sequence of records that is continually appended to—a structured commit log. The records in the partitions are each assigned a sequential id number called the offset that uniquely identifies each record within the partition
- Kafka replicates the log for each topic's partitions across a configurable number of servers (you can set this replication factor on a topic-by-topic basis). This allows automatic failover to these replicas when a server in the cluster fails so messages remain available in the presence of failures.
# What is the role of zookeeper for kafka?
- Distributed key-value storage. Zookeeper is used for metadata storing and cluster processing. 
# Where brokers information stored? How to see the list of brokers? How to see each broker details?
- brokers info is stored in zookeper
- get list of broker's IDs: `zookeeper-shell zookeeper:2181 ls /brokers/ids`
- get detailed info: `zookeeper-shell zookeeper:2181 get /brokers/ids/1`
# How to see the list of topics? Topic partitions? Where topic information is stored?
- topics info is stored in zookeeper either
- topics themself are stored in directory according `/etc/kafka/kafka.properties` 
    - `log.dirs=/var/lib/kafka/data`
- `kafka-topics --zookeeper zookeeper:2181 --list`
- `kafka-topics --bootstrap-server broker:9092 --list`
- `kafka-topics --bootstrap-server broker:9092 --describe --topic <topic>`
# How retention for topic works? How to see retention policy for topic?
- Time based retention
    - Once the configured retention time has been reached for Segment, it is marked for deletion or compaction depending on configured cleanup policy. Default retention period for Segments is 7 days.
    - `log.retention.ms`
    - `log.retention.minutes`
    - `log.retention.hours`
- Size Based Retention:
    - In this policy, we configure the maximum size of a Log data structure for a Topic partition. Once Log size reaches this size, it starts removing Segments from its end.
    - `log.retention.bytes`
# What is the consumer group? Consumer group offset? Where is it stored?
- consumer group - is group of related  consumers that perform the same task. 
    - Each consumer in group read own partition. 
    - Two consumers in one group can't read the same partition. 
    - Different consumer groups can read from different offsets in a partition
- The offsets for  groups are stored in zookeeper.
# How to see offset for consumer group? How to reset it?
- list of consumer groups
    - `kafka-consumer-groups --bootstrap-server localhost:9092 --list`
- list of offsets
    - `kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group <group> --offsets`
- reset offsets
    - `kafka-consumer-groups --bootstrap-server localhost:9092 --reset-offsets --group <group>` 
# How to see which consumer reads which partition?
    - `kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group <group>`
# How many consumers can read from one topic in the same consumer group? What happens if you run more consumers than there's partitions for the topic?
- ideally one consumer for one partition in a topic. 
- One consumer can read several partitions in one topic.
- it there are more consumers the topics then some consumers will get no data.
# How many consumers can read from single partition in the same consumer group?
- only one
# What happens if one of consumers (one of many inside the same consumer group) goes down?
- if one of consumers dies Kafka will do reblance.
# What is the key when you publish a message to kafka?
- Keys are used when records are to be written to partitions in a more controlled manner. The simplest such scheme is to generate a consistent hash of the key, and then select the partition number for that record by taking the result of the hash modulo, the total number of partitions in the topic. This assures that records with the same key are always written to the same partition.
- The key is particularly important if modeling a Kafka topic as a table in KSQL (or KTable in Kafka Streams) for query or join purposes.
# Which partition message goes to when you publish to a topic?
- according to hash of key
