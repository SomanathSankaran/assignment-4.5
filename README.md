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
