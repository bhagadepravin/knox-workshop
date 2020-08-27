# Knox-workshop

In this workshop we will configure and understand Knox authentication to use with Hadoop services in a cluster. For this workshop We need CDP-DC 7.1 cluster with Knox installed.

------------------------------------------------------------------------------------------------------------------------------


 - [LAB 1:]()
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
 
 **Step 1:**  Setup CDP-DC cluster with latest version.(Use squadron or Ycloud)
 **Step 2:**  Enable Kerberos (using squadron one click script/ Ycloud option)
 **Step 3:**  Add Knox service **[Follow the documentation]**(https://docs.cloudera.com/cloudera-manager/7.1.1/installation/topics/cdpdc-knox-install.html)
 
               -  *We need pass master secret key at the time of installation.*
               -  *The master secret is required to start the server. This secret is used to access secured artifacts by the gateway instance. By default, the keystores, trust stores, and credential stores are all protected with the master secret.*
               - *It is encrypted with AES 128 bit encryption and where possible the file permissions are set to only be accessible by the user that the gateway is running as.*
 
Once Knox is Installed.

 -  By default, Knox authentication for default topology are set to PAM.
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
