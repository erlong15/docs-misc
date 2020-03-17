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
# How retention for topic works? How to see retention policy for topic?
# What is the consumer group? Consumer group offset? Where is it stored?

# How to see offset for consumer group? How to reset it?
# How to see which consumer reads which partition?
# How many consumers can read from one topic in the same consumer group? What happens if you run more consumers than there's partitions for the topic?
# How many consumers can read from single partition in the same consumer group?
# What happens if one of consumers (one of many inside the same consumer group) goes down?
# What is the key when you publish a message to kafka?
# Which partition message goes to when you publish to a topic?