# Knox-workshop

In this workshop we will configure and understand Knox authentication to use with Hadoop services in a cluster. For this workshop We need CDP-DC 7.1 cluster with Knox installed.

The Apache Knox Gateway is a system that provides a single point of authentication and access for Apache Hadoop services in a cluster. The goal is to simplify Hadoop security for both users (i.e. who access the cluster data and execute jobs) and operators (i.e. who control access and manage the cluster). The gateway runs as a server (or cluster of servers) that provide centralized access to one or more Hadoop clusters. In general the goals of the gateway are as follows:

 - Provide perimeter security for Hadoop REST APIs to make Hadoop security easier to setup and use
 - Provide authentication and token verification at the perimeter
 - Enable authentication integration with enterprise and cloud identity management systems
 - Provide service level authorization at the perimeter
 - Expose a single URL hierarchy that aggregates REST APIs of a Hadoop cluster
 - Limit the network endpoints (and therefore firewall holes) required to access a Hadoop cluster
 - Hide the internal Hadoop cluster topology from potential attackers

------------------------------------------------------------------------------------------------------------------------------


 - [LAB 1:]()
   -  Install Knox

 - (LAB 2:) 
   -  Knox Uses Cases

 - LAB 3: 
   -  Managing Knox shared providers in Cloudera Manager
      -  SSO authentication provider
	        *  PAM to LDAP
      -  API Authentication Provider
	        *  PAM to LDAP
      -  Saving Aliases

 - LAB 3: 
   -  Modifying an existing shared provider
      -  Disabling a provider in an existing provider configuration
      -  Modifying a provider in an existing provider configuration
      -  Add a new provider in an existing provider configuration
      -  Adding a new shared provider configuration

 - LAB 4: 
   -  Generating API and UI topologies in Knox CSD
      -  Adding a known service to cdp-proxy
      -  Adding a custom service parameter to a known service (assuming the service is already enabled; see the previous point)
      -  Updating a custom service parameter
      -  Removing a custom service parameter
      -  Removing a known service from cdp-proxy
      -  Adding a custom service in cdp-proxy
      -  Adding a custom topology in the deployed Knox Gateway

 - LAB 5: 
   -  Replace knox ssl certificate
   -  Knox SSO with keycloak

 - LAB 6: 
   -  Troubleshooting
      -  How authentication works and its logging 
      -  Error codes
      -  How Knoxsso works, and its logging
      -  Ranger knox plugin sync issues
      -  Knox SSL issues
