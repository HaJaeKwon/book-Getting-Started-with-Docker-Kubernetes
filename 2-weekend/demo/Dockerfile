FROM java:8

EXPOSE 9090

ARG JAR_FILE=/var/lib/jenkins/workspace/2-weekend-demo/2-weekend/demo/build/libs/demo.jar

ADD ${JAR_FILE} demo.jar

HEALTHCHECK --interval=5s \
            --timeout=5s \
            CMD curl -f http://127.0.0.1:9090/ping || exit 1

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/demo.jar"]
