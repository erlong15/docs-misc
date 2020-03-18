# Elasticsearch FAQ

## What is an index, a shard and a replica

- An index is like a table in a relational database. It has a mapping which contains a type, which contains the fields in the index.
- An index is a logical namespace which maps to one or more primary shards and can have zero or more replica shards.
- A shard is a piece of data in an index. Each shard contains a X number of entire documents  (documents can't be sliced) and each node of your cluster holds this piece accordingly to the "shard_number" configured to the index where the data is stored.
- Elasticsearch distributes shards amongst all nodes in the cluster, and can move shards automatically from one node to another in the case of node failure, or the addition of new nodes.
- Each primary shard can have zero or more replicas. A replica is a copy of the primary shard, and has two purposes:
  - increase failover: a replica shard can be promoted to a primary shard if the primary fails
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

## What is the index template? How do we manage them?

## How to see the list of the nodes in a cluster?

- `curl -X GET "localhost:9200/_cat/nodes?v&pretty"`

## What are the node roles in Elasticsearch? (Data, master, ingest, client) How roles can be assigned to a node?

## How many master nodes should be in a cluster?

## How to add a new node to cluster? Which ports should be opened?

## What happens when one of data nodes goes down? What happens when it gets back and joins the cluster?

## Where does the node store its data?

## How to store documents in an index?

## How to get documents from an index?

## How to search for the documents and get the aggregated metrics.

## How indices are rotated? How old indices are deleted? (multiple approaches)
