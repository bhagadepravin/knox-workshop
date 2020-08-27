# Knox-workshop

In this workshop we will configure and understand Knox authentication to use with Hadoop services in a cluster. For this workshop We need CDP-DC 7.1 cluster with Knox installed.

------------------------------------------------------------------------------------------------------------------------------


 - [LAB 1:]()
   -  Knox Overview
   -  Knox Usecase
   -  Install Knox

 - [LAB 2:]()
   -  Knox Uses Cases

 - [LAB 3:]() 
   -  Managing Knox shared providers in Cloudera Manager
      -  SSO authentication provider
	        *  PAM to LDAP
      -  API Authentication Provider
	        *  PAM to LDAP
      -  Saving Aliases

 - [LAB 4:]() 
   -  Modifying an existing shared provider
      -  Disabling a provider in an existing provider configuration
      -  Modifying a provider in an existing provider configuration
      -  Add a new provider in an existing provider configuration
      -  Adding a new shared provider configuration

 - [LAB 5:]() 
   -  Generating API and UI topologies in Knox CSD
      -  Adding a known service to cdp-proxy
      -  Adding a custom service parameter to a known service (assuming the service is already enabled; see the previous point)
      -  Updating a custom service parameter
      -  Removing a custom service parameter
      -  Removing a known service from cdp-proxy
      -  Adding a custom service in cdp-proxy
      -  Adding a custom topology in the deployed Knox Gateway

 - [LAB 6:]() 
   -  Replace knox ssl certificate
   -  Knox SSO with keycloak

 - [LAB 7:]() 
   -  Troubleshooting
      -  How authentication works and its logging 
      -  Error codes
      -  How Knoxsso works, and its logging
      -  Ranger knox plugin sync issues
      -  Knox SSL issues

# LAB 1: 

# Knox Overview

![Knox](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/knox-services.png)

-  Knox can only be deployed in secure clusters with kerberos enabled
-  Knox will need to be explicitly installed - it is NOT installed by default in secure clusters
-  We will have two topologies
   -  cdp-proxy for UI access
   -  cdp-proxy-api for API access
-  These two topologies are managed in CM
   -  sets of topology specific checkboxes to include UIs and APIs within the topology and enable CM based discovery of them by Knox
   -  Provider configuration to specify the authentication and authorization provider configuration for each topology is also managed within CM
-  The same trusted proxy model used in CDP public cloud will be leveraged in DC  
-  Knox itself does nothing to block direct access to UIs and APIs but it is common for clusters to be surrounded by a firewall of one sort or another and access to the internal resources of the cluster to go through Knox. At the end of the day, direct access by external clients is a deployment decision for DC unlike CDP public cloud.
  -  once inside the cluster, there is direct access to those resources for which you have line of sight
-  We have Knox Homepage which provide a list of UI and API endpoints. 

# Knox Usecase:

I] Fresh 7.1 DC Cluster
    1. Knox is not installed
    2. Cluster is kerberized
    3. Knox is installed
       -  All UIs and APIs are selected for autodiscovery and proxying by Knox by default - admin may opt-out of those unwanted via checkboxes in Knox admin page
       -  Additional config may be required to enable trusted proxy for certain services or put HS2 in HTTP mode, etc.
       -  Any required config changes may require restart of one or more service
    4. URLs for UIs and APIs will be made available as QuickLinks|Home Page|Client Configs
    5. Gateway URLs will be able to be used for UI access
    6. Gateway URLs for API access
       - a.  HTTP Basic is the default authentication mechanism for proxied API access
       - b.  by default it authenticates against PAM (again assuming local accounts due to a secure cluster)
       - c.  Authentication is done at the gateway via the Shiro authentication provider
       - d.  Upon successful authentication the request is dispatched to the backend service API - propagating the effective user via trusted proxy (kerberos+doAs)

# Install Knox
 
 **Step 1:**  Setup CDP-DC cluster with latest version.(Use squadron or Ycloud)
 
 **Step 2:**  Enable Kerberos (using squadron one click script/ Ycloud option)
 
 **Step 3:**  Add Knox service Follow the documentation. 
 
[Doc: Installing Apache Knox!](https://docs.cloudera.com/cloudera-manager/7.1.1/installation/topics/cdpdc-knox-install.html)

- *We need pass master secret key at the time of installation.*
-  *The master secret is required to start the server. This secret is used to access secured artifacts by the gateway instance. By default, the keystores, trust stores, and credential stores are all protected with the master secret.*
- *It is encrypted with AES 128 bit encryption and where possible the file permissions are set to only be accessible by the user that the gateway is running as.*
 
Once Knox is Installed.

 -  By default, Knox authentication for default topologies are set to PAM. 
 
 **Step 4:** We need to create a pam user and he should be part of admin group on all Knox Host.
 
 ```sh
$ vi /etc/pam.d/cdp-dc

#%PAM-1.0
auth sufficient pam_unix.so
auth sufficient pam_sss.so
account sufficient pam_unix.so
account sufficient pam_sss.so


$ vi /etc/pam.d/cdp-dc-remote

#%PAM-1.0
auth sufficient pam_unix.so
auth sufficient pam_sss.so
account sufficient pam_unix.so
account sufficient pam_sss.so

$ chmod 444 /etc/shadow

# create a user who is part of admin group

$ groupadd admin
$ useradmin <username>
$ usermod -a -G admin <username>
$ id <username>
 ```
 
 **Step 5:** Use above username and password to login into Knox Homepage UI or Admin page UI.
