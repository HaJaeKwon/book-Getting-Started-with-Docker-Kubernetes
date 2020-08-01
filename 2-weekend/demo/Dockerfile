FROM java:8

EXPOSE 9090

ARG JAR_FILE=build/libs/demo.jar

ADD ${JAR_FILE} demo.jar

ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/demo.jar"]
