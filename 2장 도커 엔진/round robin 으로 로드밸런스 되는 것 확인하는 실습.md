# round robin 으로 로드밸런스 되는 것 확인하는 실습

### 목표

- driver를 bridge 타입으로 설정한 docker netwrok 생성할 수 있다
- 3개의 container 를 미리 생성한 docker network 에 연결한다
- 외부 docker container 에서 3개의 container에 hostname 으로 접근시 라운드로빈으로 각 서버가 응답하는 것을 확인한다

### 환경

- vagrant(centos7) 가상 머신
- VirtualBox 설치 필요

```bash
# vim Vagrantfile

# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.network "private_network", ip: "192.168.0.1"
    config.vm.network "forwarded_port", guest: 22, host: 1234, id: "ssh"
    config.vm.provider "virtualbox" do |vb|
        vb.memory = 4096
        vb.cpus = 4
    end
  config.vm.box = "centos/7"
  config.vm.provision "shell", inline: <<-SHELL
    yum install net-tools -y
    yum install bridge-utils -y
  SHELL
  config.vm.provision :docker,
    images: ["ubuntu:14.04"]
end
```

### 실습

- step1 - 기존 개발환경과의 격리를 위해 가상머신 생성, 접속
    - vagrant up
    - vagrant ssh

    ```bash
    [vagrant@10 ~]$ ifconfig
    docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
            ether 02:42:b2:f9:1c:ab  txqueuelen 0  (Ethernet)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
            inet6 fe80::5054:ff:fe8a:fee6  prefixlen 64  scopeid 0x20<link>
            ether 52:54:00:8a:fe:e6  txqueuelen 1000  (Ethernet)
            RX packets 551789  bytes 774310971 (738.4 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 104952  bytes 6941460 (6.6 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.0.1  netmask 255.255.255.0  broadcast 192.168.0.255
            inet6 fe80::a00:27ff:fe2a:a271  prefixlen 64  scopeid 0x20<link>
            ether 08:00:27:2a:a2:71  txqueuelen 1000  (Ethernet)
            RX packets 2  bytes 120 (120.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 73  bytes 10478 (10.2 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 1000  (Local Loopback)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    ```

    - 네트워크 인터페이스 docker0 확인가능. virtual ethernet bridge
        - 모든 docker container 는 docker0 interface 를 지난다.
        - bridge는 기본적으로 L2 Layer
            - [https://m.blog.naver.com/PostView.nhn?blogId=song_sec&logNo=220251173462&proxyReferer=https:%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=song_sec&logNo=220251173462&proxyReferer=https:%2F%2Fwww.google.com%2F)

        ![round%20robin%20%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%85%E1%85%A9%E1%84%83%E1%85%B3%E1%84%87%E1%85%A2%E1%86%AF%E1%84%85%E1%85%A5%E1%86%AB%E1%84%89%E1%85%B3%20%E1%84%83%E1%85%AC%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%A5%E1%86%BA%20%E1%84%92%E1%85%AA%E1%86%A8%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%2090b6a1dbf1124a60843f11c5acfe9fa2/docker_network.png](round%20robin%20%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%85%E1%85%A9%E1%84%83%E1%85%B3%E1%84%87%E1%85%A2%E1%86%AF%E1%84%85%E1%85%A5%E1%86%AB%E1%84%89%E1%85%B3%20%E1%84%83%E1%85%AC%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%A5%E1%86%BA%20%E1%84%92%E1%85%AA%E1%86%A8%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%2090b6a1dbf1124a60843f11c5acfe9fa2/docker_network.png)

    - eth0은 vagrant 의 기본 네트워크 인터페이스이고, eth1이 추가로 설정한 192.168. 대역의 네트워크 인터페이스
- step2 - docker netwrok 생성

    ```bash
    # 아직 veth*이 bridge에 바인딩되지 않음
    [vagrant@10 ~]$ brctl show
    bridge name	bridge id		STP enabled	interfaces
    docker0		8000.0242b2f91cab	no

    [vagrant@10 ~]$ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    8f8156c44ea9        bridge              bridge              local
    60c216d5d2aa        host                host                local
    c11658f3ee74        none                null                local

    # docker network 생성
    [vagrant@10 ~]$ docker network create --driver bridge mybridge
    b1b2f0303a5772ff8055452594dd5944dc25e34b9f3d47b2946661a2c4fa96cf
    [vagrant@10 ~]$ docker network ls
    NETWORK ID          NAME                DRIVER              SCOPE
    8f8156c44ea9        bridge              bridge              local
    60c216d5d2aa        host                host                local
    b1b2f0303a57        mybridge            bridge              local
    c11658f3ee74        none                null                local
    [vagrant@10 ~]$ !brctl
    brctl show
    bridge name	bridge id		STP enabled	interfaces
    br-b1b2f0303a57		8000.02423bb5159d	no
    docker0		8000.0242b2f91cab	no
    ```

- step3 - docker container 3개 생성

    ```bash
    [vagrant@10 ~]$ docker run -itd --name network_alias_container1 --net mybridge --net-alias alicek106 ubuntu:14.04
    a3a1948230f41bdabe9cd8310b2e88865051d37be961118568d2c5ca30430197
    [vagrant@10 ~]$ docker run -itd --name network_alias_container2 --net mybridge --net-alias alicek106 ubuntu:14.04
    a3a1948230f41bdabe9cd8310b2e88865051d37be961118568d2c5ca30430197
    [vagrant@10 ~]$ docker run -itd --name network_alias_container3 --net mybridge --net-alias alicek106 ubuntu:14.04
    a07a6cc1c03e6c26f39c82f1f33c8fb33dd344ab214dcce9990ef8a6d67900b0
    [vagrant@10 ~]$ docker container ls
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    a07a6cc1c03e        ubuntu:14.04        "/bin/bash"         6 seconds ago       Up 4 seconds                            network_alias_container3
    a3a1948230f4        ubuntu:14.04        "/bin/bash"         10 seconds ago      Up 8 seconds                            network_alias_container2
    2333ce03ab26        ubuntu:14.04        "/bin/bash"         39 seconds ago      Up 37 seconds                           network_alias_container1

    [vagrant@10 ~]$ docker run -itd --name network_alias_ping --net mybridge ubuntu:14.04
    ad933be621d1a40c88c6122733727004d5a57b0af8509ea2b59b44d566593025
    [vagrant@10 ~]$ docker exec -it --name network_alias_ping /bin/bash
    unknown flag: --name
    See 'docker exec --help'.
    [vagrant@10 ~]$ docker exec -it network_alias_ping /bin/bash
    root@ad933be621d1:/# ping -c 1 alicek106
    PING alicek106 (172.18.0.4) 56(84) bytes of data.
    64 bytes from network_alias_container3.mybridge (172.18.0.4): icmp_seq=1 ttl=64 time=0.054 ms

    --- alicek106 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.054/0.054/0.054/0.000 ms
    root@ad933be621d1:/# ping -c 1 alicek106
    PING alicek106 (172.18.0.4) 56(84) bytes of data.
    64 bytes from network_alias_container3.mybridge (172.18.0.4): icmp_seq=1 ttl=64 time=0.056 ms

    --- alicek106 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.056/0.056/0.056/0.000 ms
    root@ad933be621d1:/# ping -c 1 alicek106
    PING alicek106 (172.18.0.4) 56(84) bytes of data.
    64 bytes from network_alias_container3.mybridge (172.18.0.4): icmp_seq=1 ttl=64 time=0.122 ms

    --- alicek106 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.122/0.122/0.122/0.000 ms
    root@ad933be621d1:/# ping -c 1 alicek106
    PING alicek106 (172.18.0.2) 56(84) bytes of data.
    64 bytes from network_alias_container1.mybridge (172.18.0.2): icmp_seq=1 ttl=64 time=0.124 ms

    --- alicek106 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.124/0.124/0.124/0.000 ms

    bridge name	bridge id		STP enabled	interfaces
    br-b1b2f0303a57		8000.02423bb5159d	no		veth0684686
    							vetha93bfeb
    							vethbffe890
    							vethdb4ff7f
    docker0		8000.0242b2f91cab	no
    ```

    - 보면 알겠지만 정확히 라운드로빈 되지는 않음

### 결론

- docker 호스트 머신에는 기본적으로 docker0 network interface가 생김
    - bridge는 L2. 소프트웨어적으로 mac address를 구분해주는 장치
- —net 옵션으로 다른 네트워크를 사용가능하다. 이 경우에는 docker0로 바인딩되지 않는다
- brctl 은 bridge control
- docker bridge 는 정확히 라운드로빈 되지 않는다. 애초에 로드밸런스 기능으로 bridge를 사용하진 않을것 같다