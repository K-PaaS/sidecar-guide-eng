### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Sidecar

## Table of Contents

1. [Document Outline](#1)  
  1.1. [Purpose](#1.1)  
  1.2. [Range](#1.2)  
  1.3. [References](#1.3)  

2. [Portal Installation](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [Variable Setting](#2.2)  
  2.3. [Infrastructure Deployment](#2.3)  
  2.4. [Portal App Deployment](#2.4)  

# <div id='1'> 1. Document Outline
## <div id='1.1'> 1.1. Purpose
The purpose of this document is to provide a guide on installing Sidecar Portal in PaaS-TA Sidecar(hereinafter Sidecar) environment.

<br>

## <div id='1.2'> 1.2. Range
The installation range was prepared based on the Portal infrastructure's installation and the Portal App's deployment to verify the Sidecar Portal.  

<br>


## <div id='1.3'> 1.3. References
Paketo Buildpack Github : [https://github.com/paketo-buildpacks](https://github.com/paketo-buildpacks)  
Paketo Buildpack Document : [https://paketo.io/docs/](https://paketo.io/docs/)  

<br>


# <div id='2'> 2. Portal Installation
## <div id='2.1'> 2.1. Prerequisite
This installation guide deployed Portal Infrastructure (DB, Storage) in the form of a POD in the Kubernetes environment, and the Portal App deploys it to Sidecar.
<br>

## <div id='2.2'> 2.2. Variable Setting  
Set variables of Infra and Portal deployed in the form of POD.
- Portal Variable
```
$ cd $HOME/sidecar-deployment/install-scripts/portal
$ vi portal-app-variable.yml

#!/bin/bash

SIDECAR_VALUES_PATH=~/sidecar-deployment/install-scripts/manifest/sidecar-values.yml     # sidecar-values.yml Path
PORTAL_APP_WORKING_DIRECTORY=~/sidecar-deployment/install-scripts/portal/portal-app # Portal APP Working Path


##PORTAL VARIABLE
USER_APP_SIZE_MB=0                                      # USER My App size(MB), if value==0 -> unlimited
MONITORING_ENABLE=false                                 # Monitoring Enable Option

PORTAL_ORG_NAME="portal"                                # PaaS-TA Portal Org Name
PORTAL_SPACE_NAME="system"                              # PaaS-TA Portal Space Name
PORTAL_QUOTA_NAME="portal_quota"                        # PaaS-TA Portal Quota Name
PORTAL_SECURITY_GROUP_NAME="portal"                     # PaaS-TA Portal Security Group Name

MAIL_SMTP_HOST="smtp.gmail.com"                         # Mail-SMTP Host
MAIL_SMTP_PORT="465"                                    # Mail-SMTP Port
MAIL_SMTP_USERNAME="paasta"                             # Mail-SMTP User Name
MAIL_SMTP_PASSWORD="paasta"                             # Mail-SMTP Password
MAIL_SMTP_USEREMAIL="paas-ta@gmail.com"                 # Mail-SMTP User Email

SSH_ENABLE=false                                        # true or false
TAIL_LOG_INTERVAL=250                                   # second



##PORTAL APP INSTANCES
PORTAL_API_INSTANCE=1                                   # PORTAL-API INSTANCES
PORTAL_COMMON_API_INSTANCE=1                            # PORTAL-COMMON-API INSTANCES
PORTAL_GATEWAY_INSTANCE=1                               # PORTAL-GATEWAY INSTANCES
PORTAL_REGISTRATION_INSTANCE=1                          # PORTAL-REGISTRATION INSTANCES
PORTAL_STORAGE_API_INSTANCE=1                           # PORTAL-STORAGE-API INSTANCES
PORTAL_WEB_ADMIN_INSTANCE=1                             # PORTAL-WEB-ADMIN INSTANCES
PORTAL_WEB_USER_INSTANCE=1                              # PORTAL-WEB-USER INSTANCES


##UNCHANGE VARIABLE(if defulat install, don't change variable)
API_TYPE=sidecar
UAA_ADMIN_CLIENT_ID="admin"				# UAA Client ID
CC_DB_NAME="cloud_controller"				# PaaS-TA AP CCDB Name
CC_DB_USER_NAME="cloud_controller"			# PaaS-TA AP CCDB ID
UAA_DB_NAME="uaa"					# PaaS-TA AP UAADB Name
UAA_DB_USER_NAME="uaa"					# PaaS-TA AP UAADB ID
UAAC_PORTAL_CLIENT_ID="portalclient"			# UAAC Portal Client ID

IS_PAAS_TA_EXTERNAL_DB=false                            # (true or false)
PAAS_TA_EXTERNAL_DB_IP=                                 # PaaS-TA AP External DB IP
PAAS_TA_EXTERNAL_DB_PORT=                               # PaaS-TA AP External DB Port
PAAS_TA_EXTERNAL_DB_KIND=                               # PaaS-TA AP External DB Kind(IF USE e.g. postgres or mysql)
IS_PORTAL_EXTERNAL_DB=false                             # (true or false)
PORTAL_EXTERNAL_DB_IP=                                  # Portal External DB IP
PORTAL_EXTERNAL_DB_PORT=                                # Portal External DB Port
PORTAL_EXTERNAL_DB_PASSWORD=                            # Portal External DB Password
IS_PORTAL_EXTERNAL_STORAGE=false                        # (true or false)
PORTAL_EXTERNAL_STORAGE_IP=                             # Portal External Storage IP
PORTAL_EXTERNAL_STORAGE_PORT=                           # Portal External Storage Port
PORTAL_EXTERNAL_STORAGE_TENANTNAME=                     # Portal External Storage Tenant Name
PORTAL_EXTERNAL_STORAGE_USERNAME=                       # Portal External Storage Username
PORTAL_EXTERNAL_STORAGE_PASSWORD=                       # Portal External Storage Password
```
- Infra Variable
```
$ cd $HOME/sidecar-deployment/install-scripts/portal/infra
$ vi infra-variable.yml

#!/bin/bash

##PORTAL VARIABLE
NAMESPACE_NAME="paasta"                                 # Infra Namespace(Kubernetes) Name
PORTAL_NAME="PaaS-TA"                                   # Portal default api name
PORTAL_HEADER_AUTH="Basic YWRtaW46b3BlbnBhYXN0YQ=="     # Portal default header auth
PORTAL_DESC="PaaS-TA infra"                             # Portal default api descrition 

## MARIADB VARIABLE
MARIADB_PASSWORD=Paasta@2022                            # MariaDB Password
MARIADB_SERVICE_PORT=13306                              # MariaDB Service Port (e.g. 13306)
MARIADB_CONTAINER_PORT=13306                            # MariaDB Container Port (e.g. 13306)

## OPENSTACK SWIFT KETSTONE VARIABLE
KEYSTONE_SERVICE_PORT=25001                             # Keystone Service Port
KEYSTONE_CONTAINER_PORT=25001                           # Keystone Container Port
PROXY_SERVICE_PORT=10008                                # Proxy Service Port
PROXY_TARGET_PORT=10008                                 # Proxy Target Port
PORTAL_OBJECTSTORAGE_TENANTNAME=paasta-portal           # Binary storage tenantname
PORTAL_OBJECTSTORAGE_USERNAME=paasta-portal             # Binary storage username
PORTAL_OBJECTSTORAGE_PASSWORD=paasta                    # Binary storage password
```

## <div id='2.3'> 2.3. Infrastructure Deployment   
- Deploy the infrastructure to be used in the Sidecar Portal in the Kubernetes environment (omitted when deployed separately in the external environment) 
```
$ cd $HOME/sidecar-deployment/install-scripts/portal/infra
$ source deploy-portal-infra.sh
```

## <div id='2.4'> 2.4. Portal App Deployment   
- Deploy Portal App in the Sidecar environment
```
$ cd $HOME/sidecar-deployment/install-scripts/portal
$ source deploy-portal-app.sh

.....
.....

name                  requested state   processes           routes
portal-api            started           web:1/1, executable-jar:0/0, task:0/0   portal-api.apps.61.252.53.246.nip.io
portal-common-api     started           web:1/1, executable-jar:0/0, task:0/0   portal-common-api.apps.61.252.53.246.nip.io
portal-gateway        started           web:1/1, executable-jar:0/0, task:0/0   portal-gateway.apps.61.252.53.246.nip.io
portal-registration   started           web:1/1, executable-jar:0/0, task:0/0   portal-registration.apps.61.252.53.246.nip.io
portal-storage-api    started           web:1/1, executable-jar:0/0, task:0/0   portal-storage-api.apps.61.252.53.246.nip.io
portal-web-admin      started           web:1/1, executable-jar:0/0, task:0/0   portal-web-admin.apps.61.252.53.246.nip.io
portal-web-user       started           web:1/1                                 portal-web-user.apps.61.252.53.246.nip.io
```

When a reliable certificate is not applied to the Sidecar, the link to access the web user portal needs to be accessed by http.  
Refer to the link below for guides for operating and setting up the portal.  
[Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP Install]([../README.md](https://github.com/PaaS-TA/application-platform-guide-eng/blob/master/install/README.md)) > [Portal Container Type](https://github.com/PaaS-TA/application-platform-guide-eng/blob/master/install/portal/container_type.md) > [PaaS-TA AP Portal Operation](https://github.com/PaaS-TA/application-platform-guide-eng/blob/master/install/portal/container_type.md#4)

### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Sidecar