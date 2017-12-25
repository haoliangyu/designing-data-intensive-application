# Partitioning

Partitioning is a process to split a large dataset into smaller subset. Its main purposes are

* to improve system scalability, particularly when the dataset is too large to store and process in a machine.
* to avoid _Hot Spot_ (work load concentrating at a particular part of the system)

Normally each piece of data is stored in one particular partition. Note that a partition could be replicated and located at different nodes. Partition and replication is two independent processes.

## Ways of Partitioning

### Partitioning Randomly

Randomly assign data to different partitions. However, it provides no hint of the data location thus make the data query difficult. One has to query all nodes parallelly.

### Partitioning by Key Range

If the data key value range is known, the partitioning can be done using a set of value boundaries. For example, each partition store a particular range of data.

This is relatively simple and makes range query efficient. It is used by Google Bigtable, HBase, RethinkDB, and MongoDB.

However, it may still create hot spot if the data value or access pattern is skewed. In this case, other key values can be used to change the access distribution.

### Partitioning by Hash of Key

Hash function can produce evenly distributed values so we can use the hash of data key to determine the partition and ensure the even data distribution. Since the range of hash key is known, each partition can store a particular range.

However, it cannot efficiently do range search. As s compensation, one can only hash only one key of the data and store all data with that key in one partition, which makes the range search for secondary keys efficient.

## Secondary Indexes

While the first index is used to determine the partition location, the secondary indexes of the data could be stored in two ways.

### Local Indexes

Each partition is completely separate: each partition maintains its own secondary indexes, covering only the documents in that partition. It is also called document-partitioned index.

The write of indexes in the method is simple: all updates are within its own partition. However, for a read request, it have to query all partitions to gather data from all possible location.

This is used by MongoDB, Riak, Cassandra, ElasticSearch, SolrCloud, and VoltDB.

### Global Indexes

A global index is to cover data in all partitions, but it is split by term in order to stored in different partitions. This is also called term-partitioned index.

The advantage is that read is easy. One can just read one partition for an indexed term and knows all data keys (even they are in different partitions). But writes are slower in this case because writing one partition requires updating indexes that may be stored at other partitions.

This is used by AWS DynamoDB, Orachle.

## Rebalancing Partitions

As the data and system grow, the partitions also need to be updated. This process is called rebalancing.

### Fixed Number of Partitions

This is a simple solution: create many more partitions than there are nodes, and assign several partitions to each node. As each rebalancing, just allocate a partition from each node to the new node.

The diffcult things here is to chose the right number of partitions at the beginning, which will match the future data growth.

### Dynamic Partitioning

This is the strategy to split or merge partitions based on some standards, for example the volume of data in a partition.

### Partitioning Proportionally to nodes

This strategy instead rebalances partitions with a node created or removed.

## Request Routing

Since the data has been partitioned, a data request needs to be routed to the right partition.

Three strategies could be applied:

* Allow clients to contact any node. If that node happens to own the partition to which the request applies, it can handle the request. Otherwise, it forwards the request to the appropriate node, receives the response, and passes to the client.

* Send all requests from clients from a routing tier first, which determines the node that should handle each request and forwards it accordingly. This routing tier does not itself handle any requests and it only acts as a partition-aware load balancer.

* Require that clients be aware of the partitioning and the assignment of partitions to nodes. In this case, a client can connect directly to the appropriate node, without any intermediary.
