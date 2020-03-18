# Elasticsearch FAQ

## What is an index, a shard and a replica

- An index is like a table in a relational database. It has a mapping which contains a type, which contains the fields in the index.
- An index is a logical namespace which maps to one or more primary shards and can have zero or more replica shards.
- A shard is a piece of data in an index. Each shard contains a X number of entire documents  (documents can't be sliced) and each node of your cluster holds this piece accordingly to the "shard_number" configured to the index where the data is stored.
- Elasticsearch distributes shards amongst all nodes in the cluster, and can move shards automatically from one node to another in the case of node failure, or the addition of new nodes.
- Each primary shard can have zero or more replicas. A replica is a copy of the primary shard, and has two purposes:
  - increase failover: a replica shard can be promoted to a primary shard if the primary fails
  - `shard = hash(document_id) % (num_of_primary_shards)`
increase performance: get and search requests can be handled by primary or replica shards.
  - By default, each primary shard has one replica, but the number of replicas can be changed dynamically on an existing index. A replica shard will never be started on the same node as its primary shard.

## How many replicas has a shard and where can we configure it?

- you set up number of shards  and number of replicas when you create an index
- but number of created replicas depends of number of cluster nodes
- example of index creating

```json
curl -X PUT "localhost:9200/twitter?pretty" -H 'Content-Type: application/json' -d'
{
    "settings" : {
        "index" : {
            "number_of_shards" : 3,
            "number_of_replicas" : 2
        }
    }
}
'
```

## How to see the list of indices? How to see list of shards? (please use API endpoints)

- `curl -X GET "localhost:9200/_cat/indices?v"`
- `curl -X GET "localhost:9200/_cat/shards?v"`

## What is the cluster status? What does the color mean?

- `curl -XGET 'localhost:9200/_cluster/state?pretty'`
- `curl -X GET "localhost:9200/_cluster/health?pretty"`
- colors
  - green
    - All shards are assigned.
  - yellow
    - All primary shards are assigned, but one or more replica shards are unassigned. If a node in the cluster fails, some data could be unavailable until that node is repaired.
  - red
    - One or more primary shards are unassigned, so some data is unavailable. This can occur briefly during cluster startup as primary shards are assigned.

## How to see which shards are unassigned and the reason why are they unassigned?

- `curl -X GET "localhost:9200/_cat/shards?h=i,sh,n,st,ur&v"`

## How to reassign the shards across the cluster?

- By default, Elasticsearch will re-assign shards to nodes dynamically.
- reroute API

```json
curl -X POST "localhost:9200/_cluster/reroute?pretty" -H 'Content-Type: application/json' -d'
{
    "commands" : [
        {
            "move" : {
                "index" : "test", "shard" : 0,
                "from_node" : "node1", "to_node" : "node2"
            }
        },
        {
          "allocate_replica" : {
                "index" : "test", "shard" : 1,
                "node" : "node3"
          }
        }
    ]
}
'
```

## What is the index mapping? Which mappings are applied to an index during its creation?

Mapping is the process of defining how a document, and the fields it contains, are stored and indexed. For instance, use mappings to define:

- which string fields should be treated as full text fields.
- which fields contain numbers, dates, or geolocations.
- the format of date values.
- custom rules to control the mapping for dynamically added fields.

- Dynamic mapping
  - Fields and mapping types do not need to be defined before being used. Thanks to dynamic mapping, new field names will be added automatically, just by indexing a document. New fields can be added both to the top-level mapping type, and to inner object and nested fields.

  - The dynamic mapping rules can be configured to customise the mapping that is used for new fields.

- Explicit mapping

```json
curl -X PUT "localhost:9200/my-index?pretty" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}
'
```

## What is the index template? How do we manage them?

- An index template defines settings, mappings, and aliases that you can automatically apply when creating a new index. Elasticsearch applies a template to a new index based on an index pattern that matches the index name.

```bash
curl -X PUT "localhost:9200/_template/template_1?pretty" -H 'Content-Type: application/json' -d'
{
  "index_patterns": ["te*", "bar*"],
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "host_name": {
        "type": "keyword"
      },
      "created_at": {
        "type": "date",
        "format": "EEE MMM dd HH:mm:ss Z yyyy"
      }
    }
  }
}
'
```

```bash
curl -X GET "localhost:9200/_template/template_1?pretty"
```

## How to see the list of the nodes in a cluster?

- `curl -X GET "localhost:9200/_cat/nodes?v&pretty"`

## What are the node roles in Elasticsearch? (Data, master, ingest, client) How roles can be assigned to a node?

- Master
  - It controls the Elasticsearch cluster and is responsible for all clusterwide operations like creating/deleting an index, keeping track of which nodes are part of the cluster and assigning shards to nodes. The master node processes one cluster state at a time and broadcasts the state to all the other nodes which respond with confirmation to the master node.
- Data Node
  - It holds the data and the inverted index. By default, every node is configured to be a data node and the property node.data is set to true in elasticsearch.yml. If you would like to have a dedicated master node, then change the **node.data** property to false.
- Client Node
  - If you set both **node.master** and **node.data** to false, then the node gets configured as a client node and acts as a load balancer routing incoming requests to different nodes in the cluster.
- Ingest node
  - Use an ingest node to pre-process documents before the actual document indexing happens. The ingest node intercepts bulk and index requests, it applies transformations, and it then passes the documents back to the index or bulk APIs. 
  - **node.ingest**

## How many master nodes should be in a cluster?

- only one master node
- A node can be configured to be eligible to become a master node by setting the **node.master** property to be true (default) in elasticsearch.yml.

## How to add a new node to cluster? Which ports should be opened?
- Set up a new Elasticsearch instance.
- Specify the name of the cluster with the **cluster.name** setting in elasticsearch.yml. 
- Start Elasticsearch. The node automatically discovers and joins the specified cluster.
- 9200/tcp, 9300/tcp

## What happens when one of data nodes goes down? What happens when it gets back and joins the cluster?

- Stop
  - If we stop a node or a node in the cluster is unresponsive in specific amout of time, the master node will remove it from the cluster and reallocate the data if the removed node is a data node.

  - If the master is not alive, other master-eligible nodes will be elected new master to replace the down one within seconds.

- Start
  - When we start a node, the node is starting to ping all the nodes in the cluster for finding the master node. Once the master is found, it will ask the master to join by sending a join request; the master accepts it as a new node of the cluster and then notify all the nodes in the cluster about presense of the new node, and finally the new node connects to all other nodes.
  - If the joined node is a data node, the master will reallocate the data evenly across the nodes.

## Where does the node store its data?

- **path.home:** Home directory of the user running the Elasticsearch process. Defaults to the Java system property user.dir, which is the default home directory for the process owner.
path.conf: A directory containing the configuration files. This is usually set by setting the Java system property es.config, as it naturally has to be resolved before the configuration file is found.
- **path.plugins:** A directory whose sub-folders are Elasticsearch plugins. Sym-links are supported here, which can be used to selectively enable/disable a set of plugins for a certain Elasticsearch instance when multiple Elasticsearch instances are run from the same executable.
path.work: A directory that was used to store working/temporary files for Elasticsearch. It’s no longer used.
- **path.logs:** Where the generated logs are stored. It might make sense to have this on a separate volume from the data directory in case one of the volumes runs out of disk space.
- **path.data:** Path to a folder containing the data stored by Elasticsearch.
- `curl -X GET "localhost:9200/_nodes/stats?pretty" | grep data`

## How to store documents in an index?

```bash
curl -X POST "localhost:9200/twitter/_doc/?pretty" -H 'Content-Type: application/json' -d'
{
    "user" : "test",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

## How to get documents from an index?

```bash
curl -XGET "localhost:9200/twitter/_doc/<_id>"
```

## How to search for the documents and get the aggregated metrics.

```bash
curl -X GET "localhost:9200/_search?pretty" -H 'Content-Type: application/json' -d'
{
    "query": {
        "term": {
            "user": {
                "value": "test"
            }
        }
    }
}
'
```

- [metrics documentattion](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics.html)
- metrics example

```bash
curl -X POST "localhost:9200/exams/_search?size=0&pretty" -H 'Content-Type: application/json' -d'
{
    "aggs" : {
        "avg_grade" : { "avg" : { "field" : "grade" } }
    }
}
'
```

## How indices are rotated? How old indices are deleted? (multiple approaches)

- [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html)
- according default policy 

```bash
curl -X PUT "localhost:9200/_ilm/policy/datastream_policy?pretty" -H 'Content-Type: application/json' -d'
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
'
```

- The min_age defaults to 0ms, so new indices enter the hot phase immediately.
- Trigger the rollover action when either of the conditions are met.
- Move the index into the delete phase 90 days after rollover.
- Trigger the delete action when the index enters the delete phase.

- according conditions specified for alias
  - The rollover index API rolls an alias to a new index when the existing index meets a condition you provide. You can use this API to retire an index that becomes too large or too old.
- [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-rollover-index.html)

- example:

```bash
curl -X POST "localhost:9200/alias1/_rollover/twitter?pretty" -H 'Content-Type: application/json' -d'
{
  "conditions": {
    "max_age":   "7d",
    "max_docs":  1000,
    "max_size": "5gb"
  }
}
'
```

- you can delete old indicies using curator
  - `curator --host <ip address> delete indices --time-unit days --older-than 60 --timestring '%Y%m%d'`