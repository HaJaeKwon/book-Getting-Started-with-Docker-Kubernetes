# SSL 적용된 Remote API 호출

### 목표

- 동일 머신에서 SSL 통신에 필요한 ca.pem, server-cert.pem, server-key.pem, cert.pem, key.pem 파일을 생성한다
- 도커 데몬에 서버쪽 인증서, 키 파일들을 등록하고 https 옵션을 활성화한다
- 도커 클라이언트에서 https 요청을 보내는데 성공한다

### 환경

- Vagrant(centos7)

### 실습

- 책의 내용대로 서버 측, 클라이언트 측 파일 생성
- 도커 데몬

    ```bash
    [vagrant@localhost .docker]$ sudo dockerd --tlsverify --tlscacert=/home/vagrant/.docker/ca.pem --tlscert=/home/vagrant/.docker/server-cert.pem --tlskey=/home/vagrant/.docker/server-key.pem -H=0.0.0.0:2376 -H unix:///var/run/docker.sock
    ```

- 도커 클라이언트

    ```bash
    [vagrant@localhost ~]$ docker -H 192.168.1.1:2376 version
    Client: Docker Engine - Community
     Version:           19.03.12
     API version:       1.40
     Go version:        go1.13.10
     Git commit:        48a66213fe
     Built:             Mon Jun 22 15:46:54 2020
     OS/Arch:           linux/amd64
     Experimental:      false
    Error response from daemon: Client sent an HTTP request to an HTTPS server.

    [vagrant@localhost ~]$ docker -H 192.168.1.1:2376 --tlscacert=/home/vagrant/.docker/ca.pem --tlscert=/home/vagrant/.docker/cert.pem --tlskey=/home/vagrant/.docker/key.pem --tlsverify version
    Client: Docker Engine - Community
     Version:           19.03.12
     API version:       1.40
     Go version:        go1.13.10
     Git commit:        48a66213fe
     Built:             Mon Jun 22 15:46:54 2020
     OS/Arch:           linux/amd64
     Experimental:      false

    Server: Docker Engine - Community
     Engine:
      Version:          19.03.12
      API version:      1.40 (minimum version 1.12)
      Go version:       go1.13.10
      Git commit:       48a66213fe
      Built:            Mon Jun 22 15:45:28 2020
      OS/Arch:          linux/amd64
      Experimental:     false
     containerd:
      Version:          1.2.13
      GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
     runc:
      Version:          1.0.0-rc10
      GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
     docker-init:
      Version:          0.18.0
      GitCommit:        fec3683

    [vagrant@localhost ~]$ curl https://192.168.1.1:2376/version --cert ~/.docker/cert.pem --key ~/.docker/key.pem --cacert ~/.docker/ca.pem
    {"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"19.03.12","Details":{"ApiVersion":"1.40","Arch":"amd64","BuildTime":"2020-06-22T15:45:28.000000000+00:00","Experimental":"false","GitCommit":"48a66213fe","GoVersion":"go1.13.10","KernelVersion":"3.10.0-957.12.2.el7.x86_64","MinAPIVersion":"1.12","Os":"linux"}},{"Name":"containerd","Version":"1.2.13","Details":{"GitCommit":"7ad184331fa3e55e52b890ea95e65ba581ae3429"}},{"Name":"runc","Version":"1.0.0-rc10","Details":{"GitCommit":"dc9208a3303feef5b3839f4323d9beb36df0a9dd"}},{"Name":"docker-init","Version":"0.18.0","Details":{"GitCommit":"fec3683"}}],"Version":"19.03.12","ApiVersion":"1.40","MinAPIVersion":"1.12","GitCommit":"48a66213fe","GoVersion":"go1.13.10","Os":"linux","Arch":"amd64","KernelVersion":"3.10.0-957.12.2.el7.x86_64","BuildTime":"2020-06-22T15:45:28.000000000+00:00"}
    ```

### 결론

- 사설 인증서를 만드는 예제를 해본 느낌
- https 통신의 전체적인 flow는 알겠는데, 인증서, 대칭키, 공개키, 인증서 요청 파일, extfile.cnf ? 각각의 정확한 역할과 쓰이는 순간이 아직 이해가 되지 않는다..
- 참고
    - [https://opentutorials.org/course/228/4894](https://opentutorials.org/course/228/4894)