# The Trouble with Distributed System

A distributed system is built with a cluster of individual nodes that don't have shared memory/storage and are only connected with network. Each component of the system may go wrong and problems could happens like failed node, disconnected network, delayed communication. Building a reliable system from unreliable components requires an understanding of the type and nature of problems.

## Unreliable Network

Network communication could be lost or delayed during a round-trip conversation.

One cause of this is the asynchronous nature of our Internet. In an asynchronous network, the data packet is sent as soon as possible with all possible bandwidth. While this is optimized for bursty traffic, it also make the communication not very predictable because of the varying network condition.

Synchronous networks, which uses fixed bandwidth for each communication and make it predictable, exist, but it is either expensive or restricted.

## Unreliable Clock

Many factors make time in a distributed system unreliable:

* time itself is complicated to measure precisely. 60s for a minute and 24 hour for a day are not the truth.
* the instrument to measure time is not precise or inaccurate enough
* each node has its own clock and they may have different time

Beside the normal time-of-day clock, the streaming of time can be measured by the monotonic clock. A monotonic time is to provide incremental values as time moving forwards. Note that the absolute value is meaningless because the source of value is arbitaray, and values from two systems are not comparable because they can be measured in different way.

Node clocks can be synchronized using _Network Time Protocol_ (NTP), which allows the computer clock to be adjusted according to the time reported by a group of servers. However, even this cannot guarantee the accuracy of time because the synchronization is under the influence of network issue. Therefore, some services (Google) will provide a confidence interval instead of an absolute time point.

## Knowledge, Truth, and Lies

In a distributed system, a node is only acknowledge on itself and it knows other nodes via network communication. If it is unable to acquire the state of another node, it cannot tell whether it is because of the network failure or node failure. It has to ask the other node about the state of unresponding node. Therefore, in the system, whether the node is alive is decided by how the other nodes view it. If that node is just suffering network issue, the system could treat it as dead because of no response.

In most case, we can expect nodes are sending truth to each other. But a node may send unreal message and create a truth issue in the system. It is known as _Byzantine Generals Problem_.

## System Models

System models are abstractions that describe what things an algorithms may assume when addressing distributing system issues.

### Timing Assumptions

* Synchronous Model

  The synchronous model assumes bounded network delay, bounded process pauses, and bounded clock error. This is not a realistic model since unbounded delays and pauses are usually unable to guarantee.

* Partially Synchronous Model

  Partial synchronous means that a system behaves like a synchronous system most of the time, but it sometimes exceeds the bounds for network delay, process pauses, and clock drift. This is a realistic model.

* Asynchronous Model

  In this model, an algorithm is not allowed to make any timing assumptions. It does not even have a clock. Some algorithms can be designed for the asynchronous model, but it is very restricted.

### Node Failure Assumptions

* Crash-stop Faults

  In the crash-stop model, an algorithm may assume that a node can fail in only one way, namely crashing. This means that the node may suddenly stop responding at any moment, and therefore that node is gone forever, never come back.

* Crash-recovery Faults

  We assume that nodes may crash at any moment, and perhaps start responding again after unknown. Nodes are assumed to have stable storage that is preserved across crashes, while in-memory state is assumed to be lost.

* Byzantine Faults

  Nodes may do absolutely anything, including trying to trick and deceive other nodes.

In modeling the real system, the Partially synchronous model with crash-recovery is generally the most useful model.
