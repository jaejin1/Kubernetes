# Kubernetes Up & Running

## Kubernetes 설치하기

구글, 애저, 아마존, 로컬등에 설치 할 수 있다. 여기서는 아마존에 AWS에 설치할 것이다.

### AWS IAM - Access keys

* AWS 객체들을 관리하기 위해 IAM 계정의 Access Key를 발급받는다.
* 생성된 Access key ID와 Secret access key는 이따 사용되므로 적어놓는다.
* IAM의 권한은 다음과 같이만 줘도된다.
  * AmazonEC2FullAccess
  * AmazonRoute53FullAccess
  * AmazonS3FullAccess
  * IAMFullAccess
  * AmazonVPCFullAccess

### AWS EC2 생성

* 생성할 instance에 접속하기 위해 private key 발급받는다.
* `jaejin.pem` 같은 파일을 저장해놓는다.
* 빠른 진행을 위해 미리 설치된 AMI로 부터 인스턴스를 생성해도 되지만 처음부터 생성해보려한다.

### AWS EC2 - 접속

* ssh를 이용해 접속한다.

### SSH key gen

* 클러스터를 관리할 ssh-key 생성한다.

> $ ssh-keygen -q -f ~/.ssh/id_rsa -N ''

* 클러스터 내에서 서로 접속 하기 위해 필요하다.

### AWS Credentials

* IAM으로 생성하여 메모해둔 Access key id와 secret access key를 등록한다.

> $ sudo apt update

> $ sudo apt install -y awscli

> $ aws configure

> $ aws iam create-group --group-name kops

> $ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops

> $ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops

> $ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops

> $ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops

> $ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops

> $ aws iam create-user --user-name kops

> $ aws iam add-user-to-group --user-name kops --group-name kops

* AWS에 객체를 생성하기 위해 필요하다.

### Kops

kops를 이용해 쿠버네티스를 설치하기 위해 kops를 설치한다.

> $ sudo snap install kubectl --classic

> $ curl -LO https://github.com/kubernetes/kops/releases/download/1.9.1/kops-linux-amd64

> $ chmod +x kops-linux-amd64

> $ mv ./kops-linux-amd64 /usr/local/bin/kops

에러가 난다면 아마 버전때문일확률이 높다.

### Cluster

* 클러스터 이름을 설정한다.
* 클러스터 상태를 저장할 S3 Bucket을 만들어 준다.
* my_id에는 본인의 이름을 넣는다.

> $ export KOPS_CLUSTER_NAME=awskrug.k8s.local

> $ export KOPS_STATE_STORE=s3://kops-awskrug-my_id

> $ aws s3 mb ${KOPS_STATE_STORE} --region ap-northeast-2

### Create Cluster

> $ kops create cluster \
    --cloud=aws \
    --name=${KOPS_CLUSTER_NAME} \
    --state=${KOPS_STATE_STORE} \
    --master-size=m4.large \
    --node-size=m4.xlarge \
    --node-count=2 \
    --zones=ap-northeast-2a,ap-northeast-2c \
    --network-cidr=10.10.0.0/16

### Update Cluster

* 클러스터 생성 하기 전 수정

> $ kops edit cluster --name=${KOPS_CLUSTER_NAME}

### Create Cluster

* kops update에 --yes 옵션을 주면 실제 클러스터가 생성된다.

> $ kops update cluster --name=${KOPS_CLUSTER_NAME} --yes

### Validate Cluster

* kops validate 명령으로 생성이 완료 되었는지 확인 할 수 있다.

> $ kops validate cluster --name=${KOPS_CLUSTER_NAME}

create cluster했는데 생성이 안되면 기다려야한다. 클러스터 생성까지 5분 ~ 10분 정도 소요되기 때문이다.

### kubectl

* 생성이 완료 되었으면 다음 명령으로 정보를 조회 할 수 있다.

> $ kubectl get nodes
