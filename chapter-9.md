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
