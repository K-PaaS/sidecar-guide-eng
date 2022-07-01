### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Sidecar - local

## Table of Contents

1. [Document Outline](#1)  
  1.1. [Purpose](#1.1)  
  1.2. [Range](#1.2)  
  1.3. [References](#1.3)  

2. [PaaS-TA Sidecar - local Installation](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [Download Installation File](#2.2)  
  2.3. [Introduction and Installation of Executable Files](#2.3)  
  2.4. [Local Kubernetes Cluster Configuration](#2.4)  
  　2.4.1 [kind](#2.4.1)  
  　2.4.2 [minikube](#2.4.2)  
  2.5. [Sidecar Installation](#2.5)  
  　2.5.1 [kind](#2.5.1)  
  　2.5.2 [minikube](#2.5.2)  

# <div id='1'> 1. Document Outline
## <div id='1.1'> 1.1. Purpose
The purpose of this document is to provide a guide for configuring a Local Kubernetes Cluster and installing PaaS-TA Sidecar (hereinafter referred to as Sidecar) to a specified environment.

<br>

## <div id='1.2'> 1.2. Range
This document was written based on [cf-for-k8s v5.4.1](https://github.com/cloudfoundry/cf-for-k8s/tree/v5.4.1).  
This document is based on the installation of Sidecar after configuring the Local Kubernetes Cluster with [kind](https://kind.sigs.k8s.io/) or [minikube](https://minikube.sigs.k8s.io/docs/).

<br>

## <div id='1.3'> 1.3. References
cf-for-k8s github : [https://github.com/cloudfoundry/cf-for-k8s](https://github.com/cloudfoundry/cf-for-k8s)  
cf-for-k8s Document : [https://cf-for-k8s.io/docs/](https://cf-for-k8s.io/docs/)  
kind Document :  [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)  
minikube Document : [https://minikube.sigs.k8s.io/docs/](https://minikube.sigs.k8s.io/docs/)  

<br>

# <div id='2'> 2. PaaS-TA Sidecar - local Installation
## <div id='2.1'> 2.1. Prerequisite
The cf-for-k8s official document recommends the Local Kubernetes Cluster requirements as follows.
- At least 4 CPU, 6GB Memory
- Recommend 6-8 CPU, 8-16GB Memory
- Provides OCI-compliant registry (e.g. [Docker Hub](https://hub.docker.com/), [Google container registry](https://cloud.google.com/container-registry),  [Azure container registry](https://hub.docker.com/), [Harbor](https://goharbor.io/), etc....)  
  This guide is based on the Docker Hub. (Account registration required)

<br>

## <div id='2.2'> 2.2. Download Installation File

- Use the git clone command to download Sidecar from the following path. The version of Sidecar in this installation guide is beta.
```
$ cd $HOME
$ git clone https://github.com/PaaS-TA/sidecar-deployment.git -b beta
$ cd sidecar-deployment
```

<br>

## <div id='2.3'> 2.3. Introduction and Installation of Executable Files

- The following executable files are required to install and utilize Sidecar.

| Name   |      Description      |
|----------|-------------|
| [ytt](https://carvel.dev/ytt/) | Tools to create YAMLs used when deploying Sidecar |
| [kapp](https://carvel.dev/kapp/) | Tools to manage the lifecycle of Sidecar |
| [kubectl](https://github.com/kubernetes/kubectl) | Tool to control Kubernetes Cluster |
| [bosh cli](https://github.com/cloudfoundry/bosh-cli) | Tools for generating arbitrary passwords and certificates to be used by Sidecar |
| [cf cli](https://github.com/cloudfoundry/cli) (v7+) | Tools that interact with Sidecar |
| [docker](https://www.docker.com/) | Container-based virtualization platform |

- ytt, kapp, bosh cli, cf cli installation
```
$ source install-scripts/utils-install.sh
```

- docker installation
```
$ sudo wget -qO- http://get.docker.com/ | sh
$ sudo chmod 666 /var/run/docker.sock 
$ docker -v
Docker version 20.10.9, build c2ea9bc
```

- kubectl installation
```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"22", GitVersion:"v1.22.2", GitCommit:"8b5a19147530eaac9476b0ab82980b4088bbc1b2", GitTreeState:"clean", BuildDate:"2021-09-15T21:38:50Z", GoVersion:"go1.16.8", Compiler:"gc", Platform:"linux/amd64"}
```

<br>

## <div id='2.4'> 2.4. Local Kubernetes Cluster Configuration
Proceed by selecting the cluster configuring tool which is kind and minikube provided from this guide.  
### <div id='2.4.1'> 2.4.1. kind

- kind Download
```
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
$ kind --version 
kind version 0.10.0
```

- Create cluster
```
$ kind create cluster --config=./deploy/kind/cluster.yml --image kindest/node:v1.20.2
$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:43173
KubeDNS is running at https://127.0.0.1:43173/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

<br>

### <div id='2.4.2'> 2.4.2. minikube

- minikube Download
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
$ minikube version
minikube version: v1.23.2
```

- Create cluster
```
$ minikube start --cpus=6 --memory=8g --kubernetes-version="1.20.2" --driver=docker
  
$ kubectl cluster-info --context minikube
Kubernetes control plane is running at https://192.168.49.2:8443
KubeDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

```

<br>

## <div id='2.5'> 2.5. Sidecar Installation
### <div id='2.5.1'> 2.5.1. kind

- Generate variables (password, authentication key, etc.) to be used by Sidecar.
```
$ mkdir ./tmp
$ ./hack/generate-values.sh -d vcap.me > ./tmp/sidecar-values.yml


# Run from cat << to EOF last at once (app_registry information needs to be changed)
########################################################
$ cat << EOF >> ./tmp/sidecar-values.yml
app_registry:
  hostname: https://index.docker.io/v1/
  repository_prefix: "<dockerhub_username>"
  username: "<dockerhub_username>"
  password: "<dockerhub_password>"

add_metrics_server_components: true
enable_automount_service_account_token: true
load_balancer:
  enable: false
metrics_server_prefer_internal_kubelet_address: true
remove_resource_requirements: true
use_first_party_jwt_tokens: true
EOF
########################################################
```

- The variable setting file is created in tmp/sidecar-values.yml.
```
# File Set : ./tmp/sidecar-values.yml
$ vi ./tmp/sidecar-values.yml

#@data/values
---
system_domain: "vcap.me"
app_domains:
#@overlay/append
- "apps.vcap.me"
cf_admin_password: eukmm33ja03asdfvlnv4

blobstore:
  secret_access_key: 4itoiu40asdf0xisylq

cf_db:
  admin_password: jkb2xjel2kasdfdrgj4
......
......
app_registry:
  hostname: https://index.docker.io/v1/
  repository_prefix: "dockerhub_username"
  username: "dockerhub_username"
  password: "dockerhub_password"

add_metrics_server_components: true
enable_automount_service_account_token: true
load_balancer:
  enable: false
metrics_server_prefer_internal_kubelet_address: true
remove_resource_requirements: true
use_first_party_jwt_tokens: true
```

- Create a Sidecar deployment YAML.
```
$ ytt -f ./config -f "tmp/sidecar-values.yml" > "tmp/sidecar-rendered.yml"
```

- The Sidecar deployment YAML is created in tmp/sidecar-rendered.yml.
```
$ vi tmp/sidecar-rendered.yml

apiVersion: kapp.k14s.io/v1alpha1
kind: Config
metadata:
  name: kapp-version
minimumRequiredVersion: 0.33.0
---
apiVersion: kapp.k14s.io/v1alpha1
kind: Config
......

```

- Install Sidecar by using the created YAML file.
```
$ kapp deploy -a sidecar -f tmp/sidecar-rendered.yml -y

......
1:56:06AM: ongoing: reconcile job/restart-workloads-for-istio1-8-4 (batch/v1) namespace: cf-workloads
1:56:06AM:  ^ Waiting to complete (1 active, 0 failed, 0 succeeded)
1:56:06AM:  L ok: waiting on pod/restart-workloads-for-istio1-8-4-4mhd7 (v1) namespace: cf-workloads
1:56:23AM: ok: reconcile job/restart-workloads-for-istio1-8-4 (batch/v1) namespace: cf-workloads
1:56:23AM:  ^ Completed
1:56:23AM: ---- applying complete [305/305 done] ----
1:56:23AM: ---- waiting complete [305/305 done] ----

Succeeded
```

- Check if Sidecar is installed normally through the sample app.

```
$ cf login -a api.$(grep system_domain ./tmp/sidecar-values.yml | cut -d" " -f2 | sed -e 's/\"//g') --skip-ssl-validation -u admin -p "$(grep cf_admin_password ./tmp/sidecar-values.yml | cut -d" " -f2)"

$ cf create-org test-org
$ cf create-space test-space -o test-org
$ cf target -o test-org -s test-space

$ cf push -p ./tests/smoke/assets/test-node-app test-node-app
Pushing app test-node-app to org system / space test-space as admin...
Packaging files to upload...
Uploading files...
 558 B / 558 B [============================================================] 100.00% 1s

Waiting for API to complete processing files...
.......
.......
Build successful

Waiting for app test-node-app to start...

Instances starting...
Instances starting...

name:                test-node-app
requested state:     started
isolation segment:   placeholder
routes:              test-node-app.apps.system.domain
last uploaded:       Thu 30 Sep 07:04:54 UTC 2021
stack:               
buildpacks:          
isolation segment:   placeholder

type:            web
sidecars:        
instances:       1/1
memory usage:    1024M
start command:   node server.js
     state     since                  cpu    memory   disk     details
#0   running   2021-09-30T07:06:01Z   0.0%   0 of 0   0 of 0   


$ curl https://test-node-app.apps.$(grep system_domain ./tmp/sidecar-values.yml | cut -d" " -f2 | sed -e 's/\"//g') -k
Hello World
```

- (Refer) kind cluster Deletion
```
$ kind delete cluster
```

<br>
  


### <div id='2.5.2'> 2.5.2. minikube
- Enable Metrics-server.
```
$ minikube addons enable metrics-server
```

- Minikube tunneling to use LoadBalancer service.
```
# Running the Tunneling Background
$ minikube tunnel &>/dev/null &
```


- Generate variables (password, authentication key, etc.) to be used by Sidecar.
```
$ mkdir ./tmp
$ ./hack/generate-values.sh -d $(minikube ip).nip.io > ./tmp/sidecar-values.yml


# Run from cat << to EOF last at once (app_registry information needs to be changed)
########################################################
$ cat << EOF >> ./tmp/sidecar-values.yml
app_registry:
  hostname: https://index.docker.io/v1/
  repository_prefix: "<dockerhub_username>"
  username: "<dockerhub_username>"
  password: "<dockerhub_password>"

enable_automount_service_account_token: true
remove_resource_requirements: true
use_first_party_jwt_tokens: true
EOF
########################################################
```

- The variable setting file is created in tmp/sidecar-values.yml.
```
# Setting file : ./tmp/sidecar-values.yml
$ vi ./tmp/sidecar-values.yml

#@data/values
---
system_domain: "172.18.0.1.nip.io"
app_domains:
#@overlay/append
- "apps.172.18.0.1.nip.io"
cf_admin_password: eukmm33ja03asdfvlnv4

blobstore:
  secret_access_key: 4itoiu40asdf0xisylq

cf_db:
  admin_password: jkb2xjel2kasdfdrgj4
......
......
app_registry:
  hostname: https://index.docker.io/v1/
  repository_prefix: "dockerhub_username"
  username: "dockerhub_username"
  password: "dockerhub_password"

enable_automount_service_account_token: true
remove_resource_requirements: true
use_first_party_jwt_tokens: true
```

- Create Sidecar Deployment YAML.
```
$ ytt -f ./config -f "tmp/sidecar-values.yml" > "tmp/sidecar-rendered.yml"
```

- Sidecar deployment YAML is created in tmp/sidecar-rendered.yml.
```
$ vi tmp/sidecar-rendered.yml

apiVersion: kapp.k14s.io/v1alpha1
kind: Config
metadata:
  name: kapp-version
minimumRequiredVersion: 0.33.0
---
apiVersion: kapp.k14s.io/v1alpha1
kind: Config
......

```

- Install the sidecar using the generated YAML file.
```
$ kapp deploy -a sidecar -f tmp/sidecar-rendered.yml -y

........
2:40:51AM:  L ok: waiting on pod/restart-workloads-for-istio1-8-4-lllwn (v1) namespace: cf-workloads
2:41:18AM: ok: reconcile job/restart-workloads-for-istio1-8-4 (batch/v1) namespace: cf-workloads
2:41:18AM:  ^ Completed
2:41:18AM: ---- applying complete [296/296 done] ----
2:41:18AM: ---- waiting complete [296/296 done] ----

Succeeded
```

- Check if Sidecar is installed normally through the sample app.

```
$ cf login -a api.$(grep system_domain ./tmp/sidecar-values.yml | cut -d" " -f2 | sed -e 's/\"//g') --skip-ssl-validation -u admin -p "$(grep cf_admin_password ./tmp/sidecar-values.yml | cut -d" " -f2)"

$ cf create-org test-org
$ cf create-space test-space -o test-org
$ cf target -o test-org -s test-space

$ cf push -p ./tests/smoke/assets/test-node-app test-node-app
Pushing app test-node-app to org system / space test-space as admin...
Packaging files to upload...
Uploading files...
 558 B / 558 B [============================================================] 100.00% 1s

Waiting for API to complete processing files...
.......
.......
Build successful

Waiting for app test-node-app to start...

Instances starting...
Instances starting...

name:                test-node-app
requested state:     started
isolation segment:   placeholder
routes:              test-node-app.apps.system.domain
last uploaded:       Thu 30 Sep 07:04:54 UTC 2021
stack:               
buildpacks:          
isolation segment:   placeholder

type:            web
sidecars:        
instances:       1/1
memory usage:    1024M
start command:   node server.js
     state     since                  cpu    memory   disk     details
#0   running   2021-10-05T02:00:08Z   0.0%   0 of 0   0 of 0   


$ curl https://test-node-app.apps.$(grep system_domain ./tmp/sidecar-values.yml | cut -d" " -f2 | sed -e 's/\"//g') -k
Hello World
```
  
  
- (Refer) Minikube cluster Deletion
```
# minikube tunnel process terminate
$ kill -9 $(ps -ef | grep "minikube tunnel" | awk '{print $2}' | head -n 1)
  
# Minikube cluster deletion
$ minikube delete
```

<br>

### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Sidecar - local
