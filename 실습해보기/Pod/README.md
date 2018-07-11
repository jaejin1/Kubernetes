## Pod 생성하기

asbubam's blog를 참조했다.

이 분이 Docker hub에 올린 Docker image를 이용한다.

> $ docker run -d -p 8080:8080 asbubam/hello-node
d3e4902c8140a29d0f5a87d267f7d72bb568f7d03b026831036d2e9071e80e16

> $ curl http://localhost:8080

~~~
{18-07-11 9:41}jaejin-ui-MacBook-Pro:~ jaejin% docker run -d -p 8080:8080 asbubam/hello-node
d3e4902c8140a29d0f5a87d267f7d72bb568f7d03b026831036d2e9071e80e16
Unable to find image 'asbubam/hello-node:latest' locally
latest: Pulling from asbubam/hello-node
10a267c67f42: Pull complete
fb5937da9414: Pull complete
9021b2326a1e: Pull complete
dbed9b09434e: Pull complete
74bb2fc384c6: Pull complete
9b0a326fab3b: Pull complete
8089dfd0519a: Pull complete
f2be1898eb92: Pull complete
88c75a4701d0: Pull complete
f0e706fb13e8: Pull complete
a36c184b15a1: Pull complete
98fd9c1d5bd4: Pull complete
Digest: sha256:3d7137c208e1f3cccbd64a206d178751d0a1df07f6e5faf6fa9419454fa349a8
Status: Downloaded newer image for asbubam/hello-node:latest
888d9c165b529b6a16a6afce400a39b7600f15646cf8ecb8170799023a5b9402
~~~

~~~
{18-07-11 9:49}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% curl http://localhost:8080
Hello World!
~~~

Pod은 `kubectl run` 명령을 통해 실행할 수도 있지만 yaml 파일을 사용해서 Pod을 정의하고 수정사항을 적용하면서 진행한다. 실제 k8s 클래스터를 사용할 때도 yaml파일을 계속 작성하고 또 업데이트하는 작업을 반복하게 된다.

다음은 hello-node Pod을 정의한 hello-node.yml 파일 이다.

~~~
apiVersion: v1 # K8s api version, 특정 object는 v1beta를 사용한다.
kind: Pod      # 생성할 object, controller의 종류
metadata:      
  name: hello-node   # object의 고유 이름, 복수로 생성할 경우 prefix로 사용된다.
  labels:            # 외부에서 Pod을 찾을 때 사용할 label. 복수의 label 입력가능
    service-name: hello-node
spec:                # 기대되는 obejct의 상태
  containers:        # Pod 내부에서 생성할 컨테이너 목록
  - name: hello-node # 컨테이너 이름
    image: asbubam/hello-node # 컨테이너를 실행할 이미지
    readinessProbe:
      httpGet:       # health check 방법 exec, httpGet, tcpSocket
        path: /      # health check path
        port: 8080   # health check port
    livenessProbe:
      httpGet:
        path: /
        port: 8080

--- # 주석 한개의 yaml파일에 여러개의 Object를 정의할 때 구분자로 사용한다.
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

Pod을 생성하자.

~~~
{18-07-11 9:49}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl create -f hello-node.yml
pod/hello-node created
service/hello-node created
{18-07-11 9:49}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl get pods
NAME         READY     STATUS    RESTARTS   AGE
hello-node   0/1       Running   0          8s
{18-07-11 9:49}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl get pods
NAME         READY     STATUS    RESTARTS   AGE
hello-node   1/1       Running   0          12s
{18-07-11 9:49}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% curl http://localhost:8080
Hello World!
~~~

Pod 생성시 service가 같이 생성되는데 외부에서 Pod에 접속하기 위해서는 Service나 Ingress Controller를 통해 Pod의 포트를 expose 해줘야하나다.

## Pod 관련 kubectl 명령

* kubectl describe pod : 상세한 Pod의 현재 상태, 발생한 이벤트 기록 조회한다.

~~~
{18-07-11 9:50}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl describe pod hello-node
Name:         hello-node
Namespace:    default
Node:         docker-for-desktop/192.168.65.3
Start Time:   Wed, 11 Jul 2018 09:49:46 +0900
Labels:       service-name=hello-node
Annotations:  <none>
Status:       Running
IP:           10.1.0.4
Containers:
  hello-node:
    Container ID:   docker://db88a41a10e4494397f79480d6595ae3bbb9ea994adb238ac3a4d9e07d27f9c1
    Image:          asbubam/hello-node
    Image ID:       docker-pullable://asbubam/hello-node@sha256:3d7137c208e1f3cccbd64a206d178751d0a1df07f6e5faf6fa9419454fa349a8
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 11 Jul 2018 09:49:50 +0900
    Ready:          True
    Restart Count:  0
    Liveness:       http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-jzrsj (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-jzrsj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-jzrsj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  Scheduled              12m   default-scheduler            Successfully assigned hello-node to docker-for-desktop
  Normal  SuccessfulMountVolume  12m   kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-jzrsj"
  Normal  Pulling                12m   kubelet, docker-for-desktop  pulling image "asbubam/hello-node"
  Normal  Pulled                 12m   kubelet, docker-for-desktop  Successfully pulled image "asbubam/hello-node"
  Normal  Created                12m   kubelet, docker-for-desktop  Created container
  Normal  Started                12m   kubelet, docker-for-desktop  Started container
~~~

* kubectl logs : 컨테이너 로그를 확인할 수 있다.

~~~
{18-07-11 10:03}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl logs hello-node
npm info it worked if it ends with ok
npm info using npm@3.10.10
npm info using node@v6.10.3
npm info lifecycle hello-node@0.0.1~prestart: hello-node@0.0.1
npm info lifecycle hello-node@0.0.1~start: hello-node@0.0.1

> hello-node@0.0.1 start /usr/src/app
> node server.js

hello-node app listening on port 8080
~~~

* kubectl exec : Pod 컨테이너에 명령을 수행한다.

~~~
{18-07-11 10:04}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl exec -it hello-node ps aux | grep server.js
root        18  0.0  1.5 886504 32244 ?        Sl   00:49   0:00 node server.js
{18-07-11 10:04}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl exec -it hello-node sh
# ls
Dockerfile  circle.yml	docker	      package.json  test
README.md   deployment	node_modules  server.js
# exit
~~~

* kubectl delete pod : pod 삭제

~~~
{18-07-11 10:04}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl delete pod hello-node
pod "hello-node" deleted
{18-07-11 10:05}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl delete -f hello-node.yml
service "hello-node" deleted
Error from server (NotFound): error when deleting "hello-node.yml": pods "hello-node" not found
{18-07-11 10:05}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl create -f hello-node.yml
pod/hello-node created
service/hello-node created
{18-07-11 10:05}jaejin-ui-MacBook-Pro:~/dev/kubernetes/github/실습해보기@master✗✗✗✗✗✗ jaejin% kubectl delete -f hello-node.yml
pod "hello-node" deleted
service "hello-node" deleted
~~~
