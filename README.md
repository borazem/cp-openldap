## OpenLDAP

Installs OpenLDAP and phpLDAPadmin with a small number of initial users for the purposes of demonstrating LDAP integration capabilities of IBM Cloud Paks.

Forked from https://github.com/ibm-cloud-architecture/icp-openldap

Updated to work with new versions of Kubernetes (>=1.16.x)

## Installation
To install the chart, you'll need the [helm cli](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/app_center/create_helm_cli.html?view=kc) and the [IBM Cloud Private CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/manage_cluster/install_cli.html?view=kc). Note: the IBM Cloud Private CLI version level must match the version level that is downloadable via your ICP console, under ***Menu > Command Line Tools > Cloud Private CLI***.

1. Get the source code of the helm chart
   `git clone https://github.com/dymaczew/cp-openldap.git`
2. Package the helm chart using the helm cli
   `helm package cp-openldap`
3. If you have not, log in to your cluster from the IBM® Cloudak CLI and log in to the Docker private image registry.
   `cloudctl login -a https://<cluster_domain> --skip-ssl-validation`
4a. If you have Tiller available (CloudPak for MCM 1.3 or 2.0) install the Helm chart using helm v2 cli 
   `helm install --name ldap-slap --namespace kube-public --values cp-openldap/values.yaml --tls`
4b. For other CloudPaks render the chart and apply using oc apply
   `helm template --name ldap-slap --namespace kube-public --values cp-openldap/values.yaml | kubectl apply -f -`

## Assets

Kubernetes Assets in this chart.

**General Settings**
General settings

default values below

```
General:
  project: "ldap"
  serviceAccount: "ldap"
  sccName: "ldapanyuidscc"
```

**OpenLDAP**
OpenLDAP

see details in [official site](http://www.openldap.org/)

default values below

```
OpenLdap:
  Image: "docker.io/osixia/openldap"
  ImageTag: "1.1.10"
  ImagePullPolicy: "Always"
  Component: "openldap"

  InitImage: "docker.io/busybox"
  InitImageTag: "1.30.1"
  InitImagePullPolicy: "Always"

  Replicas: 1

  Cpu: "512m"
  Memory: "200Mi"

  Domain: "local.io"
  AdminPassword: "admin"
  Https: "false"
  SeedUsers: 
    usergroup: "icpusers"
    userlist: "user1,user2,user3,user4"
    initialPassword: "ChangeMe"
```

**phpLDAPadmin**
LDAP admin UI

see details in [official site](http://phpldapadmin.sourceforge.net/)

default values below
```
PhpLdapAdmin:
  Image: "docker.io/osixia/phpldapadmin"
  ImageTag: "0.7.0"
  ImagePullPolicy: "Always"
  Component: "phpadmin"

  Replicas: 1

  NodePort: 31080

  Cpu: "512m"
  Memory: "200Mi"
```

## Setup IBM Cloud Private LDAP integration

Detailed information about LDAP support in ICP avilable on the [IBM KnowledgeCenter](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/user_management/configure_ldap.html)

After the chart is deployed, follow these steps to setup LDAP authentication
 
 1. From the helm release page take note of the OpenLDAP cluster ip and port for your deployment
 2. Navigate to ***Manage > Authentication*** and insert the following details
    #### LDAP Connection
    - Name: `ldap`
    - Type: `Custom`
    - URL: `ldap://<cluster-ip>:389`
    
    #### LDAP authentication
    - Base DN: `dc=local,dc=io` (default value, adjust as needed)
    - Bind DN: `cn=admin,dc=local,dc=io` (default value, adjust as needed)
    - Admin Password: `admin` (default value, adjust as needed)
    
    #### LDAP Filters
    - Group filter: `(&(cn=%v)(objectclass=groupOfUniqueNames))`
    - User filter: `(&(uid=%v)(objectclass=person))`
    - Group ID map: `*:cn`
    - User ID map: `*:uid`
    - Group member ID map: `groupOfUniqueNames:uniquemember`

    Click ***Save***
    
 3. Add users or groups to teams by navigating to ***Manage > Teams***
    - Click ***Create team***
    - Search group or user names to add, and select appropriate roles for each
    

## Credit

Inspired by work done by the [Samsung Cloud Native Computing Team](https://github.com/samsung-cnct) .
