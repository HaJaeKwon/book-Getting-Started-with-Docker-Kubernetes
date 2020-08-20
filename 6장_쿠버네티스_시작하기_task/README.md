# 6장 쿠버네티스 시작하기

#### 쿠버네티스는 여러개의 컴포넌트로 구성돼 있습니다.
* 마스터 노드에서는 API 서버 (kube-apiserver), 컨트롤러 매니저(kube-controller-manager), 스케줄러(ube-scheduler), DNS 서버 (coreDNS)
* 모든 노드에서 오버레이 네트워크 구성을 위해 프락시(kube-proxy) 와 네트워크 플러그인(calico, flannel 등)
* 쿠버네티스 클러스터 구성을 위해 kubelet 이라는 에이전트가 모든 노드에서 실행
* 쿠버네티스에서 반드시 도커를 사용해야 하는 것은 아니며, OCI(Open Container Initiative) 라는 컨테이너의 런타임 표준을 구현한 CRI(Container Runtime Interface) 를 갖추고 있다면 어떠한 컨테이너를 써도 문제는 없습니다

##### kubectl 명령어

* kubectl api-resources
* kubectl apply -f nginx-pod.yml
* kubectl get pods
* kubectl describe pods my-nginx-pod
* kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart-Never bash
* kubectl exec -it my-nginx-pod bash
* kubectl logs my-nginx-pod
* kubectl delete -f nginx-pod.yml
* -c
  * kubectl exec, logs 등의 명령을 수행할때 파드의 어떤 컨테이너에 대해 명령어를 수행할지 명시할 수 있습니다
  
### 포드(Pod): 컨테이너를 다루는 기본 단위

#### kubectl 로 nginx pod 만들어보기

```shell script
kubectl apply -f nginx-pod.yml
```

같은 클러스터 내부여야 접근 가능

```shell script
> kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart=Never bash
If you don't see a command prompt, try pressing enter.
root@debug:/# 
root@debug:/# curl 10.1.0.72
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

#### 포드 vs 도커 컨테이너

* 쿠버네티스가 pod를 사용하는 이유는 컨테이너 런타임의 인터페이스 제공 등 여러 가지가 있지만, 그 이유 중 하나는 여러 리눅스 네임스페이스를 공유하는 여러 컨테이너를 추상화된 집합으로 사용하기 위해서 입니다.
* pod 에는 1개 이상의 컨테이너가 포함될 수 있습니다.

nginx-pod-with-ubuntu.yml 실행해보면 1개 pod, 2개 container 가 실행되는데 ubuntu-sidecar-container 에 접속해서 curl localhost 해보면 nginx 에 접근이 되는 것을 확인할 수 있다

이는 포드 내의 컨테이너들이 네트워크 네임스페이스 등과 같은 리눅스 네임스페이스를 공유하기 때문이다. 동일한 네트워크 환경을 가진다

#### 완전한 애플리케이션으로서의 포드

Nginx 컨테이너가 실행되기 위해 부가적인 기능이 필요할 수 있습니다.

Nginx의 설정 파일의 변경사항을 갱신해주는 설정 리로더 프로세스나 로그를 수집하는 프로세스 등이 같이 실행되어야 할 수 있습니다.

이런 기능확장을 위해 추가 컨테이너를 함께 포드에 포함시킬 수 있습니다.

이런 부가적인 컨테이너를 사이드카 컨테이너라고 하고, 포드 내의 다른 컨테이너와 네트워크 환경 등을 공유하게 됩니다.

### 레플리카셋(Relica Set): 일정 개수의 포드를 유지하는 컨트롤러

포드만 yaml 파일에 정의해 사용하는 것은 여러가지 한계점이 있습니다.
> kubectl get pod -o wide 로 사용하는 컨테이너 ip 를 확인 후 직접 종료해주면 종료된 상태로 컨테이너가 남게됩니다.

이런 한계점을 해결하기 위해 레플리카셋이라는 쿠버네티스 오브젝트를 함께 사용하는 것이 일반적입니다.
* 정해진 수의 동일한 포드가 항상 실행되도록 관리합니다.
* 노드 장애 등의 이유로 포드를 사용할 수 없다면 다른 노드에서 포드를 다시 생성합니다.

> kubectl apply -f replicaset-nginx.yml

레플리카셋을 삭제하거나 수정하면 포드에 변경사항이 반영되고, 레플리카셋은 일정 개수의 포드를 유지하도록 합니다.

* 이미 생성된 리소스의 속성을 변경하기
    * kubectl edit, kubectl patch, kubectl appy -f
    
**포드와 레플리카셋은 연결되있지 않습니다.** 느슨한 연결 (loosely coupled) 은 라벨 셀렉터를 이용해 이뤄집니다.

* kubectl get pods --show-labels
* kubectl get pods -l app
* kubectl get pods -l app=my-nginx-pods-label

포드에 변경사항을 주고, 레플리카셋의 동작을 테스트해보기

* 생성된 포드를 수동으로 삭제하기
  * kubectl delete pods my-nginx-pod
* 생성된 포드의 라벨을 제거하기
  * kubectl edit pods replicaset-nginx-2fhd7
* 라벨을 가진 포드를 미리 만들어두고 레플리카셋을 생성하기

### 레플리케이션 컨트롤러 vs 레플리카셋

이전 버전의 쿠버네티스에서는 레플리케이션 컨트롤러를 사용했습니다.

가장 큰 차이는 표현식(matchExpression) 기반의 라벨 셀렉터를 사용할 수 있다는 점입니다.
```yaml
...
selector:
    matchExpression:
      - key: app
        values:
          - my-nginx-pods-label
          - your-nginx-pods-label
        operator: In
    template:
...
```

### 디플로이먼트(Deployment): 레플리카셋, 포드의 배포를 관리

실제로 쿠버네티스 운영환경에서는 디플로이먼트라는 이름의 오브젝트를 yaml 파일에 정의해 사용합니다.

디플로이먼트는 레플리카셋의 상위 오브젝트이기에 생성하면 포드와 레플리카셋을 함께 생성됩니다.

> kubectl apply -f deployment-nginx.yml

#### 디플로이먼트를 사용하는 이유

애플리케이션을 업데이트할 때 레플리카셋의 변경 사항을 저장하는 리비전을 남겨 롤백을 가능하게 해주고, 무중단 서비스를 위해 포드의 롤링 업데이트의 전략을 지정할 수도 있습니다.

> kubectl apply -f deployment-nginx.yml --record

> kubectl set image deployment my-nginx-deployment nginx=nginx:1.11 --record

> kubectl rollout history deployment my-nginx-deployment

> kubectl rollout undo deployment my-nginx-deployment --to-revision=1

#### 서비스(Service): 포드를 연결하고 외부에 노출

포트를 외부로 노출하거나, 다른 디플로이먼트의 포드들이 내부적으로 접근하려면 서비스라고 부르는 별도의 쿠버네티스 오브젝트를 생성해야 합니다.

서비스의 핵심 기능
* 여러개의 포드에 쉽게 접근할 수 있도록 고유한 도메인 이름을 부여합니다.
* 여러 개의 포드에 접근할 때, 요청을 분산하는 로드 밸런서 기능을 수행합니다.
* 클라우드 플랫폼의 로드 밸런서, 클러스터 노드의 포트 등을 통해 포드를 외부로 노출합니다.

서비스를 생성하기 전에 먼저 디플로이먼트를 먼저 생성합니다

> kubectl apply -f deployment-hostname.yml

> kubectl get pods -o wide
>
> -o 옵션을 사용하면 리소스의 정보 출력 형태를 변경할 수 있습니다. yaml, json, wide

> kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart=Never curl 10.1.0.90

목적에 따라 적절한 서비스의 종류를 선택해야 합니다.

* ClusterIP 타입: 쿠버네티스 내부에서만 포드들에 접근할 때 사용합니다. 외부로 포드를 노출하지 않기 때문에 쿠버네티스 클러스터 내부에서만 사용되는 포드에 적합니다.
* NodePort 타입: 포드에 접근할 수 있는 포트를 클러스터의 모든 노드에 동일하게 개방합니다. 따라서 외부에서 포드에 접근할 수 있는 서비스 타입입니다.
* LoadBalancer 타입: 클라우드 플랫폼에서 제공하는 로드 밸런서를 동적으로 프로비저닝해 포드에 연결합니다. 일반적으로는 클라우드 플랫폼 환경에서만 사용할 수 있습니다.


##### ClusterIP

> kubectl apply -f hostname-svc-clusterip.yaml

> kubectl run -i --tty --rm debug --image=alicek106/ubuntu:curl --restart=Never -- bash

> root@debug:/# curl 10.98.156.131:8080 --silent

동일한 서비스 IP:PORT 로 요청을 보내면 서로 다른 컨테이너로 연결되는 것을 확인할 수 있습니다. (내부 DNS 사용)

> 서비스의 라벨 셀렉터와 포드의 라벨이 매칭되 연결되면 자동으로 엔드포인트 오브젝트를 별도로 생성합니다.
>
> 서비스가 가리키고 있는 도착점을 나타냅니다.
>
> kubectl get ep

##### NodePort

실제 운영 환경에서 NodePort로 서비스를 외부에 제공하기는 포트번호를 80, 443으로 설정하기에는 적절치 않아 제약사항이 많습니다.

외부 오픈을 위해서는 **인그레스**라고 부르는 오브젝트를 사용하는 편

> 특정 클라이언트가 같은 포드로만 처리되도록 하려면 sessionAffinity 항목을 ClientIP 로 설정한다

##### LoadBalancer

LoadBalancer 과정은 AWS에서 진행해야해서 실습은 추후에..

아마도 kubectl 명령어를 EC2 에서 실행하거나, kubectl 을 AWS 랑 연동해두는 작업이 필요할 것 같다

1. LoadBalancer 타입의 서비스가 생성됨과 동시에 모든 워커 노드는 포드에 접근할 수 있는 랜덤한 포트를 개방합니다.
2. 클라우드 플랫폼에서 생성된 로드 밸런서로 요청이 들어오면 이 요청은 쿠버네티스의 워커 노드 중 하나로 전달되며, 이때 사용되는 포트는 1번에서 개방된 포트인 32620 포트입니다.
3. 워커 노드로 전달된 요청은 포드 중 하나로 전달되어 처리됩니다.

> AWS 로드 밸런서의 타입이 기본은 classic 인데, metadata.annotation 항목을 정의하면 NLB도 사용할 수 있다

#### 트래픽의 분배를 결정하는 서비스의 속성: externalTrafficPolicy

LoadBalancer 타입의 서비스를 사용하면 외부로부터 들어온 요청은 각 노드 중 하나로 보내지며, 그 노드에서 다시 포드 중 하나로 전달됩니다.

이는 기본값인 Cluster 의 동작입니다.

다른 포드에서 요청이 전달되면 SNAT 가 발생해 트래픽의 출발지 주소가 바뀌는 문제가 발생합니다.

Local 로 설정하게 되면 로드밸런서는 포드가 위치한 노드로만 요청을 전달하며, 해당 노드 내의 포드에서만 요청이 분산됩니다.

Local 로 설정하는게 무조건 좋은 것은 아니고 각 노드에 포드가 고르지 않게 스케줄링 되었을때 요청이 고르게 분산되지 않을 수 있습니다.

> 쿠버네티 스케쥴링 기능 중 PodAntiAffinity 등을 사용하면 포드를 최대한 고르게 배포할 수 있습니다.

#### 요청을 외부로 리다이렉트하는 서비스: ExternalName

서비스의 타입을 ExternalName 으로 생성하면 서비스가 외부 도메인을 가리키도록 설정할 수 있습니다.

> CNAME, A Record 의 차이점: https://dev.plusblog.co.kr/30
