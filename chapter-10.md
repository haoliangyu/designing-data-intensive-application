# Batch Processing

## Map Reduce and Distributed Filesystems

MapReduce is a system that run sequences of map/reduce jobs on a cluster. A single MapReduce job is comparable to a single Unix process, which takes one or more inputs and produce one or more outputs.

The MapReduce job doesn't modify the input in order to generate the output.

To have a standard inputs and outputs for MapReduce jobs, it usually read and write files on a distributed filesystem. In Hadoop's ecosystem, this is called HDFS (Hadoop Distributed File System), an open-source implementation of Google File System.

### MapReduce Job Execution
