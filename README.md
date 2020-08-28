# Knox-workshop

In this workshop we will configure and understand Knox authentication to use with Hadoop services in a cluster. For this workshop We need Kerberos enabled CDP-DC 7.X cluster with Knox installed.

------------------------------------------------------------------------------------------------------------------------------


 - [LAB 1:](https://github.com/bhagadepravin/knox-workshop/blob/master/README.md#lab-1)
   -  Knox Overview
   -  Knox Usecase
   -  Knox Configs

 - [LAB 2:](https://github.com/bhagadepravin/knox-workshop/blob/master/README.md#lab-2)
   -  Install Knox

 - [LAB 3:](https://github.com/bhagadepravin/knox-workshop/blob/master/README.md#lab-3)
    -  Generating API and UI topologies in Knox CSD
       -  Adding a known service to cdp-proxy
       -  Adding a custom service parameter to a known service (assuming the service is already enabled)
       -  Updating a custom service parameter
       -  Removing a custom service parameter
       -  Removing a known service from cdp-proxy
       -  Adding a custom service in cdp-proxy
       -  Adding a custom topology in the deployed Knox Gateway


 - [LAB 4:](https://github.com/bhagadepravin/knox-workshop/blob/master/README.md#lab-4) 
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

Each of these configurations describes a Knox topology called **cdp-proxy** and **cdp-proxy-api**, respectively. 
End-users are allowed to add/remove/update services into/from these topologies as well as creating their own custom topology(ies) by editing the appropriate safety valve (see below).

It is very important that these topologies will be deployed by CM only, and only if Knox’s service auto-discovery feature is turned on using the *Enable/Disable Service Auto-Discovery* checkbox on CM UI:

![Enable/Disable Service Auto-Discovery](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/auto_discovery.png)

As you can see, the Knox CSD comes with some pre-defined service parameters out-of-the-box (e.g. ***CM-API:httpclient.connectionTimeout=5m***), but it does not mean CM-API will be added in ***cdp-proxy*** and/or ***cdp-proxy-api*** topologies. 

If end-users want a known service (that is, the service is an officially supported Knox service with all the required service definition files) to be included in these topologies they need to enable that service on CM UI using the corresponding gateway_auto_discovery_***[cdp-proxy|cdp-proxy-api]_enabled_$service*** checkbox. 

For instance, enabling Atlas API and UI services in cdp-proxy end-users should check ***[gateway_auto_discovery_cdp_proxy_enabled_atlas***[ and ***gateway_auto_discovery_cdp_proxy_enabled_atlas_ui***:

![auto_discovery_cdp_proxy_enabled_atlas](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/auto_discovery_cdp_proxy_enabled_atlas.png)

If end-users want a custom service *(that is, the service is not officially supported by the Knox team, but end-users have their own service definition files)* in **cdp-proxy** and/or **cdp-proxy-api** topologies they simply need to add the service declaration in any of the above shown *Knox Simplified Topology Management* panels.

In the following samples, I’ll guide you on how to **add/remove/update** known and custom services to **cdp-proxy**.


## 1. Adding a known service to *cdp-proxy*

In this sample, we are going to add **ATLAS** and **ATLAS UI** to *cdp-proxy*. You should simply enable *gateway_auto_discovery_cdp_proxy_enabled_atlas* and *gateway_auto_discovery_cdp_proxy_enabled_atlas_ui* checkboxes on Knox’s Configuration page in CM and save the changes. 
As a result, the **'Refresh needed'** stale configuration indicator appears. 
You should click it and wait until the refresh process finished.

![knox_atlas](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/knox-atlas.png)

You can check if **ATLAS** and **ATLAS-UI** were added to *cdp-proxy* by hitting the following URL:
(https://$KNOX_GATEWAY_HOST:PORT/$GATEWAY_PATH/admin/api/v1/topologies/cdp-proxy)


## 2. Adding a custom service parameter to a known service (assuming the service is already enabled; see the previous point)

In this sample, we are going to add a custom service parameter with a custom value (*myCustomServiceParameter=myValue*) to ATLAS in cdp-proxy.

It is as simple as adding a new line in the ***Knox Simplified Topology Management - cdp-proxy*** panel in the following format: **$SERVICE_NAME[:$PARAMETER_NAME=$PARAMETER_VALUE]**.
The 'url' and 'version' parameter names are preserved keywords to set the given service's URL and version. Valid declarations:

```bash
HIVE:url=http://localhost:123
HIVE:version:3.0.0
HIVE:test.pramameter.name=test.parameter.value
```

After you added the new ATLAS entry and saved the changes the **'Refresh needed'** stale configuration indicator appears. You should click it and wait until the refresh process finished.

![ATLAS-myCustomServiceParameter](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/ATLAS-myCustomServiceParameter.png)
![atlas-cdp-resources](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/atlas-cdp-resources.png)


You can confirm if **ATLAS** in *cdp-proxy* got updated with the new service parameter the same way as described above.
You can check by hitting the following URL:

(https://$KNOX_GATEWAY_HOST:PORT/$GATEWAY_PATH/admin/api/v1/topologies/cdp-proxy)


## 3. Updating a custom service parameter

In this sample, we are going to update a previously entered service parameter - ***myCustomServiceParameter=myValue*** to ***myNewValue***- from **ATLAS** in *cdp-proxy*. We simply change that entry, save our changes and refresh our cluster.


## 4. Removing a custom service parameter

In this sample, we are going to remove a previously entered service parameter - ***myCustomServiceParameter=myNewValue*** - from **ATLAS** in *cdp-proxy*. We simply remove that entry, save our changes and refresh our cluster.


## 5. Removing a known service from cdp-proxy

In this sample, we are going to remove the previously added **ATLAS** and **ATLAS-UI** services from *cdp-proxy*. We disable the *gateway_auto_discovery_cdp_proxy_enabled_atlas* and *gateway_auto_discovery_cdp_proxy_enabled_atlas_ui* checkboxes on Knox’s Configuration page in CM, save the changes and refresh the cluster.


## 6. Adding a custom service in cdp-proxy

In this sample, we are going to add a custom service (MY_SERVICE) in *cdp-proxy* with the following attributes:
-  version - the service’s version. We will set it to 1.0.0
-  url - the service URL. We will set it to https://sampleHost:1234
-  service parameter - a sample service parameter. We will define a custom service parameter just like above for ATLAS

It is very important to note that adding a custom service only makes sense if the service definition files ***(service.xml and rewrite.xml)*** are existing in the ***KNOX_DATA_DIR/services*** folder.

To achieve the goals we need to add 3 new entries with the above-listed parameters in ***Knox Simplified Topology Management - cdp-proxy***. Then we save the changes, refresh the cluster and check if the newly added custom service is available in *cdp-proxy*.

![knox-custom](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/%20custom%20service.png)


## 7. Adding a custom topology in the deployed Knox Gateway

In this sample, we are going to add a custom service (MY_SERVICE) in custom-topology with the following attributes:
-  **providerConfigRef** - a string representing a reference of an existing share-provider. We will use the pre-configured pam provider
-  **version** - the service’s version. We will set it to 1.0.0
-  **url** - the service URL. We will set it to *https://sampleHost:1234*
-  **service parameter** - a sample service parameter. We will define a custom service parameter just like above for ATLAS

To achieve our goals we need to add a new entry in Knox Gateway Advanced Configuration Snippet (Safety Valve) for conf/cdp-resources.xml:

```bash
Name = custom-topology
Value =providerConfigRef=pam#
MY_SERVICE:version=1.0.0#
MY_SERVICE:url=https://sampleHost:1234#
MY_SERVICE:myCustomServiceParameter=myValue
#Description = This is a custom topology with one service called MY_SERVICE
```

In addition to the above-described attributes, you can also define service-discovery related configuration in your custom descriptor:
-  ***discoveryType*** - the type of service discovery. Valid values are: ClouderaManager, Ambari
-  ***discoveryAddress*** - the service-discovery end-point
-  ***discoveryUser*** - the user name which the service discovery feature uses to connect to the previously set address. This is optional, if not present Knox will try to fetch the discovery user using the previously saved cm.discovery.user Gateway level alias (CM generates it automatically in case service discovery is enabled).
-  ***discoveryPasswordAlias*** - the password alias (on Gateway level) the service discovery uses to get the connection password using Knox’s alias service. Please note that the alias should be created in advance! This is optional, if not present Knox will try to fetch the discovery password using the previously saved cm.discovery.password Gateway level alias (CM generates it automatically in case service discovery is enabled).
-  ***cluster*** - the cluster to be discovered

# LAB 4:

# Managing Knox shared providers in Cloudera Manager

-  Modifying the SSO authentication provider used by the UIs using the Knox SSO capabilities such as the Admin and Home Page UIs
-  Modifying the API authentication provider used by pre-defined topologies such as admin, metadata or cdp-proxy-api
-  Adding/modifying new/existing shared provider configurations
-  Saving aliases using a new Knox Gateway command

## 1. SSO authentication provider

With the newest version of CM a new Knox configuration has been added, called ***Knox Simplified Topology Management - SSO Authentication Provider***, with the following initial configuration:

```bash
role=authentication
authentication.name=ShiroProvider
authentication.param.sessionTimeout=30
authentication.param.redirectToUrl=/${GATEWAY_PATH}/knoxsso/knoxauth/login.html
authentication.param.restrictedCookies=rememberme,WWW-Authenticate
authentication.param.urls./**=authcBasic
authentication.param.main.pamRealm=org.apache.knox.gateway.shirorealm.KnoxPamRealm
authentication.param.main.pamRealm.service=login
```
![sso](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/sso.png)


Every change here goes directly into **knoxsso** topology that affects **manager**, **homepage** and **cdp-proxy** topologies as they are using the federation provider.

In the following sample you will see how to change the PAM authentication *(which comes OOTB with Knox)* to LDAP authentication. It is as simple as replacing the PAM related configuration entries in the above ShiroProvider to LDAP related properties (e.g. with demo LDAP server configuration):

![knox-sso](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/knoxsso.png)


```bash
role=authentication
authentication.name=ShiroProvider
authentication.param.sessionTimeout=30
authentication.param.redirectToUrl=/${GATEWAY_PATH}/knoxsso/knoxauth/login.html
authentication.param.restrictedCookies=rememberme,WWW-Authenticate
authentication.param.urls./**=authcBasic
authentication.param.main.ldapRealm=org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm
authentication.param.main.ldapContextFactory=org.apache.knox.gateway.shirorealm.KnoxLdapContextFactory
authentication.param.main.ldapRealm.contextFactory=$ldapContextFactory
authentication.param.main.ldapRealm.contextFactory.authenticationMechanism=simple
authentication.param.main.ldapRealm.contextFactory.url=ldap://localhost:33389
authentication.param.main.ldapRealm.contextFactory.systemUsername=uid=guest,ou=people,dc=hadoop,dc=apache,dc=org
authentication.param.main.ldapRealm.contextFactory.systemPassword=${ALIAS=knoxLdapSystemPassword}
authentication.param.main.ldapRealm.userDnTemplate=uid={0},ou=people,dc=hadoop,dc=apache,dc=org
```

```
# Start Demo LDAP
get the parcel location from Knox Node:

$ ps aux | grep knox
$ /opt/cloudera/parcels/CDH-7.2.1-1.cdh7.2.1.p0.4847773/lib/knox/bin/ldap.sh start
```


After you finished editing the properties you have to save the configuration changes. This will make the **Refresh Needed** stale configuration indicator appear. Once the cluster refresh finishes, all topologies that are configured to use Knox SSO will be authenticated by the configured LDAP server.

![knox-ldap](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/ldap.png)


## 2. API Authentication Provider

Similarly to the previous point, a new Knox configuration has been added, called ***,Knox Simplified Topology Management - API Authentication Provider***, with the following initial configuration:

```
role=authentication
authentication.name=ShiroProvider
authentication.param.sessionTimeout=30
authentication.param.urls./**=authcBasic
authentication.param.main.pamRealm=org.apache.knox.gateway.shirorealm.KnoxPamRealm
authentication.param.main.pamRealm.service=login
```

Every change here goes directly into **admin**, **metadata** and **cdp-proxy-api** topologies.
Since the only difference between this initial configuration and the one Knox SSO uses is two additional parameters **(redirectToUrl and restrictedCookies)** I would not like to repeat the same steps on how to change PAM authentication to LDAP as they are exactly the same.


![sso-api](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/sso-api.png)

https://github.com/bhagadepravin/commands/blob/master/knox-cdp-dc.md


## 3. Saving Aliases

There is a new command available for the Knox Gateway role which allows end-users to save an **alias=password** pair to an arbitrary number of topologies on each host where an instance of the Knox Gateway is installed without the need of running the Knox CLI tool manually.

A new password-type input field is added, called *save_alias_command_input_password*. The format of an entry in this input field should be:
***topology_name_1[:topology_name_2:...:topology_name_N].alias_name=password***

Sample: **cdp-proxy-api:admin:metadata.knoxLdapSystemPassword=guest-password**

After the end-user entered a meaningful and valid value and saved the configuration changes he/she can run the command from Knox’s action list: **Actions/Save Alias**

Tip: if you need to add a Gateway level alias, please use ***__gateway*** as topology  name. For instance: ***__gateway.knoxLdapSystemPassword=admin-password***

![save-alias](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/save%20alias.png)
![alias-input](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/save%20alias%20input.png)
![alias-action](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/action.png)
![save-alias](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/save-alias.png)
![alias-log](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/save%20alias%20cmds.png)
![admin-jceks](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/admin-credentials.png)
![gateway-jceks](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/gateway-credentials.png)


# LAB 5:

# Modifying an existing shared provider

At the time of this document being written, the following shared provider configurations are deployed in CDP DC with Knox OOTB (pam and sso are available only if service auto-discovery is enabled fo Knox Gateway role):
*admin* (used by the *admin* topology)
*homepage* (used by the *homepage* topology)
*knoxsso* (*homepage*, *manager* and *cdp-proxy* topologies are configured to use this one)
*manager* (used by the *manager* topology)
*metadata* (used by the *metadata* topology)
*pam* (used by the *cdp-proxy* topology)
*pam* (used by the *cdp-proxy-api* topology)

The following changes are allowed in any of these shared providers:
-  disable a particular provider
-  modify a particular provider
-  add a new provider

All of these actions can be done via editing the ***Knox Gateway Advanced Configuration Snippet (Safety Valve) for conf/cdp-resources.xml*** by implementing the following language elements:


-  the key of a new entry should be like this: *providerConfigs: providerConfig_1 [,providerConfig_2,..,providerConfig_3]*
-  the value should contain the following *name/value* pairs separated by a hash (#) character:
    -  $*role=webappsec|authentication|federation|identity-assertion|authorization|hostmap|ha*
    -  *$role.name=ROLE_NAME (e.g. ShiroProvider)*
    -  *$role.enabled=true|false (optional; defaults to 'true')*
    -  *$role.param.param_1=value_1 (parameters are optional too).*   
    -  *$role.param_N.param1=value_N*


**Step 1** Disabling a provider in an existing provider configuration

In this sample you will see how to disable the **authorization** provider in the **manager** shared provider configuration. This particular authorization provider is set as follows (in its JSON descriptor):
```bash
{
         "role": "authorization",
         "name": "AclsAuthz",
         "enabled": "true",
         "params": {
            "knox.acl.mode": "OR",
            "knox.acl": "KNOX_ADMIN_USERS;KNOX_ADMIN_GROUPS;*"
         }
      }
```

To disable it the following should be done:
  -  1. add the following entry in the above-referenced safety valve:
     -  name = *providerConfigs:manager*
     -  value = *role=authorization#authorization.name=AclsAuthz#authorization.enabled=**false**#authorization.param.knox.acl=KNOX_ADMIN_USERS;KNOX_ADMIN_GROUPS;*#authorization.param.knox.acl.mode=OR*
  -  2.  save your changes
  -  3.  refresh the cluster

As you can see only the enabled flag was changed.

![manager1](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/manager1.png)
![manager2](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/manager2.png)
![manager3](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/manager3.png)


**Step 2**.  Modifying a provider in an existing provider configuration

In this sample you will see how to modify the previously disabled authorization provider in the manager shared provider configuration.
The steps are the same as described in the section before but the value of the safety valve entry is different:

*role=authorization#authorization.name=AclsAuthz#authorization.enabled=false#authorization.param.knox.acl=***myTestUser***;KNOX_ADMIN_GROUPS;*#authorization.param.knox.acl.mode=OR*


With this change you are authorizing a user called **myTestUser** to login and execute administrative actions on the Knox Admin UI.


**Step 3**.  Add a new provider in an existing provider configuration

As of now, the **manager** shared provider configuration does not have the **HA** provider set.
In this sample you will see how to add a new **HA** provider (this time only the **ATLAS** service will be configured for high availability) in the **manager** shared provider configuration with keeping all the changes from the previous sections.

As you might figured it out, the steps are the same again except for the value in the safety valve, which has to be set to this one:

*role=authorization#authorization.name=AclsAuthz#authorization.enabled=false#authorization.param.knox.acl=myTestUser;KNOX_ADMIN_GROUPS;*#authorization.param.knox.acl.mode=OR#***role=ha#ha.name=HaProvider#ha.param.ATLAS=enabled=true;maxFailoverAttempts=3;failoverSleep=1000;maxRetryAttempts=300;retrySleep=1000****


**Step 4**. Adding a new shared provider configuration

It is possible that you add a brand new shared provider configuration too. In this sample you will see how to create **testProviders** with the following providers set:

-  *authentication: ShiroProvider / PAM*
-  *identity-assertion: Default*
-  *authorization: Ranger (XASecurePDPKnox)*

The following should be added in the :
  -  1.  add the following entry in the above-referenced safety valve:
         -  name  = *providerConfigs:testProviders*
         -  value =  *role=authentication#authentication.name=ShiroProvider#authentication.param.main.pamRealm=org.apache.knox.gateway.shirorealm.KnoxPamRealm#authentication.param.main.pamRealm.service=login#role=identity-assertion#identity-assertion.name=Default#role=authorization#authorization.name=XASecurePDPKnox*
  -  2.  save your changes
  -  3.  refresh the cluster

Since all providers you add via this mechanism are **enabled** by default I shortened the value field by omitting the enabled flag.

```bash
curl -ik -u knoxui:knoxui "https://pbhagade-boo-1.pbhagade-boo.root.hwx.site:8443/gateway/admin/api/v1/providerconfig/testProviders"
```

![testProviders](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/testProviders.png)
![testProviders2](https://github.com/bhagadepravin/knox-workshop/blob/master/jpeg/test-shiroprovider.png)



