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
