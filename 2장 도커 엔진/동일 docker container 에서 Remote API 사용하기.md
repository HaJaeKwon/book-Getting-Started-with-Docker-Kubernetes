# 동일 docker container 에서 Remote API 사용하기 실습

### 목표

- 동일 호스트에서 rest api, remote api 를 이용해 docker container 정보를 조회한다

### 환경

- Vagrant(centos7)

### 실습

- step1 - dockerd 재시작

    ```bash
    [root@10 vagrant]# dockerd -H tcp://192.168.0.1:2375 -H unix://var/run/docker.sock
    INFO[2020-07-26T17:16:56.364532843Z] Starting up
    WARN[2020-07-26T17:16:56.365221705Z] [!] DON'T BIND ON ANY IP ADDRESS WITHOUT setting --tlsverify IF YOU DON'T KNOW WHAT YOU'RE DOING [!]
    failed to load listeners: can't create unix socket var/run/docker.sock: listen unix var/run/docker.sock: bind: no such file or directory
    [root@10 vagrant]# dockerd -H tcp://192.168.0.1:2375 -H unix://var/run/docker.sock --debug
    INFO[2020-07-26T17:17:34.881515582Z] Starting up
    ...
    INFO[2020-07-26T17:17:53.255631869Z] API listen on /var/run/docker.sock
    INFO[2020-07-26T17:17:53.255655293Z] API listen on 192.168.0.1:2375
    ```

    - 이때 기존 dockerd 이 실행 중이면 root 권한이라도 실패한다. systemctl stop docker
- step2 - rest api 호출

    ```bash
    [vagrant@10 ~]$ curl 192.168.0.1:2375/version
    {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"19.03.12","Details":{"ApiVersion":"1.40","Arch":"amd64","BuildTime":"2020-06-22T15:45:28.000000000+00:00","Experimental":"false","GitCommit":"48a66213fe","GoVersion":"go1.13.10","KernelVersion":"3.10.0-957.12.2.el7.x86_64","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.2.13","Details":{"GitCommit":"7ad184331fa3e55e52b890ea95e65ba581ae3429"}},{"Name":"runc","Version":"1.0.0-rc10","Details":{"GitCommit":"dc9208a3303feef5b3839f4323d9beb36df0a9dd"}},{"Name":"docker-init","Version":"0.18.0","Details":{"GitCommit":"fec3683"}}],"Version":"19.03.12","ApiVersion":"1.40","MinAPIVersion":"1.12","GitCommit":"48a66213fe","GoVersion":"go1.13.10","Os":"linux","Arch":"amd64","KernelVersion":"3.10.0-957.12.2.el7.x86_64","BuildTime":"2020-06-22T15:45:28.000000000+00:00"}
    ```

### 결론

- docker 호스트머신에서 docker server를 IP:PORT 로 바인딩 시켜두면 외부에서 해당 IP:PORT를 통해 docker 제어가 가능하다.
- 이런 기능으로 사내 전체 docker 머신을 제어하는 모니터링 시스템도 만들 수 있을듯
- 보안을 위해 https 적용 필요