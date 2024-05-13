# docker build . -t susukkekki/bookshop-cart:240406-1517
# docker run -p 8083:8083 susukkekki/bookshop-cart:240406-1517
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.9

ARG JAVA_PACKAGE=java-17-openjdk-headless
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en'
# Install java and nginx
RUN microdnf install curl ca-certificates ${JAVA_PACKAGE} nginx \
    && microdnf update \
    && microdnf clean all \
    && mkdir /deployments \
    && echo "securerandom.source=file:/dev/urandom" >> /etc/alternatives/jre/conf/security/java.security

COPY ./target/cart-0.0.5-SNAPSHOT.jar /deployments/cart.jar

EXPOSE 8083

CMD [ "java", "-jar", "/deployments/cart.jar"]
