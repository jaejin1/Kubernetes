kubectl 명령줄 유틸리티는 강력한 도구이다. 이것을 이용해 객체를 생성하고 쿠버네티스 API와 상호작용을 할 것이다. 그 전에 모든 쿠버네티스 객체에 사용할 수 있는 기본적인 kubectl 명령에 대해 먼저 살펴보장.

## 네임스페이스

쿠버네티스는 네임스페이스(namespace)를 사용해 클러스터 내부의 객체를 관리한다. 각 네임스페이스는 객체 집합을 담고 있는 폴더로 생각하면 된다.

기본적으로 kubectl 명령줄 도구는 기본 네임스페이스(default namespace)와 상호작용 한다.
예를들어, kubectl --namespace=mystuff로 실행하면 mystuff 네임스페이스를 참조한다.

## 컨텍스트

기본 네임스페이스를 좀 더 영구적으로 변경하려면 컨텍스트(context)를 사용한다. 이 명령은 $HOME/.kube/config에 있는 kubectl 설정파일에 기록된다. 이 파일은 클러스터를 탐색하고 인증하는 방법도 포함하고 있다.
예를들어 다음 kubectl 명령을 사용해 기본 네임스페이스를 변경할 수 있다.

> $ kubectl config set-context my-context --namespace=mystuff

새로운 컨텍스트를 생성하지만 아직 실제로 사용하지는 않는다. 새로운 컨텍스트의 사용을 위해 다음 명령을 실행한다.

> $ kubectl config use-context my-context

~~~
ubuntu@ip-172-31-41-207:~$ kubectl config set-context my-context --namespace=mystuff
Context "my-context" created.
ubuntu@ip-172-31-41-207:~$ kubectl config use-context my-context
Switched to context "my-context".
~~~

컨텍스트는 각기 다른 클러스터나 사용자를 클러스터에 인증할 때도 사용할 수 있다. 이때는 set-context 명령과 함께 --user나 --cluster 플래그를 사용한다.

## 쿠버네티스 API 객체 보기

쿠버네티스는 자신 내부에 있는 모든 것을 RESTful 자원으로 표현한다. 보고 있는 책에는 이러한 자원을 **쿠버네티스 객체(Kubernetes object)** 로 표현한다. 각 쿠버네티스 객체는 고유한 HTTP 경로를 갖는다.

예를 들어 https://your-k8s.com/api/v1/namespaces/default/pods/my-pod 는 기본 네임스페이스의 my-pod라는 포트를 가리킨다. kubectl 명령은 HTTP 요청을 작성해 이런 URL에 있는 **쿠버네티스 객체로 접근** 한다.

kubectl을 사용해 쿠버네티스 객체를 보는 가장 기본적인 명령은 get이다. 현재 네임스페이스에 있는 모든 자원 목록을 조회하려면 kubectl get <자원 이름>을 실행한다. 반면 특정 자원을 보려면 kubectl get <자원 이름> <객체 이름>을 실행한다.

기본적으로 kubectl은 API 서버 응답을 사람이 읽을 수 있는 형태로 보여준다. 하지만 각 객체의 상세 정보를 터미널의 줄 맞춤으로 많은 정보가 누락되는데, 많은 정보를 얻는 방법 중 하나는 -o wide 플래그를 사용하는 것이다. 그럼 더욱 긴 행을 사용해 더 많은 정보를 보여준다. 객체의 모든 정보를 보고 싶은 경우 -o json 또는 -o yaml 플래그를 사용해 각각 가공되지 않은 형태의 JSON이나 YAML로 볼 수 있다.

kubectl 출력결과의 형태를 변경하는 옵션 중 많이 사용하는 것은 header를 제거하는 것이다.
이것은 종종 유닉스 파이프(|)와 함께 유용하게 사용가능하다. kubectl을 --no-headers 플래그와 함께 사용하면 표 형태의 출력 결과에서 헤더 부분을 생략할 수 있다.

또 다른 옵션은 객체에서 특정 필드를 추출하는 것이다. kubectl은 JSONPath 쿼리 언어를 사용해 반환된 객체에서 필드를 선택할 수 있다.

특정 객체에 대한 세부 정보는 다음 명령의 실행으로 확인한다.

> $ kubectl describe <자원 이름> <객체 이름>

실행 결과 해당 객체에 대한 설명과 함께 클러스터와 관련된 객체와 이벤트 정보를 확인 할 수 있다.

## 쿠버네티스 객체의 생성, 업데이트, 삭제

쿠버네티스 API의 객체는 JSON이나 YAML 파일 형태로 표현된다. 이런 파일은 쿼리에 대한 응답으로 서버에서 반환되거나 API 요청으로 서버에 게시된다. 이런 YAML이나 JSON파일을 사용해 쿠버네티스 서버의 객체를 생성, 업데이트, 삭제할 수 있다.

obj.yaml 파일에 저장된 간단한 객체가 있다고 가정하자. kubectl 명령을 사용해 쿠버네티스에 이 객체를 생성할 수 있다.

> $ kubectl apply -f obj.yaml

~~~
ubuntu@ip-172-31-41-207:~$ kubectl apply -f obj.yaml
W0710 07:15:24.783351    9856 factory_object_mapping.go:423] Failed to download OpenAPI (Get http://localhost:8080/swagger-2.0.0.pb-v1: dial tcp 127.0.0.1:8080: getsockopt: connection refused), falling back to swagger
~~~

이 객체의 자원 종류는 객체 파일에서 가져오므로 별도로 지정할 필요가 없다.

객체를 변경한 뒤에는 apply 명령을 사용해 객체를 업데이트 할 수 있다.

> $ kubectl apply -f obj.yaml

로컬 파일을 수정하는 대신 대화형 편집을 하고 싶은 경우 edit 명령을 사용할 수 있다. 이 명령은 가장 최근의 객체 상태를 다운로드하여 정의가 포함된 편집기를 실행한다.

> $ kubectl edit <자원이름> <객체이름>

~~~
ubuntu@ip-172-31-41-207:~$ kops edit cluster --name=${KOPS_CLUSTER_NAME}
Edit cancelled, no changes made.
~~~

내용은 이런식으로 되어있다.

~~~
apiVersion: kops/v1alpha2
kind: Cluster
metadata:
  creationTimestamp: 2018-07-10T04:48:57Z
  name: awskrug.k8s.local
spec:
  api:
    loadBalancer:
      type: Public
  authorization:
    rbac: {}
  channel: stable
  cloudProvider: aws
  configBase: s3://kops-awskrug-jaejin/awskrug.k8s.local
  etcdClusters:
  - etcdMembers:
    - instanceGroup: master-ap-northeast-2a
      name: a
    name: main
  - etcdMembers:
    - instanceGroup: master-ap-northeast-2a
      name: a
    name: events
  iam:
    allowContainerRegistry: true
    legacy: false
  kubernetesApiAccess:
  - ~~~
  kubernetesVersion: 1.9.6
  masterInternalName: api.internal.awskrug.k8s.local
  masterPublicName: api.awskrug.k8s.local
  networkCIDR: 10.10.0.0/16
  networking:
    kubenet: {}
  nonMasqueradeCIDR: ~~~
  sshAccess:
  - 0.0.0.0/0
  subnets:
  - cidr: ~~~
    name: ap-northeast-2a
    type: Public
    zone: ap-northeast-2a
  - cidr: ~~~
    name: ap-northeast-2c
    type: Public
    zone: ap-northeast-2c
  topology:
    dns:
      type: Public
    masters: public
    nodes: public
~~~

다음 명령으로 객체를 삭제할 수 있다.

> $ kubectl delete -f obj.yaml

자원 종류와 이름을 사용해 객체를 삭제할 수도 있다.

> $ kubectl delete <자원 이름> <객체 이름>

## 라벨과 애노테이션

라벨(label)과 애노테이션(annotation)은 객체용 태그이다. 이 둘의 차이는 06 장에서 다룬다. 여기서는 annotate와 label명령을 사용해 쿠버네티스 객체가 가진 애노테이션과 라벨을 업데이트 하는 방법을 살펴본다.

예를들어, 이름이 bar인 포드에 color=red 라벨을 추가하려면 다음과 같이 명령을 실행한다.

> $ kubectl label pods bar color=red

애노테이션을 사용하는 구문도 동일하다.

label과 annotate 명령은 기본적으로 이미 존재하는 라벨에 대한 덮어쓰기를 수행할 수 없다.
덮어쓰기를 하려면 --overwrite 플래그를 사용해야한다.

라벨의 삭제는 -<라벨 이름> 구문을 사용한다.

> $ kubectl label pods bar -color

위 명령을 실행하면 bar 포드에서 color 라벨을 삭제할 것이다.

## 디버깅 명령

kubectl은 컨테이너를 디버깅하는 명령을 갖고 있다. 동작 중인 컨테이너의 로그를 보려면 다음 명령을 실행한다.

> $ kubectl logs <포드이름>

~~~
ubuntu@ip-172-31-41-207:~$ kubectl logs sample-node-b77d6dbd9-5mxm9
Listening on port 3000!
~~~

포드 내에 다수의 컨테이너가 있는 경우 -c 플래그를 사용하면 특정 컨테이너를 선택하여 볼 수 있다.

kubectl logs는 기본적으로 로그를 보여주고 종료한다. 종료하지 않고 발생하는 로그를 터미널로 지속적으로 보낼려면 -f(follow) 플래그를 사용한다.

또한 exec 명령을 사용해 동작 중인 컨테이너에서 다른 명령을 실행할 수도 있다.

> $ kubectl exec -it <포드 이름> -- bash

명령 실행 후 동작 중인 컨테이너 내부에서 대화형 셸을 이용해 더 많은 디버깅을 수행할 수 있다. 마지막으로 cp명령을 사용하면 컨테이너에서 컨테이너로 파일을 복사할 수 있다.

> $ kubectl cp <포드이름>:/path/to/remote/file /path/to/local/file

위 명령을 실행하면 동작 중인 컨테이너에서 로컬 머신으로 파일을 복사 할 것이다. 특정 디렉토리를 저장하거나 역방향의 경로로 하면 로컬 머신에서 컨테이너로 파일을 복사할 수 있다.

## 요약

kubectl은 쿠버네티스 클러스터의 강력한 애플리케이션 관리 도구이다.
