# Knox-workshop

In this workshop we will configure and understand Knox authentication to use with Hadoop services in a cluster. For this workshop We need CDP-DC 7.1 cluster with Knox installed.

The Apache Knox Gateway is a system that provides a single point of authentication and access for Apache Hadoop services in a cluster. The goal is to simplify Hadoop security for both users (i.e. who access the cluster data and execute jobs) and operators (i.e. who control access and manage the cluster). The gateway runs as a server (or cluster of servers) that provide centralized access to one or more Hadoop clusters. In general the goals of the gateway are as follows:

  - Provide perimeter security for Hadoop REST APIs to make Hadoop security easier to setup and use
 - Provide authentication and token verification at the perimeter
Enable authentication integration with enterprise and cloud identity management systems
Provide service level authorization at the perimeter
Expose a single URL hierarchy that aggregates REST APIs of a Hadoop cluster
Limit the network endpoints (and therefore firewall holes) required to access a Hadoop cluster
Hide the internal Hadoop cluster topology from potential attackers
