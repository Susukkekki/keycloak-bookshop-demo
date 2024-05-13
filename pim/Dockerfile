# docker build . -t susukkekki/bookshop-pim:240406-1544
# docker run -p 8082:8082 susukkekki/bookshop-pim:240406-1544
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.9

ARG JAVA_PACKAGE=java-17-openjdk-headless
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en'
# Install java and nginx
RUN microdnf install curl ca-certificates ${JAVA_PACKAGE} nginx \
    && microdnf update \
    && microdnf clean all \
    && mkdir /deployments \
    && echo "securerandom.source=file:/dev/urandom" >> /etc/alternatives/jre/conf/security/java.security

COPY ./target/pim-0.0.5-SNAPSHOT-runner.jar /deployments/pim.jar

EXPOSE 8082

ENV QUARKUS_JAVA_OPTS="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
CMD java -jar /deployments/pim.jar $QUARKUS_JAVA_OPTS
# CMD [ "/deployments/entrypoint.sh" ]
