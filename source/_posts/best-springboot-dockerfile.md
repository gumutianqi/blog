---
title: Spring-Boot项目中 Dockerfile 最佳实践
date: 2018-06-08 14:01:17
categories: Docker 
tags: 
 - SpringBoot
 - Docker
permalink: best-springboot-dockerfile
---

> 原创内容

经过长期使用 Spring-Boot 和 Docker 的项目实践，现在将我们正在使用的 Dockerfile 分享出来供大家参考；

{% asset_img spring-boot-docker.png [Spring-Boot-Docker] %}


<!-- more -->

---

#  最佳 Dockerfile

## 废话不多说，直接上

```
# Version 1.0.0
# Data 2018-06-16
FROM weteam/java:jdk8

LABEL maintainer="larrykoo@126.com"

RUN mvn clean package

COPY dist/$APP_NAME-*.jar app.jar

ENV APP_NAME="<Your-App-Name>"
ENV APP_VERSION="1.0.0" \
    APP_ENV="dev" \
    APP_CONFIG_ENABLED="true" \
    APP_CONFIG_URI="http://config-server:8080" \
    APP_CONFIG_USERNAME="root" \
    APP_CONFIG_PASSWORD="root" \
    APP_CONFIG_NAME="app" \
    APP_CONFIG_PROFILE="dev" \
    APP_CONFIG_LABEL="dev" \
    JVM_LOG_HOME="/logs/$APP_NAME" \
    JAVA_TIMEZONE="Asia/Shanghai" \
    JAVA_MEM_OPTS="-XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=256m -Xss256K -XX:SurvivorRatio=8" \
    JAVA_JVM_OPTS="-Xms512m -Xmx1024m -Xmn128m"
ENV JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom \
        -Duser.timezone=$JAVA_TIMEZONE \
        -Djava.awt.headless=true \
        -Djava.net.preferIPv4Stack=true \
        -XX:+PrintGCDetails \
        -XX:+PrintGCApplicationStoppedTime \
        -Xloggc:$JVM_LOG_HOME/gc.log \
        -XX:+HeapDumpOnOutOfMemoryError \
        -XX:HeapDumpPath=$JVM_LOG_HOME/heapdump.hprof "

VOLUME /logs

EXPOSE 8080

ENTRYPOINT exec java -server $JAVA_JVM_OPTS $JAVA_MEM_OPTS $JAVA_OPTS -jar /app.jar \
    --spring.profiles.active=$APP_ENV

```

## Dockerfile 解析

// TODO




