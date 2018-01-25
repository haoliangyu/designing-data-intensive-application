# Batch Processing

## Map Reduce and Distributed Filesystems

MapReduce is a system that run sequences of map/reduce jobs on a cluster. A single MapReduce job is comparable to a single Unix process, which takes one or more inputs and produce one or more outputs.

The MapReduce job doesn't modify the input in order to generate the output.

To have a standard inputs and outputs for MapReduce jobs, it usually read and write files on a distributed filesystem. In Hadoop's ecosystem, this is called HDFS (Hadoop Distributed File System), an open-source implementation of Google File System.

### MapReduce Job Execution

The execution of MapReduce is similar to this:

1. Read a set of input files from the distributed file system and break it into records.

2. Call the mapper function to extract a key and value from each input.

3. Sort all of the key-value pairs by key.

4. Call the reducer function to iterate over sorted key-value pairs. If there are multiple occurrences of the same key, the sorting has made them adjacent in the list, so it is easy to combine those values without having to keep a lot of state in the memory.

See page 403 for more details in variation of mappers and reducers.

#### Mapper

A mapper is a function called for every input record, and its job is to extract the key and value from the input record. For each input, it may generate any number of key-value pairs. It does not keep any state from one input record to the next, so each record is handled independently.

#### Reducer

A reducer function takes the key-value pairs produced by the mappers, collects all the value belonging to the same key, and iterates over that collection of values, then produces outputs.

#### Distributed Execution

Since the mapper and reducer is stateless functions, they can be run independently in the distribute network nodes.

#### MapReduce Workflows

Since both the input and output of a MapReduce job is stored in the distribute file system, the output of one MapReduce job can be the input of another MapReduce job. This forms a chain of MapReduce jobs, called MapReduce workflow, and can be built for very complex systems.
