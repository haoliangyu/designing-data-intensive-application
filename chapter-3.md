# Storage and Retrieval

## Sorted String Table (SSTable)

### What is it?

A row-based (like csv), append-only table (no in-place value update and just append new value), sorted (row by value) table in the disk

### Why is it good?

Since it is sorted, it is very simple to compress the raw table (like using the run-length encoding) and merge two tables into a new table with latest values (using merge-sort).

## Log-structured Merge-Tree (LSM-Tree)

### What is it?

A hybrid solution (memory + disk) to handle data storable problem, particularly for systems with high volume of write operations.

The storage engine works as follows:

1. When a write comes in, add it to an in-memory balanced tree data structure (like, a red-black tree), A.K.A, memtable.

2. When the memtable gets bigger than some thresholds, write it out into a SSTable file. This can be done efficiently because the tree already maintains the key-value pairs sorted by key. The new SSTable becomes the most recent segment of the database. While the SSTable is being written out to disk, writes can continue to a new memtable instance.

3. In order to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment, then in the next-older segment, etc.

4. From time to time, run a merging and compacting (compressing) process in the background to combine segment files and to discard overwritten or deleted values.

### Why is it good?

It is good at handling high write throughput, because the hard disk is very fast at appending data (sequential write), comparing to random access and write.

Better compression rate.

### Why is it not so good?

Because it has to look for a key in segments, looking for historical data or handling large amount of read access is slower than other database structure.

It needs time and resource to merge and compress segments, which could be a challenge in high write-in.

### Who are using?

LevelDB, RocksDB, HBase, Cassandra

## B-Tree

### What is it?

B-Tree breaks the database into a fixed-size blocks (or pages) and write/read one page at a time (lock).

Each page can be identified using an address or location, which allows one page to refer to another, like a pointer, but on disk instead of in memory.

One page is designed as the root of the B-Tree. Whenever you want to look up a key in the index, you start here. The page contains several keys (nodes), and the keys between references indicate where the boundaries between those ranges lie.

At the bottom of the tree, leaf pages either contain the value of each key inline, or contain references to the pages where values are actually stored.

Once a page is bigger than a certain size, the page is split.

### Why is it good?

Very reliable and have a good balance at write & read

Why is it not so good?

May not be as capable in write as LSM-Tree

### Who are using?

All relation database implements B-Tree.

## Transaction Processing vs. Analytics

Online transaction processing (OLTP) is a database usage pattern to describe applications that have request but small read/write access and require low response time. These applications are usually customer-facing application.

Online analytic processing (OLAP) is a database usage pattern to describe applications that generally have infrequent read operation with large amount of data. These applications are usually analyst-facing and for data analysis (aggregation).

To optimize OLAP, a data warehouse is built for big-data access, for example, using write-optimized structure, and out of the customer (OLTP) databases after Extract-Transform-Load (ETL).

## Column-Oriented Storage

Instead of storing complete rows in each database segment files, column-oriented storage stores complete columns. It enables to quickly access all data belonging to a specific column (because it skips all data at other columns).

Note that at each column segment files, the row order needs to be maintained so that we know at nth position of a file is the nth row.

Sorted COS is particularly good for data compression because same values are sequential then. While the first column index sorts all values and changes the value position, the second index can just be sorted reference table to the actual value position.
