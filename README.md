# assignment-4.5
QN 1. Explain what is checksum and the importance of checksum and how hadoop performs checksum
CHECK SUM 
It is one of the common method of error checking while transmission.This methods divide the data in m number of k Bytes. Now at the sender end ,we will add each segment and find the sum and take the complement of the sum and at the  receiver end we will add these segments and to these we will add the checksum and we will get the complement of it and if it is equal to zero.(0000)
THE Logic is that we are sending the set of data and its sum in compliment form and in sender end we are actually adding the checksum with the input and taking the inverse and ofcource the result will be zero.
In a cluster of thousands of nodes, failures of a node (most commonly storage faults) are daily occurrences. A replica stored on a DataNode may become corrupted because of faults in memory, disk, or network. HDFS generates and stores checksums for each data block of an HDFS file. Checksums are verified by the HDFS client while reading to help detect any corruption caused either by client, DataNodes, or network. When a client creates an HDFS file, it computes the checksum sequence for each block and sends it to a DataNode along with the data. A DataNode stores checksums in a metadata file separate from the block's data file. When HDFS reads a file, each block's data and checksums are shipped to the client. The client computes the checksum for the received data and verifies that the newly computed checksums matches the checksums it received. If not, the client notifies the NameNode of the corrupt replica and then fetches a different replica of the block from another DataNode.
When a client opens a file to read, it fetches the list of blocks and the locations of each block replica from the NameNode. The locations of each block are ordered by their distance from the reader. When reading the content of a block, the client tries the closest replica first. If the read attempt fails, the client tries the next replica in sequence. A read may fail if the target DataNode is unavailable, the node no longer hosts a replica of the block, or the replica is found to be corrupt when checksums are tested.
• A separate checksum is created for every dfs.bytes-per-checksum bytes of data.
• The default is 512 bytes, and because a CRC-32C checksum is 4 bytes long. The
storage overhead is less than 1%.
• Datanodes are responsible for verifying the data they receive before storing the data and
its checksum.
• When the clients read data from datanodes, they verify checksums as well, comparing
them with the ones stored at the datanodes.
• -get command does the checksum verification during the data read.
• -copyFromLocal doesn’t perform checksum during data read.
• -ignoreCrc option with the -get is equivalent to -copyToLocal command.
• Disabling checksum verification is useful if we have a corrupt file that we want to inspect
so that we can decide what to do with it. 

QN 2. Explain the anatomy of file write to HDFS
ANATOMY of writing a file
process
Step 1:  client machine will split the file  into blocks(of size 128MB in case of Hadoop 2.o and of size 64 MB in case of Hadoop 1.o) say there is a 300 MB file the client machine will first reduce into 3 blocks of size 128MB,128MB,44MB.So the processing time will  be one third of the processing time 
STEP2: 2. Client  contacts namenode to create a new file in the filesystem’s namespace, with no blocks associated with it. The namenode performs various checks to make sure the file doesn’t already exist and that the client has the right permissions to create the file. If these checks pass, the namenode makes a record of the new file (in edits) and the Name Node orders the file where to write
STEP 3: Client writes to data node1 as assigned by the NameNode the block 1  and Now the slave node(Data Node1 )Copies those to other data nodes depending upon by replication factor as ordered by name node
STEP 4:Name Node thus makes use of Rack awareness and orders the Data Node to store the REPLICATE blocks.
Anatomy of writing a file Steps in Detail

1.Client creates, calls create() on DistributedFileSystem. 
2. DistributedFileSystem contacts namenode to create a new file in the filesystem’s namespace, with no blocks associated with it. The namenode performs various checks to make sure the file doesn’t already exist and that the client has the right permissions to create the file. If these checks pass, the namenode makes a record of the new file (in edits) 
3. DistributedFileSystem returns an FSDataOutputStream for the client to start writing data. FSDataOutputStream uses DFSOutputStream, which handles communication with the datanodes and namenode.
4.Client signals write() method on FSDataOutputStream.
5. DFSOutputStream splits data into packets and writes it to an internal queue called the data queue. • The data queue is consumed by the DataStreamer, which asks the namenode to give a location for the datanodes where blocks will be stored. • The list of datanodes form a pipeline with a number of datanodes equals replication factor. • The DataStreamer streams the packets to the first datanode in the pipeline. • First datanode stores each packet and forwards it to the second datanode in the pipeline. • Similarly, the second datanode stores the packet and forwards it to the third datanode in the pipeline. • The DFSOutputStream also maintains an internal queue called the ack queue. • A packet is removed from the ack queue only when it has been acknowledged by all the datanodes in the pipeline. • So there are two queues: data queue (containing packets that are to be written) and ack queue (containing packets that are yet to be acknowledged)
6. When the client has finished writing data, it calls close() on the stream. This flushes all the remaining packets to the datanode pipeline and waits for acknowledgments. 7. DistributedFileSystem contacts the namenode to signal that the file write activity is complete. • The namenode already knows which blocks the file is made up of (because DataStreamer had asked for block allocations), so it only has to wait for blocks to be minimally replicated before returning successfully. • Closing a file in HDFS performs an implicit hflush(). • After a successful return from hflush(), HDFS guarantees that the data written up to that point in the file has reached all the datanodes in the write pipeline and is visible to all new readers.
	
  qn 3 3. Explain how HDFS handles failures during file write
  
  1. The pipeline is closed and any packets in the ack queue are added to the front of the data queue.
2. The current block on the good datanodes is given a new identity, which is communicated to the
namenode
3. The failed datanode is removed from the pipeline, and a new pipeline is constructed from the two
good datanodes.
4. The remainder of the block’s data is written to the good datanodes in the pipeline.
5. The namenode notices that the block is under-replicated, and it arranges for a further replica to
be created on another node.
6. As long as dfs.namenode.replication.min replicas (which defaults to 1) are written, the write will
succeed.
7. The block will be asynchronously replicated across the cluster until its target replication factor is
reached (dfs.replication, which defaults to 3).


