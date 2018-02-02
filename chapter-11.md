# Stream Processing

In the batch processing, the input data is **bounded**. The size and length of data is predictable and known. If the input data is **unbounded** and it arrives in a continuous way, we use a stream processing system to process it.

A continuous flow of inputs is a stream, of which the source is called _producer_ and the recipient is called _consumer_. In principle, a record from the producer can be stored into a durable storage and the consumer can periodically poll records from the storage system. But this polling could be expensive as the size of storage grows and becomes slow in the time.

### Messaging Systems

Within a messaging system, the consumer and producer can communicate with the following ways:

* Using TCP/UDP to build a direct communication channel

* Use a **webhook** function at the producer to call an endpoint of the consumer

* Use a message broker as a media to transfer and queue messages (compared to the database, a message broker usually doesn't hold the assumption of durable storage and large message)

If multiple consumers are involved, there are usually two ways of messaging:

* Load Balancing

  Each message is delivered once and to one of the consumers.

* Fan-out

  Each message is delivered to all consumers.

To confirm the successful delivery of a message, the consumer should send an **acknowledge** for the message to the message broker. This is in case the failure of message processing/delivery and the message broker can retry the delivery if the message is not acknowledged.

### Partitioned Logs

Log-based message broker is a hybrid system of the durable storage of database and the low-latency notification of normal message broker.

This kind of system uses an append-only log to store all messages in a sequence. Like a database, a log can be partitioned and stored in different nodes.

Within each partition, the broker assigns a monotonically increasing sequence number to every message. Note that there is no ordering guarantee across different partitions.

When a consumer subscribes to a queue, it uses a pointer with an offset to read the log sequentially. It starts from an offset and read until it meets an unread message and then update the offset to the message number.

If the message is sent in a fan-out mode, each consumer can independently read the log to read all records. But if in the load balancing mode, a message needed to be delivered once and one solution is to assign a consumer to one single partition, which means one consumer only subscribes to one log.

### Change Data Detection

_Change Data Detection_ is a process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems.

This is can be resolved by the _dual write_ process. The application explicitly writes to each part of the system when data changes. However, this method is not robust to failure and suffers from performance issue.

Since a database internally stores data in a replication log, this can be treated as the partitioned log for stream processing, which allows external systems to subscribe.

Since the stream subscription is asynchronous, it is able to decouple the producer and consumer.

### Event Sourcing

Beside emitting data results in the stream, a producer can emit events that represent the action taken to produce the data results.

This keeps the history of actions, instead of data.

### Fault Tolerance

Similar to the batch process, the goal of fault tolerance is to enable retry after the failure and return the current result. It ensures the process output only one result even after multiple retries (exactly-once schema).

However, since a stream is an unbounded data, tolerating error in the stream processing cannot be as simple as waiting until the stream finishes before restarts. A stream is unstoppable and infinite.

#### Microbatching and Checkpointing

One approach is to break the stream in to small blocks and treat each block like a minimal batch processing.

There is a trade-off in choosing the right batch size. If the size is too small, it increases the overhead to create and schedule blocks. But if the chosen block is too large, the stream could have a significant lag because of the processing time.

A variant approach is to periodically generate rolling-back checkpoints of state and write them to durable storage. If a stream operator crashes, it can restart from its most recent checkpoint and discard any output generated between the last checkpoint and the crash. _Does this only work for aggregation operator?_ (in Apache Flink)

#### Atomic Commit

Within a restricted environment, atomic commit for stream processing could be implemented efficiently. Similar features is implemented in Google Cloud Dataflow and VoltDB.

#### Idempotence

An idempotent operation is one that you can performance multiple times, and it has the same effect as if you performed it once. This function should be deterministic: given a specific input, the output should be always the same.

Even if an operation is not naturally idempotent, it can often be made idempotent with extra metadata. For example, by given a monotonically increasing offset to each action, the application can tell whether has been updated by the same action with the last-updated offset. It can help avoid performing the action twice.
