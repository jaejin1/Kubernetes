## 쿠버네티스 클라이언트

공식적인 쿠버네티스 클라이언트는 쿠버네티스 API와 상호작용하기 위한 명령줄 도구인 kubectl이다. kubectl은 포드(pod), 레플리카세트(ReplicaSet), 서비스(Service) 같은 대부분의 쿠버네티스 객체를 관리하는데 사용하며, 클러스터의 전반적인 상태를 탐색하고 확인하는데 사용한다.

### 클러스터 상태 확인

클러스터 버전을 다음과 같이 사용한다.

> $ kubectl version

~~~
ubuntu@ip-172-31-41-207:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.7", GitCommit:"b30876a5539f09684ff9fde266fda10b37738c9c", GitTreeState:"clean", BuildDate:"2018-01-16T21:59:57Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.6", GitCommit:"9f8ebd171479bec0ada837d7ee641dec2f8c6dd1", GitTreeState:"clean", BuildDate:"2018-03-21T15:13:31Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
~~~

서버와 클라이언트 두 버전이 다른 것에 대해 걱정 안해도된다.

클러스터의 상태가 양호한지 확인한다.

> $ kubectl get componentstatuses

~~~
ubuntu@ip-172-31-41-207:~$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
~~~

출력 결과 쿠버네티스 클러스터의 구성요소 확인 가능.
컨트롤러 관리자(ontroller-manager)는 클러스터의 동작을 제어하는 다양한 컨트롤러를 실행하는 역할을 담당한다.
스케줄러(scheduler)는 클러스터의 다른 노드에 다른 포트를 배치한다.

### 쿠버네티스 워커 노드 목록 조회

다음 명령으로 클러스터의 모든 노드 목록을 조회 가능

> $ kuberctl get nodes

~~~
ubuntu@ip-172-31-41-207:~$ kubectl get nodes
NAME                                             STATUS    ROLES     AGE       VERSION
ip-10-10-32-56.ap-northeast-2.compute.internal   Ready     node      33m       v1.9.6
ip-10-10-61-15.ap-northeast-2.compute.internal   Ready     master    34m       v1.9.6
ip-10-10-93-73.ap-northeast-2.compute.internal   Ready     node      33m       v1.9.6
~~~

쿠버네티스 노드는 클러스터를 관리하는 API 서버, 스케줄러 같은 컨테이너를 포함하는 마스터(master) 노드와 컨테이너가 실행되는 워커(worker) 노드로 구분된다. 쿠버네티스는 일반적으로 사용자의 작업 부하(workload)가 클러스터의 전체 운영에 악영향을 주지 않도록 마스터 노드의 작업을 스케줄링 하지않음.

kubectl describe 명령을 사용해 특정 노드의 상세 정보 확인가능하다.

> $ kubectl describe nodes {node_name}

먼저 노드에 대한 기본 정보가 표시된다.

~~~
ubuntu@ip-172-31-41-207:~$ kubectl describe nodes ip-10-10-32-56.ap-northeast-2.compute.internal
Name:               ip-10-10-32-56.ap-northeast-2.compute.internal
Roles:              node
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=m4.xlarge
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/region=ap-northeast-2
                    failure-domain.beta.kubernetes.io/zone=ap-northeast-2a
                    kops.k8s.io/instancegroup=nodes
                    kubernetes.io/hostname=ip-10-10-32-56.ap-northeast-2.compute.internal
                    kubernetes.io/role=node
                    node-role.kubernetes.io/node=
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             <none>
CreationTimestamp:  Tue, 10 Jul 2018 04:52:42 +0000
~~~

해당 노드가 리눅스 운영체제 에서 실행중이고 amd64 프로세서에서 실행중임을 알 수 있다.

다음으로 노드 자체 운영 정보를 확인할 수 있다.

~~~
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Tue, 10 Jul 2018 04:52:48 +0000   Tue, 10 Jul 2018 04:52:48 +0000   RouteCreated                 RouteController created a route
  OutOfDisk            False   Tue, 10 Jul 2018 05:30:45 +0000   Tue, 10 Jul 2018 04:52:42 +0000   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure       False   Tue, 10 Jul 2018 05:30:45 +0000   Tue, 10 Jul 2018 04:52:42 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Tue, 10 Jul 2018 05:30:45 +0000   Tue, 10 Jul 2018 04:52:42 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  Ready                True    Tue, 10 Jul 2018 05:30:45 +0000   Tue, 10 Jul 2018 04:53:02 +0000   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:   ...
  ExternalIP:   ...
  InternalDNS:  ...
  ExternalDNS:  ...
  Hostname:     ...
~~~

충분한 상태의 디스크와 메모리 공간을 확보하고 있으며 쿠버네티스 마스터에게 정상 상태를 보고한다.

다음은 머신의 용량 정보이다.

~~~
Capacity:
 cpu:     4
 memory:  16435604Ki
 pods:    110
Allocatable:
 cpu:     4
 memory:  16333204Ki
 pods:    110
~~~

다음은 실행 중인 도커의 버전 정보, 쿠버네티스와 리눅스 커널 버전 정보등 노드의 소프트웨어에 대한 정보이다.

~~~
System Info:
 Machine ID:                 65f0096b4a6b41259f052aed60b248c6
 System UUID:                EC29E7D2-9065-8629-C9B9-5C49C848F342
 Boot ID:                    a2a105d6-fbf7-4431-8e46-4ae2bbace576
 Kernel Version:             4.4.121-k8s
 OS Image:                   Debian GNU/Linux 8 (jessie)
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://17.3.2
 Kubelet Version:            v1.9.6
 Kube-Proxy Version:         v1.9.6
PodCIDR:                     100.96.1.0/24
ExternalID:                  i-05149d5ae0d49cf0b
~~~

마지막으로 이 노드에서 현재 실행 중인 포드의 정보를 확인할 수 있다.

~~~
Non-terminated Pods:         (7 in total)
  Namespace                  Name                                                         CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                                                         ------------  ----------  ---------------  -------------
  default                    sample-node-b77d6dbd9-5mxm9                                  250m (6%)     500m (12%)  256Mi (1%)       512Mi (3%)
  default                    sample-web-5f778985dc-7jqqw                                  250m (6%)     500m (12%)  256Mi (1%)       512Mi (3%)
  kube-system                heapster-666d5cbb6b-dr77r                                    132m (3%)     132m (3%)   256Mi (1%)       256Mi (1%)
  kube-system                kube-dns-756bfc7fdf-bpcrj                                    260m (6%)     0 (0%)      110Mi (0%)       170Mi (1%)
  kube-system                kube-dns-autoscaler-787d59df8f-bnnht                         20m (0%)      0 (0%)      10Mi (0%)        0 (0%)
  kube-system                kube-proxy-ip-10-10-32-56.ap-northeast-2.compute.internal    100m (2%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kubernetes-dashboard-5bd6f767c7-bmpn5                        0 (0%)        0 (0%)      0 (0%)           0 (0%)
~~~

이 출력에서 노드의 포드를 확인할 수 있다. 전체 요청된 자원뿐아니라 각 포드가 노드에 요청하는 CPU, 메모리 자원에 대한 정보를 확인할 수 있다.

## 클러스터 구성요소

쿠버네티스 클러스터를 구성하는 많은 구성요소가 실제로 쿠버네티스를 통해 배포된다. 이 모든 구성요소들은 kube-system 네임스페이스에서 실행된다.

### 쿠버네티스 프록시

쿠버네티스 프록시는 쿠버네티스 클러스터 내 서비스의 로드밸런싱을 위해 네트워크 트래픽을 라우팅한다. 이 작업은 클러스터의 모든 노드에 프록시가 있어야 가능하다.
쿠버네티스는 데몬세트(DaemonSet) API 객체를 가지고있다. 이 객체는 많은 클러스터에서 프록시 기능을 수행한다.

> $ kubectl get daemonSets --namespace=kube-system kube-proxy

### 쿠버네티스 DNS

쿠버네티스는 클러스터에 정의된 서비스의 이름 지정과 검색을 제공하는 DNS 서버를 실행한다. 이 DNS서버는 클러스터에서 복제된 서비스로 실행된다. 클러스터의 크기에 따라 클러스터에 하나 이상의 DNS 서버가 실행 중임을 확인할 수 있다. DNS서비스는 복제본을 관리하는 쿠버네티스 디플로이먼트(deployment)로 실행된다.

> $ kubectl get deployments --namespace=kube-system kube-dns

~~~
ubuntu@ip-172-31-41-207:~$ kubectl get deployments --namespace=kube-system kube-dns
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-dns   2         2         2            2           59m
~~~

DNS 서버에 대한 로드밸런싱을 수행하는 쿠버네티스 서비스도 있다.

> $ kubectl get services --namespace=kube-system kube-dns

~~~
ubuntu@ip-172-31-41-207:~$ kubectl get services --namespace=kube-system kube-dns
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kube-dns   ClusterIP   100.64.0.10   <none>        53/UDP,53/TCP   1h
~~~

이것은 클러스터의 DNS 서비스 주소가 100.64.0.10 임을 나타낸다. 클러스터의 컨테이너에 로그인하면 컨테이너의 /etc/resolv.conf 파일에 기록되어 있음을 확인할 수 있다.

### 쿠버네티스 UI

UI는 단일 복제본으로 실행되며, 신뢰성과 업그레이드를 위해 쿠버네티스 디플로이먼트로 관리된다. 다음을 사용해 UI 서버를 확인할 수 있다.

> $ kubectl get deployments --namespace=kube-system kubernetes-dashboard

~~~
ubuntu@ip-172-31-41-207:~$ kubectl get deployments --namespace=kube-system kubernetes-dashboard
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubernetes-dashboard   1         1         1            1           55m
~~~

또한 대시보드는 로드밸런싱을 수행하는 서비스를 갖는다.

~~~
ubuntu@ip-172-31-41-207:~$ kubectl get services --namespace=kube-system kubernetes-dashboard
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP        PORT(S)         AGE
kubernetes-dashboard   LoadBalancer   100.69.104.110   a2460a0f883fe...   443:30902/TCP   56m
~~~

kubectl 프록시를 사용해 UI에 접근할 수 있다.
다음 명령을 실행해 쿠버네티스 프록시를 실행한다.

> $ kubectl proxy

~~~
ubuntu@ip-172-31-41-207:~$ kubectl proxy
Starting to serve on 127.0.0.1:8001
~~~

http://localhost:8001/ui에 접속해 쿠버네티스 웹 UI를 볼 수 있다.
