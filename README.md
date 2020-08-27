# Knox-workshop

In this workshop we will configure and understand Knox authentication to use with Hadoop services in a cluster. For this workshop We need Kerberos enabled CDP-DC 7.X cluster with Knox installed.

------------------------------------------------------------------------------------------------------------------------------


 - [LAB 1:](https://github.com/bhagadepravin/knox-workshop/blob/master/README.md#lab-1)
   -  Knox Overview
   -  Knox Usecase
   -  Knox Configs

 - [LAB 2:](https://github.com/bhagadepravin/knox-workshop/blob/master/README.md#lab-2)
   -  Install Knox

 - [LAB 3:]()
    -  Generating API and UI topologies in Knox CSD
       -  Adding a known service to cdp-proxy
       -  Adding a custom service parameter to a known service (assuming the service is already enabled; see the previous point)
       -  Updating a custom service parameter
       -  Removing a custom service parameter
       -  Removing a known service from cdp-proxy
       -  Adding a custom service in cdp-proxy
       -  Adding a custom topology in the deployed Knox Gateway


 - [LAB 4:]() 
   -  Managing Knox shared providers in Cloudera Manager
      -  SSO authentication provider
	        *  PAM to LDAP
      -  API Authentication Provider
	        *  PAM to LDAP
      -  Saving Aliases

 - [LAB 5:]() 
   -  Modifying an existing shared provider
      -  Disabling a provider in an existing provider configuration
      -  Modifying a provider in an existing provider configuration
      -  Add a new provider in an existing provider configuration
      -  Adding a new shared provider configuration

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

I] *Fresh 7.x DC Cluster*
  1. Knox is not installed
  2. Cluster is kerberized
  3. Knox is installed
       -  All UIs and APIs are selected for autodiscovery and proxying by Knox by default - admin may opt-out of those unwanted via checkboxes in Knox admin page
       -  Additional config may be required to enable trusted proxy for certain services or put HS2 in HTTP mode, etc.
       -  Any required config changes may require restart of one or more service
  4. URLs for UIs and APIs will be made available as QuickLinks|Home Page|Client Configs
  5. Gateway URLs will be able to be used for UI access
  6. Gateway URLs for API access
       -  HTTP Basic is the default authentication mechanism for proxied API access
       -  By default it authenticates against PAM (again assuming local accounts due to a secure cluster)
       -  Authentication is done at the gateway via the Shiro authentication provider
       -  Upon successful authentication the request is dispatched to the backend service API - propagating the effective user via trusted proxy (kerberos+doAs)

# [Knox configs](https://docs.cloudera.com/cloudera-manager/7.1.1/installation/topics/cdpdc-knox-install-parameters.html)



# LAB 2:

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
 ![Knox Homepage](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/image.png)
 
 
 
 # LAB 3:
 
# Generating API and UI topologies in Knox

In CDP DC there is no Cloudbreak who can deploy cdp-proxy and cdp-proxy-api topologies in Knox. To make it easier for end-users we introduced two new CM configurations where these topologies can be managed within CM:
  -  *Knox Simplified Topology Management - **cdp-proxy***
  -  *Knox Simplified Topology Management - **cdp-proxy-api***

![Knox Simplified Topology Management](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/Knox%20Simplified%20Topology%20Management%20.png)

Each of these configurations describes a Knox topology called **cdp-proxy** and **cdp-proxy-api**, respectively. End-users are allowed to add/remove/update services into/from these topologies as well as creating their own custom topology(ies) by editing the appropriate safety valve (see below).

It is very important that these topologies will be deployed by CM only, and only if Knox’s service auto-discovery feature is turned on using the *Enable/Disable Service Auto-Discovery* checkbox on CM UI:

![Enable/Disable Service Auto-Discovery](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/auto_discovery.png)

As you can see, the Knox CSD comes with some pre-defined service parameters out-of-the-box (e.g. ***CM-API:httpclient.connectionTimeout=5m***), but it does not mean CM-API will be added in ***cdp-proxy*** and/or ***cdp-proxy-api*** topologies. If end-users want a known service (that is, the service is an officially supported Knox service with all the required service definition files) to be included in these topologies they need to enable that service on CM UI using the corresponding gateway_auto_discovery_***[cdp-proxy|cdp-proxy-api]_enabled_$service*** checkbox. For instance, enabling Atlas API and UI services in cdp-proxy end-users should check ***[gateway_auto_discovery_cdp_proxy_enabled_atlas***[ and ***gateway_auto_discovery_cdp_proxy_enabled_atlas_ui***:
