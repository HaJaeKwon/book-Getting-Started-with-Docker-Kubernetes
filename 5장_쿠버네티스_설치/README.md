# 5장 쿠버네티스 설치

#### kops 사용을 위해 AWS에서 Role을 포함한 IAM 사용자 추가하기
> https://blog.naver.com/alice_k106/221342005691

##### kops 란?
kops(Kubernetes Operation)이란 쿠버네티스 생성 및 관리를 쉽게 하도록 도와주는 오픈소스 툴로서, 프로덕션 레벨의 쿠버네티스 클러스터를 간단한 CLI 명령을 통해 생성, 관리, 업그레이드, 삭제할 수 있도록 지원한다.

AWS EKS를 사용할 수도 있지만, 테스트 프로젝트이고 가격이 부담된다면 kops를 사용해볼 수 있다.

terraform을 통한 프로비저닝도 가능하다.

```shell script
> aws s3api create-bucket --bucket jk-k8s-bucket --create-bucket-configuration LocationConstraint=ap-northeast-2

> aws s3api put-bucket-versioning --bucket jk-k8s-bucket --versioning-configuration Status=Enabled

> export NAME=mycluster.k8s.local

> export KOPS_STATE_STORE=s3://jk-k8s-bucket
kops 명령어를 사용하기 위해 `KOPS_STATE_STORE` 필수

> ssh-keygen -t rsa -N "" -f ./id_rsa

> kops create cluster --zones ap-northeast-2a --networking calico --ssh-public-key ./id_rsa.pub $NAME

> kops edit ig --name=mycluster.k8s.local nodes

> kops edit ig --name=mycluster.k8s.local master-ap-northeast-2a

> kops update cluster --yes $NAME

> kops create secret --name mycluster.k8s.local sshpublickey admin -i id_rsa.pub
로컬 환경에서 진행했다면 AWS의 k8s cluster로 접속하기 위해 SSH Key Pair를 등록해야 한다

 jaekwon  ~/workspace  kops validate cluster
Using cluster from kubectl context: mycluster.k8s.local

Validating cluster mycluster.k8s.local

INSTANCE GROUPS
NAME			ROLE	MACHINETYPE	MIN	MAX	SUBNETS
master-ap-northeast-2a	Master	t2.medium	1	1	ap-northeast-2a
nodes			Node	t2.medium	3	3	ap-northeast-2a

NODE STATUS
NAME							ROLE	READY
ip-172-20-47-172.ap-northeast-2.compute.internal	node	True
ip-172-20-52-77.ap-northeast-2.compute.internal		master	True
ip-172-20-58-239.ap-northeast-2.compute.internal	node	True
ip-172-20-63-183.ap-northeast-2.compute.internal	node	True

VALIDATION ERRORS
KIND	NAME					MESSAGE
Pod	kube-system/kube-dns-64f86fb8dd-8qp5x	system-cluster-critical pod "kube-dns-64f86fb8dd-8qp5x" is not ready (kubedns)

Validation Failed

Validation failed: cluster not yet healthy

# dns 오브젝트가 생성되지 않아 vadliate 는 실패했다

> kops delete cluster mycluster.k8s.local --yes


```
