# docker build . -t susukkekki/bookshop-shop:240406-1548
# docker run -p 8081:8081 susukkekki/bookshop-shop:240406-1548
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.9

ARG JAVA_PACKAGE=java-17-openjdk-headless
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en'
# Install java and nginx
RUN microdnf install curl ca-certificates ${JAVA_PACKAGE} nginx \
    && microdnf update \
    && microdnf clean all \
    && mkdir /deployments \
    && echo "securerandom.source=file:/dev/urandom" >> /etc/alternatives/jre/conf/security/java.security

COPY ./target/shop-0.0.5-SNAPSHOT-runner.jar /deployments/shop.jar

EXPOSE 8081

ENV QUARKUS_JAVA_OPTS="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
CMD java -jar /deployments/shop.jar $QUARKUS_JAVA_OPTS
