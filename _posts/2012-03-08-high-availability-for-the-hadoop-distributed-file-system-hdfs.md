---
layout: post
title: "High Availability for the Hadoop Distributed File System (HDFS)"
category: Reading Notes 
tags: ["读文章", "HDFS", "分布式系统"]
---
{% include JB/setup %}

1. HDFS, the Hadoop Distributed File System, is the primary storage system of Hadoop, and is responsible for storing and serving all data stored in Hadoop. MapReduce is a distributed processing framework designed to operate on data stored in HDFS.

2. Despite very high level of reliability, HDFS has always had a well-known single point of failure which impacts HDFS's availability: the system relies on a single Name Node to coordinate access to the file system data.

3. In the case of HBase, used to directly serve customer requests in real time. Adding high availability(HA) to the HDFS Name Node became one of the top priorities for the HDFS community.

4. The remainder of this post discuss the implementation of a new feature for HDFS, called the "HA Name Node".

5. The goal of the HA Name Node project is to add support for deploying two Name Nodes in an active/passive configuration. This is a common configuration for highly-available distributed systems, and HDFS's architecture lends itself well to this design. Even in a non-HA configuration, HDFS already requires both a Name Node and another node with similar hardware specs which performs checkpointing operations for the Name Node.

6. The HDFS Name Node is primarily responsible for serving two types of file system metadata: file system namespace information and block locations.

7. All mutations to the file system namespace, such as file renames, permission changes, file creations, block allocation, etc, are written to a persistent write-ahead log by the Name Node before returning success to a client call. 

8. In addition to this edit log, periodic checkpoints of the file system, called the fsimage, are also created and stored on-disk on the Name Node. 

9. Block locations are stored only in memory.

10. The locations of all blocks are received via "block reports" sent from the Data Nodes when the Name Node is started.

11. The goal of the HA Name Node is to provide a hot standby Name Node that can take over serving the role of the active Name Node with no downtime.

12. Empirically, starting a Name Node from cold state can take tens of minutes to load the namespace information (fsimage and edit log)from disk, and up to an hour to receive the necessary block reports from all Data Nodes in a large cluster.

13. To address the issue of sharing state between the active and standby Name Nodes, the HA Name Node feature allows for the configuration of a special shared edits directory. This directory should be available via a network file system, and should be read/write accessible from both Name Nodes. This directory is treated as being required by the active Name Node, meaning that success will not be returned to a client call unless the file system change has been written to the edit log in this directory. The standby Name Node polls the shared edits directory frequently, looking for new edits written by the active Name Node, and reads these edits into its own in-memory view of the file system state.

14. The design of the HA Name Node is such that the passive Name Node is capable of performing this checkpointing role, thus requiring no additional Hadoop server machines beyond what HDFS already requires.

15. Requiring a single shared edits directory does not necessarily imply a new single point of failure.

16. The other part of keeping the standby Name Node hot is making sure that it has up-to-date block location information. 

17. Since block locations aren't written to the Name Node edit log, reading from the shared edits directory is not sufficient to share this file system metadata between the two Name Nodes. To address this issue, when HA is enabled, all Data Nodes in the cluster are configured with the network addresses of both Name Nodes.
