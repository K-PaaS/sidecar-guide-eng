### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Sidecar - local

## Table of Contents

1. [문서 개요](#1)  
  1.1. [목적](#1.1)  
  1.2. [범위](#1.2)  
  1.3. [참고자료](#1.3)  

2. [PaaS-TA Sidecar - local 설치](#2)  
  2.1. [Prerequisite](#2.1)  
  2.2. [설치 파일 다운로드](#2.2)  
  2.3. [실행 파일 소개 및 설치](#2.3)  
  2.4. [Local Kubernetes Cluster 구성](#2.4)  
  　2.4.1 [kind](#2.4.1)  
  　2.4.2 [minikube](#2.4.2)  
  2.5. [Sidecar 설치](#2.5)  
  　2.5.1 [kind](#2.5.1)  
  　2.5.2 [minikube](#2.5.2)  

# <div id='1'> 1. 문서 개요
## <div id='1.1'> 1.1. 목적
본 문서는 Local Kubenetes Cluster를 구성하고 해당 환경에서 PaaS-TA Sidecar(이하 Sidecar)를 설치하기 위한 가이드를 제공하는 데 목적이 있다.

<br>

## <div id='1.2'> 1.2. 범위
본 문서는 [cf-for-k8s v5.4.1](https://github.com/cloudfoundry/cf-for-k8s/tree/v5.4.1)을 기준으로 작성하였다.  
본 문서는 [kind](https://kind.sigs.k8s.io/) 혹은 [minikube](https://minikube.sigs.k8s.io/docs/)로 Local Kubernetes Cluster를 구성 후 Sidecar 설치 기준으로 작성하였다.

<br>

## <div id='1.3'> 1.3. 참고자료
cf-for-k8s github : [https://github.com/cloudfoundry/cf-for-k8s](https://github.com/cloudfoundry/cf-for-k8s)  
cf-for-k8s Document : [https://cf-for-k8s.io/docs/](https://cf-for-k8s.io/docs/)  
kind Document :  [https://kind.sigs.k8s.io/](https://kind.sigs.k8s.io/)  
minikube Document : [https://minikube.sigs.k8s.io/docs/](https://minikube.sigs.k8s.io/docs/)  

<br>

# <div id='2'> 2. PaaS-TA Sidecar - local 설치
## <div id='2.1'> 2.1. Prerequisite
cf-for-k8s 공식 문서에서는 Local Kubernetes Cluster 요구 조건을 다음과 같이 권고하고 있다.
- 최소 4 CPU, 6GB Memory
- 권장 6-8 CPU, 8-16GB Memory
- OCI 호환 레지스트리 제공 (e.g. [Docker Hub](https://hub.docker.com/), [Google container registry](https://cloud.google.com/container-registry),  [Azure container registry](https://hub.docker.com/), [Harbor](https://goharbor.io/), etc....)  
  본 가이드는 Docker Hub 기준으로 가이드가 진행된다. (계정가입 필요)

<br>

## <div id='2.2'> 2.2. 설치 파일 다운로드

- git clone 명령을 통해 다음 경로에서 Sidecar 다운로드를 진행한다. 본 설치 가이드에서의 Sidecar의 버전은 베타 버전이다.
```
$ cd $HOME
$ git clone https://github.com/PaaS-TA/sidecar-deployment.git -b beta
$ cd sidecar-deployment
```

<br>

## <div id='2.3'> 2.3. 실행 파일 소개 및 설치

- Sidecar를 설치 & 활용하기 위해선 다음과 같은 실행파일이 필요하다.

| 이름   |      설명      |
|----------|-------------|
| [ytt](https://carvel.dev/ytt/) | Sidecar을 배포 시 사용 되는 YAML을 생성하는 툴 |
| [kapp](https://carvel.dev/kapp/) | Sidecar의 라이프사이클을 관리하는 툴 |
| [kubectl](https://github.com/kubernetes/kubectl) | Kubernetes Cluster를 제어하는 툴 |
| [bosh cli](https://github.com/cloudfoundry/bosh-cli) | Sidecar에서 사용될 임의의 비밀번호와 certificate를 생성하는 툴 |
| [cf cli](https://github.com/cloudfoundry/cli) (v7+) | Sidecar와 상호 작용하는 툴 |
| [docker](https://www.docker.com/) | 컨테이너 기반의 가상화 플랫폼 |

- ytt, kapp, bosh cli, cf cli 설치
```
$ source install-scripts/utils-install.sh
```

- docker 설치
```
$ sudo wget -qO- http://get.docker.com/ | sh
$ sudo chmod 666 /var/run/docker.sock 
$ docker -v
Docker version 20.10.9, build c2ea9bc
```

- kubectl 설치
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

## <div id='2.4'> 2.4. Local Kubernetes Cluster 구성
본 가이드에서 제공되는 cluster 구성 도구 kind와 minikube를 선택하여 진행한다.  
### <div id='2.4.1'> 2.4.1. kind

- kind 다운로드
```
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.10.0/kind-linux-amd64
$ chmod +x ./kind
$ sudo mv ./kind /usr/local/bin/kind
$ kind --version 
kind version 0.10.0
```

- cluster 생성
```
$ kind create cluster --config=./deploy/kind/cluster.yml --image kindest/node:v1.20.2
$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:43173
KubeDNS is running at https://127.0.0.1:43173/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

<br>

### <div id='2.4.2'> 2.4.2. minikube

- minikube 다운로드
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
$ minikube version
minikube version: v1.23.2
```

- cluster 생성
```
$ minikube start --cpus=6 --memory=8g --kubernetes-version="1.20.2" --driver=docker
  
$ kubectl cluster-info --context minikube
Kubernetes control plane is running at https://192.168.49.2:8443
KubeDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

```

<br>

## <div id='2.5'> 2.5. Sidecar 설치
### <div id='2.5.1'> 2.5.1. kind

- Sidecar에서 사용할 변수(비밀번호, 인증키 등)를 생성한다.
```
$ mkdir ./tmp
$ ./hack/generate-values.sh -d vcap.me > ./tmp/sidecar-values.yml


# cat << 부터 EOF 마지막까지 한번에 실행 (app_registry 정보 변경 필요)
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

- 변수 설정파일은 tmp/sidecar-values.yml에 생성된다.
```
# 설정파일 : ./tmp/sidecar-values.yml
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

- Sidecar 배포 YAML를 생성한다.
```
$ ytt -f ./config -f "tmp/sidecar-values.yml" > "tmp/sidecar-rendered.yml"
```

- Sidecar 배포 YAML은 tmp/sidecar-rendered.yml에 생성된다.
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

- 생성된 YAML파일을 이용하여 Sidecar를 설치한다.
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

- Sidecar가 정상설치 되었는지 샘플앱을 통해 확인한다.

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

- (참고) kind cluster 삭제
```
$ kind delete cluster
```

<br>
  


### <div id='2.5.2'> 2.5.2. minikube
- Metrics-server를 활성화 한다.
```
$ minikube addons enable metrics-server
```

- LoadBalancer Service 사용을 위한 Minikube 터널링을 한다.
```
# 터널링 백그라운드 실행
$ minikube tunnel &>/dev/null &
```


- Sidecar에서 사용할 변수(비밀번호, 인증키 등)를 생성한다.
```
$ mkdir ./tmp
$ ./hack/generate-values.sh -d $(minikube ip).nip.io > ./tmp/sidecar-values.yml


# cat << 부터 EOF 마지막까지 한번에 실행 (app_registry 정보 변경 필요)
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

- 변수 설정파일은 tmp/sidecar-values.yml에 생성된다.
```
# 설정파일 : ./tmp/sidecar-values.yml
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

- Sidecar 배포 YAML를 생성한다.
```
$ ytt -f ./config -f "tmp/sidecar-values.yml" > "tmp/sidecar-rendered.yml"
```

- Sidecar 배포 YAML은 tmp/sidecar-rendered.yml에 생성된다.
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

- 생성된 YAML파일을 이용하여 sidecar를 설치한다.
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

- Sidecar가 정상설치 되었는지 샘플앱을 통해 확인한다.

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
  
  
- (참고) Minikube cluster 삭제
```
# minikube tunnel process 종료
$ kill -9 $(ps -ef | grep "minikube tunnel" | awk '{print $2}' | head -n 1)
  
# Minikube cluster 삭제
$ minikube delete
```

<br>

### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Sidecar - local
