# docker build . -t susukkekki/bookshop-checkout:240406-1533
# docker run -p 3000:3000 susukkekki/bookshop-checkout:240406-1533
FROM registry.access.redhat.com/ubi8/ubi-minimal:8.9

ARG JAVA_PACKAGE=java-17-openjdk-headless
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en'
# Install java and nginx
RUN microdnf install curl ca-certificates ${JAVA_PACKAGE} nginx \
    && microdnf update \
    && microdnf clean all \
    && mkdir /deployments \
    && echo "securerandom.source=file:/dev/urandom" >> /etc/alternatives/jre/conf/security/java.security

COPY ./deployment/checkout.conf /etc/nginx/conf.d/default.conf
COPY ./dist /usr/share/nginx/html

EXPOSE 3000

# ENTRYPOINT [ "nginx -g 'daemon off;'" ]
CMD nginx -g "daemon off;"
