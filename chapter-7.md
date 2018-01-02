# Transactions

_Transactions_ are an abstraction layer that allows an application to protect that certain concurrency problems and certain kinds of hardware and software faults don't appear.

The guarantee provided by transactions could be summarized as ACID:

* Atomicity

  An _atomic_ transaction (group of operations) can only be completed or aborted all in once. Once aborted all writes must be discarded.

* Consistency

  It requires data must obey certain conditions, therefore _consistent_, though it is usually the application, not the database, to provide such conditions.

* Isolation

  _Isolated_ transactions would not affect each other. That says, theoretically, they can be executed in sequence (serializable).

* Durability

  A successful transaction should write _durable_ data.

Note that the implementation of ACID is formalized and it is up to the database vendor.

## Weak Isolation

### Dirty Reads

One client reads another client's writes before they have been committed. This problem can be resolved by not allowing reads from the uncommitted (dirty) writes, which is also referred as _read committed isolation_. Stronger levels prevent dirty reads as well.

### Dirty Writes

One client overwrites data that another client has written, but not yet committed. **Almost all transaction implements prevent dirty writes.**

### Read Skew

A client sees different parts of the database at different points in time. This issue is most commonly prevented with snapshot isolation, which allows a transaction to read from a consistent snapshot at one point in time. It is usually implemented with _multiple-version concurrency control_ (MVCC).

This could happen when the client has two reads in a commit and an 3rd-party write is committed between those two reads. The results of two writes could be inconsistent because they read data at different state. MVCC guarantees a transaction to always read a consistent snapshot of the data.

With snapshot isolation, writes don't block reads and reads don't block writes.

This is also called _nonrepeatable reads_.

### Lost Updates

Two clients concurrently perform a read-modify-write cycle. One overwrites the other's write without incorporating its changes, so data is lost. Some implementations of snapshot isolation prevent this anomaly automatically, which others require a manual lock (SELECT...FOR UPDAE).

### Write Skew

A transaction reads something, makes a decision based on the value it saw, and writes the decision to the database. However, by the time the write is made, the premise of decision is no longer true because the data may have been updated by other transaction. Only serializable isolation prevents this anomaly.

### Phantom Reads

A transaction reads objects that match some research conditions. Another client makes a write that affect that search. Snapshot isolation prevents straightforward phantom reads, but phantoms in the context of write skew require special treatment, such as index-range locks.

## Serializable Isolation

Though weak isolation can prevents some of anomalies, only serializable isolation protects against all issues.

### Serial Execution

The simplest solution to concurrency issues is to remove concurrency itself by executing transactions in strict sequence. This comes with significant performance penalty and should be used when:

* each transaction is very fast to execute
* the transaction throughput is low enough to process on a single CPU core

### Two-phase Locking

It add locks to both read and write to allow them to block all other operations. In this way, there will be only one transaction executing at one point in the time.

### Seralizable Snapshot Isolation

It uses an optimistic approach to detect transactions that causes consistency issue and abort/lock them. It will allows transactions (of no problem) to proceed without blocking. When a transaction wants to commit, it is checked, and it is aborted if the execution was not seralizable.
