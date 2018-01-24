# Consistency and Consensus

## Linearizability

Linearizability is a recency guarantee that makes a system acting as if there is one replica, regardless of the actual number of replicas. Under this guarantee, the user don't need to consider issues like in inconsistent data read (because of replication lag).

This is useful particularly for **time-sensitive** situations where most recent data is concerned. (see page 330 for more details)

## Implementation

(page 327)

In a linearizable system, there must be some point in time (between the start and end of the write operation) at which the value of x automatically flips from 0 to 1, all subsequent reads must also return the new value, even if the write operation has not yet completed (returned).

It uses a compare-and-set operator _cas(id, value_old, value_new)_ to perform data update. The operator only updates the data (with id) when the current value is equal to the old value, otherwise the operation is aborted. It ensures that if two writes happen at the same time, the most recent one, which has the correct _value_old_, will success and the others will fail.

Therefore, both write and read operate on the most recent data.

## Cost

The Linearizability has lower performance because of the network delay.

## CAP Theorem

_CAP_ stands for:

* Consistency (linearizable)
* Availability
* Partition (network failure tolerance)

This is a historical rule-of-thumb that says a design of distributed system could only satisfy two of three, but not all. However, this rule doesn't consider many other problems (network delays, dead nodes, etc) in the distributed system. Therefore, it is not a precise rule for designing.

## Ordering Guarantee

Ordering is a way to preserve causality of events. In this ordering, event A that dependents on event B must happens after event B. If a system (like those with snapshot isolation snapshot) obeys the ordering imposed by causality, it has causally consistency.

However, causally consistency doesn't guarantee _total order_, which says every two events are comparable in ordering. That's because two events may not have any relationship and they don't have to be before or after the other one. Also two concurrent events are incomparable.

Note that a linearizable system is a total order system since it acts like one machine. That says linearizability is stronger than causal consistency.

## Sequence Number Ordering

Figuring out the causality dependency of historical events is difficult. A reasonable way is to determine the order using _sequence numbers_ or _timestamps_. Instead of a time-of-day clock, the _timestamps_ comes from a logical clock that generates a sequence of numbers to identify operations. In particular this sequence of numbers could be generated in a total order to make assigned events in order.

### Lamport Timestamps

The _Lamport Timestamps_ is a unique identifier that use both the node ID and a counter number. It uses two rules to guarantee the total order:

* The timestamp with a greater counter number is the greater timestamp
* If the counter numbers are the same, the one with the greater node ID is the greater timestamp

Every node and every client keeps track of the maximum counter number it has so far, and includes the maximum on every request. When a node receives a request or response with a maximum number larger than its own, it immediately increases it own to that maximum. So that a node will know which value is the latest one and moves towards it.

**It is not enough!**

Although the total order would provide the time ordering of each event, the order number is given by each node individually and a node doesn't know whether another node receive a conflicting operation concurrently. It has to know the status of other nodes in order to resolve the conflict and the total order solely cannot help.

### Total Order Broadcast

To resolve this problem, an external service could be used to produce messages in total order and broadcast them to the nodes. Particularly, it should have two safety properties:

* Reliable delivery
  No messages are lost: if a message is delivered to one node, it is delivered to all nodes.

* Totally ordered delivery
  Messages are delivered to every node in the same order.

## Distributed Transactions and Consensus

### Atomic Commit and Two-Phase Commit (2PC)

Two-phase commit (2PC) is a method to achieve atomic commit/operation in a distributed network. It is a practical, but not perfect way.

#### Distributed Atomic Commit

In a single node, a commit fails if any operation within the commit fails. But in a distributed network, operations are conducted in multiple nodes, where some operations will success or fail. With the definition of atomic commit, if an operation fails in a node, the commit should be aborted. A system must know the results from all nodes in order to approve or abort a commit.

#### Solution

2PC uses a new system component called coordinator to determine the consensus of nodes. The method has two separate phases to commit:

1. after all nodes success to write/read, the coordinator will send a message to all nodes to ask if they are ready to commit
2. only after all nodes are ready, the coordinator will send a commit request to all nodes
3. with the request message, nodes are dedicated to commit the change and will keep retrying if fails

#### Problems

* coordinator as a possible single-point-failure

### Fault-Tolerant Consensus

The consensus problem is normally formalized as follows:

> one or more nodes may propose values, and the consensus algorithm decides on one of those values.

A consensus algorithm must satisfy the following properties:

* Uniform agreement
  No two nodes decide differently.

* Integrity
  If a node decides value v, then v was proposed by some node.

* Termination
  Every node that does not crash eventually decides some value.

#### Algorithms

The first and second property defines the core idea of the algorithm, while the last one is to ensure that the decided value make senses.

Best known algorithms include: Viewstamped Replication, Paxo, Raft, and Zab. Instead of proving one consensus, they decide on a sequence of values, which makes them total order broadcast algorithms:

* Due to the agreement of consensus, all nodes decide to deliver the same message in the same order.

* Due to the integrity property, messages are not duplicated.

* Due to the validity property, messages are not corrupted and not fabricated out of thin air.

* Due to the termination property, messages are not lost.

#### Limitations

* Acquiring consensus of nodes is a form of synchronous operations and it limits the performance of system.

* Consensus systems always require a strict majority to operate.

* Most consensus algorithms assume a fixed set of nodes that participate in voting.

* Consensus systems generally rely on timeouts to detect failed nodes.

* Sometimes consensus algorithms are particularly sensitive to network problems.
