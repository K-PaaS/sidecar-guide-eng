### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Logging Service

## Table of Contents

1. [문서 개요](#1)  
   1.1. [목적](#1.1)  
   1.2. [범위](#1.2)  
   1.3. [참고자료](#1.3)

2. [Logging 서비스 설치](#2)  
   2.1. [Prerequisite](#2.1)  
   2.2. [Variable 설정](#2.2)  
   2.3. [Infra 배포](#2.3)  
   2.4. [Sidecar 설정 변경](#2.4)  
   2.5. [Portal Log API 배포](#2.5)

3. [Logging 서비스 관리](#3)  
   3.1. [Logging 서비스 활성화](#3.1)

<br>

# <div id='1'> 1. 문서 개요
## <div id='1.1'> 1.1. 목적
본 문서는 PaaS-TA Sidecar(이하 Sidecar) 환경에서 Logging 서비스를 사용하기 위한 가이드를 제공하는 데 목적이 있다.

<br>


## <div id='1.2'> 1.2. 범위
본 문서는 Logging 서비스를 검증하기 위한 기본 설치를 기준으로 작성하였다.

<br>


## <div id='1.3'> 1.3. 참고자료
cf-for-k8s github : [https://github.com/cloudfoundry/cf-for-k8s](https://github.com/cloudfoundry/cf-for-k8s)  
cf-for-k8s Document : [https://cf-for-k8s.io/docs/](https://cf-for-k8s.io/docs/)  
cf-k8s-logging github : [https://github.com/cloudfoundry/cf-k8s-logging](https://github.com/cloudfoundry/cf-k8s-logging)

<br><br>

# <div id='2'> 2. Logging 서비스 설치
## <div id='2.1'> 2.1. Prerequisite
본 설치 가이드는 Kubernetes 환경에 Logging Infra (InfluxDB)를 POD의 형태로 배포하며, Portal Log API는 Sidecar에 배포를 진행한다.  
**Sidecar 설치 시 Logging 서비스를 활성화**하고 싶다면, [2.2. Variable 설정](#2.2) 진행 후 Sidecar 설치를 진행한다.

<br>

## <div id='2.2'> 2.2 Variable 설정
POD의 형태로 배포되는 Infra와 Logging 서비스 변수를 설정한다.
- Logging Variable
> $ vi $HOME/sidecar-deployment/install-scripts/logging/logging-service-variable.yml
```shell script
#!/bin/bash

## LOGGING VARIABLE
ENABLE_LOGGING_SERVICE=true                         # (e.g. true or false)
LOGGING_OUTPUT_PLUGIN=influxdb                      # Logging output plugin (influxdb or http) --> recommend using influxdb plugin
FLUENTD_NAMESPACE=cf-system                         # Fluentd Namespace(Kubernetes) Name
```
- Infra Variable
> $ vi $HOME/sidecar-deployment/install-scripts/logging/infra/infra-variable.yml
```shell script
#!/bin/bash

## COMMON VARIABLE
LOGGING_NAMESPACE=logging                           # Logging Infra Namespace(Kubernetes) Name

## INFLUXDB VARIABLE
INFLUXDB_IP=influxdb.logging.svc.cluster.local      # InfluxDB IP
INFLUXDB_HTTP_PORT="8086"                           # InfluxDB Port
INFLUXDB_USERNAME=admin                             # InfluxDB Username
INFLUXDB_PASSWORD=PaaS-TA2020                       # InfluxDB Password
INFLUXDB_HTTPS_ENABLED=true                         # (e.g. true or false)
INFLUXDB_DATABASE=logging_db                        # InfluxDB DB Name
INFLUXDB_MEASUREMENT=logging_measurement            # InfluxDB Measurement Name
INFLUXDB_LIMIT=50                                   # InfluxDB query limit
INFLUXDB_RETENTION_POLICY=7d                        # InfluxDB retention policy
INFLUXDB_TIME_PRECISION=s                           # Level of timestamp stored (hour(h), minutes(m), second(s), millisecond(ms), microsecond(u), nanosecond(ns)
```

<br>

## <div id='2.3'> 2.3. Infra 배포
- Kubernetes 환경에 Sidecar Logging 에서 사용 될 Infra를 배포한다.
```shell script
$ cd $HOME/sidecar-deployment/install-scripts/logging/infra
$ source deploy-logging-infra.sh
```

- Infra가 정상적으로 배포되었는지 확인한다.
> $ kubectl get statefulset,pods -n logging
```
NAME                        READY   AGE
statefulset.apps/influxdb   1/1     1m
NAME             READY   STATUS    RESTARTS   AGE
pod/influxdb-0   1/1     Running   0          1m
```

<br>

## <div id='2.4'> 2.4. Sidecar 설정 변경
Logging 서비스를 활성화 하기 위해서는 Sidecar 설정 중 일부를 변경해야 한다.
- `enable-logging-service.sh` 파일을 실행하여 Sidecar 설정을 변경한다.
```shell script
$ cd $HOME/sidecar-deployment/install-scripts/logging
$ source enable-logging-service.sh
```

- Sidecar 설정이 정상적으로 변경되었는지 확인한다.
> $ kubectl get configmap fluentd-config-ver-1 -n cf-system -o yaml
```diff
apiVersion: v1
data:
  aggregate_drains.conf: ""
  fluentd.conf: |

...

+    # seperate process for logging-service
+    <match **>
+      @type copy
+      <store>
+        @type relabel
+        @label @SIDECAR
+      </store>
+      <store>
+        @type relabel
+        @label @LOGGING
+      </store>
+    </match>
+
+    # for sidecar
+    <label @SIDECAR>
       <match **>
         @type copy
         <store ignore_error>
           @type syslog_rfc5424
           host log-cache-syslog
           port 8082
           transport tcp
           <format>
             @type syslog_rfc5424
             proc_id_field instance_id
             app_name_field app_id
             structured_data_field structured_data
           </format>

           <buffer>
             @type memory
             flush_mode immediate
             flush_thread_count 8
           </buffer>
         </store>
         @include /fluentd/etc/aggregate_drains.conf
       </match>
+    </label>
+
+    # for logging-service
+    <label @LOGGING>
+      <filter **>
+        @type grep
+        <exclude>
+          key $.instance_id
+          pattern /^0$/
+        </exclude>
+      </filter>
+
+      <filter kubernetes.**>
+        @type record_transformer
+        remove_keys stream,docker,kubernetes
+      </filter>
+
+      <filter forwarded.**>
+        @type record_transformer
+        remove_keys source_type
+      </filter>
+
+      <filter kubernetes.** forwarded.**>
+        @type record_transformer
+        enable_ruby true
+        <record>
+          id ${record.dig("app_id")}
+          message \{\"cf_app_id\":\"${record.dig("app_id")}\"\,\"msg\":\"${record.dig("log")}\"\,\"instance_id\"\:\"${record.dig("instance_id")}\"\}
+        </record>
+      </filter>
+
+      <filter kubernetes.** forwarded.**>
+        @type record_transformer
+        remove_keys app_id,instance_id,log,structured_data
+      </filter>
+
+      <match **>
+        @type influxdb
+        host influxdb.logging.svc.cluster.local
+        port 8086
+        user admin
+        password PaaS-TA2020
+        dbname logging_db
+        measurement logging_measurement
+        time_precision s
+        tag_keys ["id"]
+        time_key time
+        flush_interval 60
+        use_ssl true
+        verify_ssl false
+        sequence_tag _seq
+      </match>
+    </label>

...

```

> $ kubectl get daemonset,pods -n cf-system
```
NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/fluentd   5         5         5       5            5           <none>          47h
NAME                                                      READY   STATUS      RESTARTS   AGE
pod/ccdb-migrate-cf4pf                                    0/2     Completed   0          47h
pod/cf-api-clock-589587f877-mtvbb                         2/2     Running     0          47h
pod/cf-api-controllers-6f9b86449d-sws96                   3/3     Running     1          47h
pod/cf-api-deployment-updater-5948c9df7f-ntbxj            2/2     Running     1          47h
pod/cf-api-server-544485c45b-2xvqn                        6/6     Running     2          47h
pod/cf-api-worker-8c6cd69c7-npgp7                         3/3     Running     0          47h
pod/eirini-api-6f78d469cf-pf2xp                           2/2     Running     0          47h
pod/eirini-app-migration-fgxq6                            0/1     Completed   0          47h
pod/eirini-event-reporter-779958dc9d-5g4fv                2/2     Running     6          47h
pod/eirini-event-reporter-779958dc9d-jpl6v                2/2     Running     5          47h
pod/eirini-instance-index-env-injector-556b856f89-vnl9d   1/1     Running     0          47h
pod/eirini-task-reporter-5f7b5766cf-725hc                 2/2     Running     5          47h
pod/eirini-task-reporter-5f7b5766cf-x6fnq                 2/2     Running     2          47h
pod/fluentd-l87rf                                         2/2     Running     0          25h
pod/fluentd-mfkpx                                         2/2     Running     0          25h
pod/fluentd-nlf8l                                         2/2     Running     1          25h
pod/fluentd-rwqf6                                         2/2     Running     1          25h
pod/fluentd-vbmms                                         2/2     Running     0          25h
pod/log-cache-backend-5f795cbdcd-f2msz                    3/3     Running     0          47h
pod/log-cache-frontend-764dd79ccc-jxzv7                   3/3     Running     0          47h
pod/metric-proxy-b5c86f55f-rgff4                          2/2     Running     0          47h
pod/routecontroller-66458dfdbb-pztvw                      2/2     Running     6          47h
pod/uaa-6fc9cf8bcb-9mq7r                                  3/3     Running     0          41h
```

<br>

## <div id='2.5'> 2.5. Portal Log API 배포
- Manifest 파일을 수정한다.
> $ vi $HOME/sidecar-deployment/install-scripts/portal/portal-app/portal-app-1.2.13/portal-log-api-2.3.2/manifest.yml
```yaml
applications:
- name: portal-log-api
  memory: 1G
  instances: 1
  buildpacks:
  - java_buildpack
  path: paas-ta-portal-log-api.jar
  env:

    ...

    ### logging info (InfluxDB)
    influxdb_ip: influxdb.logging.svc.cluster.local
    influxdb_url: https://influxdb.logging.svc.cluster.local:8086
    influxdb_username: admin
    influxdb_password: PaaS-TA2020
    influxdb_database: logging_db
    influxdb_measurement: logging_measurement
    influxdb_limit: 50
    influxdb_httpsEnabled: true
```

- Sidecar 환경에 Portal Log API를 배포한다.
```
$ cd $HOME/sidecar-deployment/install-scripts/portal/portal-app/portal-app-1.2.13/portal-log-api-2.3.2
$ cf push -b paketo-buildpacks/java
Pushing app portal-log-api to org portal / space system as admin...
Applying manifest file /home/ubuntu/sidecar-deployment/install-scripts/portal/portal-app/portal-app-1.2.13/portal-log-api-2.3.2/manifest.yml...
...
Waiting for app portal-log-api to start...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
Instances starting...
name:                portal-log-api
requested state:     started
isolation segment:   placeholder
routes:              portal-log-api.apps.61.252.53.246.nip.io
last uploaded:       Wed 28 Jun 06:59:35 UTC 2023
stack:               
buildpacks:          
isolation segment:   placeholder
type:            web
sidecars:        
instances:       1/1
memory usage:    1024M
start command:   java org.springframework.boot.loader.JarLauncher
     state     since                  cpu    memory   disk     details
#0   running   2023-06-28T07:01:55Z   0.0%   0 of 0   0 of 0   
type:            executable-jar
sidecars:        
instances:       0/0
memory usage:    1024M
start command:   java org.springframework.boot.loader.JarLauncher
There are no running instances of this process.
type:            task
sidecars:        
instances:       0/0
memory usage:    1024M
start command:   java org.springframework.boot.loader.JarLauncher
There are no running instances of this process.
```

<br><br>

# <div id='3'> 3. Logging 서비스 관리
## <div id='3.1'> 3.1. Logging 서비스 활성화
PaaS-TA 포탈에서 서비스를 사용하기 위해 Logging 서비스 활성화 코드 등록을 해 주어야 한다.

-	PaaS-TA 운영자 포탈에 접속한다.  
     ![001]

-	운영관리의 코드관리 메뉴로 이동하여 다음과 같이 코드를 등록한다.

> ※ Group Table  
> 코드 ID  : LOGGING  
> 코드 이름 : Logging Service  
> ![002]
>
> ※ Detail Table  
> Key : enable_logging_service  
> Value : true  
> 요약 : Logging Service Enable Code  
> 사용 : Y  
> ![003]
![004]

[001]:./images/service/logging-service/image001.png
[002]:./images/service/logging-service/image002.png
[003]:./images/service/logging-service/image003.png
[004]:./images/service/logging-service/image004.png

<br>


### [Index](https://github.com/PaaS-TA/Guide/blob/master/README.md) > [PaaS-TA Sidecar Install](./README.md) > Logging Service