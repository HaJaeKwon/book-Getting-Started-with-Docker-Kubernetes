FROM openjdk:8-jre-alpine

EXPOSE 9091

ARG JAR_FILE=./build/libs/demo.jar

ADD ${JAR_FILE} demo.jar

HEALTHCHECK --interval=5s \
            --timeout=5s \
            CMD curl -f http://127.0.0.1:9091/ping || exit 1

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/demo.jar"]
