k8s의 상태를 나타내는 엔티티를 Object라 하고, object가 Spec에 정의된 상태로 유지 될 수 있도록 지속적으로 변화시키는 주체를 Controller라고 한다.

**ReplicaSet은 Pod(object)을 복제 생성하고, 복제된 Pod의 개수를 (Spec에 정의된 개수만큼) 지속적으로 유지하는 Controller이다.**

## ReplicationController vs Replicaset

ReplicaSet 공식문서에 **ReplicaSet is the next-generation Replication Controller** 라고 설명하고 있다. ReplicationController는 deprecated될 예정이고 현재는 기존 ReplicationController가 하던 역할을 ReplicaSet과 이후에 나올 deployment가 대신하고 있다.

ReplicationController에서 ReplicaSet으로 변경되면서 달라진 점 중 하나는 **set-based selector** 의 지원이다.

ReplicaSet은 관리할 대상 Pod을 찾을때 Label에 매칭되는 Pod을 찾게 되는데, 이때 ReplicationController는 동일한 문자열의 label만 찾을 수 있는 반면 ReplicaSet은 다음과같이 `IN, NotIn, Exist, DoesNotExist` operator를 사용해서 찾을 수 있다.

~~~
selector:
  # ReplicationController는 matchLabels만 사용가능
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
    - {key: service, operator: Exists, values: [user]}
    - {key: service, operator: DoesNotExist, values: [db]}
~~~

## ReplicaSet을 사용해보장

* hello-node.yml

~~~
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello-node
  labels:
    service-name: hello-node
spec:
  replicas: 2
  selector:
    matchLabels:
      service-name: hello-node
  template:
    metadata:
      name: hello-node
      labels:
        service-name: hello-node
    spec:
      containers:
      - name: hello-node
        image: asbubam/hello-node
        readinessProbe:
          httpGet:
            path: /
            port: 8080
        livenessProbe:
          httpGet:
            path: /
            port: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: hello-node
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    service-name: hello-node
~~~

전 hello-node.yml에서

~~~
spec:
  replicas: 2
  selector:
    matchLabels:
      service-name: hello-node
  template:
    metadata:
      name: hello-node
      labels:
        service-name: hello-node
~~~

부분만 추가 되었다.

~~~
{18-07-11 10:21}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl create -f hello-node.yml --save-config
replicaset.apps/hello-node created
service/hello-node created
{18-07-11 10:21}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get rs
NAME         DESIRED   CURRENT   READY     AGE
hello-node   2         2         0         10s
{18-07-11 10:21}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin%
{18-07-11 10:21}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin%
{18-07-11 10:21}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get rs
NAME         DESIRED   CURRENT   READY     AGE
hello-node   2         2         2         24s
{18-07-11 10:21}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pods
NAME               READY     STATUS    RESTARTS   AGE
hello-node-45nn5   1/1       Running   0          34s
hello-node-fq9z8   1/1       Running   0          34s
~~~

동일한 역할을 수행하는 2개의 hello-node pod이 생성되었는데, http 요청은 2개의 pod중 어떤 pod으로 연결되나? 어떻게 이뤄지는걸까요?

## ReplicaSet이 생성되는 과정

`kubectl create`명령으로 ReplicaSet 생성을 요청하면 생성하고, ReplicaSet Controller에 의해서 Pod을 생성한다.

kubectl describe rs 명령의 결과에서 ReplicaSet Controller에 의해서 2개의 Pod이 생성되었음을 확인 할 수 있다.

~~~
{18-07-11 10:36}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl describe rs
Name:         hello-node
Namespace:    default
Selector:     service-name=hello-node
Labels:       service-name=hello-node
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"service-name":"hello-node"},"name":"hello-node","namespace":"defaul...
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  service-name=hello-node
  Containers:
   hello-node:
    Image:        asbubam/hello-node
    Port:         <none>
    Host Port:    <none>
    Liveness:     http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  15m   replicaset-controller  Created pod: hello-node-fq9z8
  Normal  SuccessfulCreate  15m   replicaset-controller  Created pod: hello-node-45nn5
~~~

## hello-node.yml

~~~
apiVersion: apps/v1 # apps 는 api endpoint의 group을 의미한다.
kind: ReplicaSet
metadata:
  name: hello-node # ReplicaSet의 고유 이름
  labels:          # 외부에서 ReplicaSet을 찾을 때 사용할 label. 복수의 label 입력가능
    service-name: hello-node
spec:              # 기대되는
  replicas: 2      # 2개의 Pod 복제(replica)를 생성한다.
  selector:        # 위에서 설명했던 Label Selector - 이 Label에 매칭하는 Pod을 관리
    matchLabels:   # 여기서는 label이 동일한 Pod을 select한다.
      service-name: hello-node # service-name label이 hello-node인 Pod을 2개 유지
  template:        # Pod 생성 시 사용할 template를 정의한다. 기존의 Pod 정의와 동일하다.
    metadata:
      name: hello-node
      labels:
        service-name: hello-node
    spec:
      containers:
      - name: hello-node
        image: asbubam/hello-node
        readinessProbe:
          httpGet:
            path: /
            port: 8080
        livenessProbe:
          httpGet:
            path: /
            port: 8080

--- # 한개의 yaml파일에 여러개의 Object를 정의할 때 구분자로 사용한다.
# Service Object 정의 - 다음 글에서 자세히 설명한다.
apiVersion: v1
kind: Service
metadata:
  name: hello-node
spec:
  type: LoadBalancer # 외부에 Pod을 어떤 형태로 노출할지 결정
  ports:
  - port: 8080       # 외부에 노출할 포트
    targetPort: 8080 # 컨테이너의 포트
  selector:
    service-name: hello-node # Service가 연결할 대상 Pod의 label
~~~

* apiversion 이 V1이였는데 apps/v1을 사용하고 있다. apps는 api endpodt의 group을 의미한다.
* ReplicaSet 정의를 위해서는 replicas: 2로 replica 개수를 설정하고, matchLabels(label selector)를 통해서 Replicaset의 관리대상 Pod을 정의 하는 것이 전부다.

## Pod replica 개수를 변경해보자

replicas: 3으로 파일 내용을 변경

~~~
{18-07-11 10:39}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% vi hello-node.yml
{18-07-11 10:39}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl apply -f hello-node.yml
replicaset.apps/hello-node configured
service/hello-node unchanged
~~~

새로은 Pod이 한개 추가되어 3개의 Pod이 실행상태로 유지된다.

~~~
{18-07-11 10:39}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pods
NAME               READY     STATUS    RESTARTS   AGE
hello-node-45nn5   1/1       Running   0          18m
hello-node-fq9z8   1/1       Running   0          18m
hello-node-sws8g   1/1       Running   0          33s
{18-07-11 10:40}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get rs
NAME         DESIRED   CURRENT   READY     AGE
hello-node   3         3         3         19m
{18-07-11 10:40}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl describe rs hello-node
Name:         hello-node
Namespace:    default
Selector:     service-name=hello-node
Labels:       service-name=hello-node
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"service-name":"hello-node"},"name":"hello-node","namespace":"defaul...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  service-name=hello-node
  Containers:
   hello-node:
    Image:        asbubam/hello-node
    Port:         <none>
    Host Port:    <none>
    Liveness:     http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  19m   replicaset-controller  Created pod: hello-node-fq9z8
  Normal  SuccessfulCreate  19m   replicaset-controller  Created pod: hello-node-45nn5
  Normal  SuccessfulCreate  44s   replicaset-controller  Created pod: hello-node-sws8g
~~~

## Pod의 복구

ReplicaSet은 **Pod의 개수를 desired 상태** 로 유지한다.

1개의 Pod을 종료시키면 replicaSet은 새로운 Pod을 생성한다.

~~~
{18-07-11 10:41}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pods
NAME               READY     STATUS    RESTARTS   AGE
hello-node-45nn5   1/1       Running   0          20m
hello-node-fq9z8   1/1       Running   0          20m
hello-node-sws8g   1/1       Running   0          2m
{18-07-11 10:42}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl delete pod hello-node-45nn5
pod "hello-node-45nn5" deleted
{18-07-11 10:42}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pod
NAME               READY     STATUS    RESTARTS   AGE
hello-node-fq9z8   1/1       Running   0          21m
hello-node-pb5zz   0/1       Running   0          7s
hello-node-sws8g   1/1       Running   0          2m
{18-07-11 10:42}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl describe rs hello-node
Name:         hello-node
Namespace:    default
Selector:     service-name=hello-node
Labels:       service-name=hello-node
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"service-name":"hello-node"},"name":"hello-node","namespace":"defaul...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  service-name=hello-node
  Containers:
   hello-node:
    Image:        asbubam/hello-node
    Port:         <none>
    Host Port:    <none>
    Liveness:     http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:    http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  21m   replicaset-controller  Created pod: hello-node-fq9z8
  Normal  SuccessfulCreate  21m   replicaset-controller  Created pod: hello-node-45nn5
  Normal  SuccessfulCreate  2m    replicaset-controller  Created pod: hello-node-sws8g
  Normal  SuccessfulCreate  23s   replicaset-controller  Created pod: hello-node-pb5zz
~~~

## ReplicaSet은 Pod을 어떻게 찾을까??

ReplicaSet은 Label을 통해서 관리 대상 Pod을 selecting한다고 했는데 그럼 Pod에서 해당 Label을 삭제하면 어떤 현상이 발생할까?

먼저 -show-labels 옵션을 사용해서 현재 생성되어 있는 Pod을 조회하고

~~~
{18-07-11 10:44}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pod --show-labels
NAME               READY     STATUS    RESTARTS   AGE       LABELS
hello-node-fq9z8   1/1       Running   0          22m       service-name=hello-node
hello-node-pb5zz   1/1       Running   0          1m        service-name=hello-node
hello-node-sws8g   1/1       Running   0          4m        service-name=hello-node
~~~

첫번째 **hello-node-fq9z8"** Pod에서 `service-name` Label을 삭제한다.

~~~
{18-07-11 10:44}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl label pods/hello-node-fq9z8 service-name-
pod/hello-node-fq9z8 labeled
~~~

다시한번 `kubectl get pod -show-labels`명령을 실해하면 `service-name=hello-node` Label을 가진 새로운 container가 생성됨을 확인할 수 있다.

~~~
{18-07-11 10:45}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pod --show-labels
NAME               READY     STATUS    RESTARTS   AGE       LABELS
hello-node-64lnv   1/1       Running   0          25s       service-name=hello-node
hello-node-fq9z8   1/1       Running   0          24m       <none>
hello-node-pb5zz   1/1       Running   0          3m        service-name=hello-node
hello-node-sws8g   1/1       Running   0          5m        service-name=hello-node
~~~

label을 삭제한 **hello-node-fq9z8** Pod은 hello-node-ReplicaSet의 대상에서만 제외되었을뿐, 계속 Running 상태를 유지하고 있음에 유의.

**hello-node-fq9z8** Pod에 `service-name=hello-node` Label을 다시 추가하자.

~~~
{18-07-11 10:45}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl label pods/hello-node-fq9z8 service-name=hello-node
pod/hello-node-fq9z8 labeled
{18-07-11 10:46}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pod --show-labels
NAME               READY     STATUS    RESTARTS   AGE       LABELS
hello-node-fq9z8   1/1       Running   0          25m       service-name=hello-node
hello-node-pb5zz   1/1       Running   0          4m        service-name=hello-node
hello-node-sws8g   1/1       Running   0          6m        service-name=hello-node
~~~

## ReplicaSet은 Replication 관리만 한다.

당연한 얘기지만 ReplicaSet은 Pod의 Replication 관리만 한다. ReplicaSet에 의해 3개의 Pod이 실행 중인 상태에서 ReplicaSet을 삭제하면 실행 중인 Pod은 어떻게 뙬까?

~~~
{18-07-11 10:54}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl delete rs hello-node --cascade=false
replicaset.extensions "hello-node" deleted
{18-07-11 10:54}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기/ReplicaSet@master✗✗✗✗✗✗ jaejin% kubectl get pods --show-labels
NAME               READY     STATUS    RESTARTS   AGE       LABELS
hello-node-fq9z8   1/1       Running   0          33m       service-name=hello-node
hello-node-pb5zz   1/1       Running   0          12m       service-name=hello-node
hello-node-sws8g   1/1       Running   0          14m       service-name=hello-node
~~~

**cascade=false** 옵션으로 ReplicaSet을 삭제할 경우 해당 ReplicaSet의 관리 대상이였던 Pod은 삭제되지 않고 실행상태로 유지 간으하다. ReplicaSet이 Pod을 소유하는 개념이 아닌 특정한 Label Selector룰에 따라 Pod의 개수/상태를 유지하는 역할만 수행함을 확인할 수 있다.
