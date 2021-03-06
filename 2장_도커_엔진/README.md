# 2장 정리 - 하재권

# 도커 엔진

- docker container
    - exec -i -t
        - 상호입출력이 가능한 상태로 docker container 내부로 접속
        - 새로운 가상 모니터가 생긴다. tty가 새로 생김
    - run -d
        - 내부에서 데몬 프로세스를 띄우고, 새로운 shell을 생성하진 않는다
- docker volume
- docker network
    - bridge
        - [round robin 으로 로드밸런스 되는 것 확인하는 실습](2%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20-%20%E1%84%92%E1%85%A1%E1%84%8C%E1%85%A2%E1%84%80%E1%85%AF%E1%86%AB%20e9a4b3c1af4141b5a15fb223b3ff0e9a/round%20robin%20%E1%84%8B%E1%85%B3%E1%84%85%E1%85%A9%20%E1%84%85%E1%85%A9%E1%84%83%E1%85%B3%E1%84%87%E1%85%A2%E1%86%AF%E1%84%85%E1%85%A5%E1%86%AB%E1%84%89%E1%85%B3%20%E1%84%83%E1%85%AC%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%80%E1%85%A5%E1%86%BA%20%E1%84%92%E1%85%AA%E1%86%A8%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%92%E1%85%A1%E1%84%82%E1%85%B3%2090b6a1dbf1124a60843f11c5acfe9fa2.md)
        - 라운드로빈 DNS 단점
            - 일반적으로는 healthcheck 기능이 없기에 HA 구성이 어렵다
            - 캐싱 주기를 짧게 가져가도 그 시간만큼은 HA 구성이 안되기에 주기를 짧게 가져가기도 한다
                - dig [blog.naver.com](http://blog.naver.com/) 등으로 캐싱 주기 확인 가능
            - GSLB + load balancing 구성을 하기도 한다
            - AWS Route53, Dyn DNS
            - 클라이언트가 DNS에 의해 다른 서버 IP를 할당받으면 기존 세션이 끊어질 수 있다
    - host
    - none
    - container
    - macVLAN
    - overlay
- container logging
- container 자원할당제한
    - memory, cpu, block io
- docker image
    - docker images
    - docker inspect
    - docker history
    - docker save/load or export/import
        - ?
    - docker push
- docker file
    - docker build
        - —no-cache
        - —cache-from
    - docker run -d -P
        - -P: EXPOSE 설정해둔 외부 open port 들을 모두 열겠다
    - .dockerignore
    - multi stage build
    - ENV, VOLUME, ARG, USER
        - container 내부에선 기본적으로 root기에 보안 측면에서 최종적으로 app 배포시에는 새로운 사용자를 지정하는걸 추천
    - Onbuild, Stopsignal, Healthcheck, Shell
        - Healthcheck 기능으로 앱의 healthcheck 를 확인하는...용도로 자주 쓸까? 오케스트레이션 툴에선 또 다른 방식이 있을듯
    - ADD, COPY
        - ADD: 외부 URL 설정 가능. image build 시에 어떤 파일이 들어올지 예측 불가능. 비추천
        - COPY: 로컬 file을 복사
    - ENTRYPOINT, CMD
        - 둘 중 하나는 입력되어야함. JSON 포맷으로 입력되지 않으면 /bin/sh -c 로 동작하니 주의
        - ENTRYPOINT: 보통은 script를 등록, 둘다 입력하면 CMD는 ENTRYPOINT의 파라미터로 동작한다
        - CMD: 수행할 명령 지정. 보통 /bin/bash
- Dockerfile best practice
    - 역슬래쉬(\)
        - apt-get install 시에 가독성 향상
    - RUN + && 으로 docker layer 줄이기
        - 도중에 생기는 불필요한 파일들, 마지막엔 삭제되는 파일들이 있을때 docker image size를 줄이는데 유용
        - 이미지간에 공통으로 사용하는 라이브러리가 있다면 parent image 를 만들어두면 유용
- 도커 데몬
    - 클라이언트: /usr/bin/docker
    - 서버: /usr/bin/dockerd

- [동일 docker container 에서 Remote API 사용하기 실습](2%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20-%20%E1%84%92%E1%85%A1%E1%84%8C%E1%85%A2%E1%84%80%E1%85%AF%E1%86%AB%20e9a4b3c1af4141b5a15fb223b3ff0e9a/%E1%84%83%E1%85%A9%E1%86%BC%E1%84%8B%E1%85%B5%E1%86%AF%20docker%20container%20%E1%84%8B%E1%85%A6%E1%84%89%E1%85%A5%20Remote%20API%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5%20%208ccf21bc2e4b4507aec7a9e53107ed49.md)

- [SSL 적용된 Remote API 호출](2%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%20-%20%E1%84%92%E1%85%A1%E1%84%8C%E1%85%A2%E1%84%80%E1%85%AF%E1%86%AB%20e9a4b3c1af4141b5a15fb223b3ff0e9a/SSL%20%E1%84%8C%E1%85%A5%E1%86%A8%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%83%E1%85%AC%E1%86%AB%20Remote%20API%20%E1%84%92%E1%85%A9%E1%84%8E%E1%85%AE%E1%86%AF%2034041fd5088e47b3b076928930e8d0ca.md)
