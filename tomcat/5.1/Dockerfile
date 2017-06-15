FROM amazonlinux:2017.03.0.20170401

MAINTAINER OmarAmaro <amaroom@gmail.com>
# reference https://github.com/lucee/lucee-dockerfiles

ENV LUCEE_JAVA_OPTS "-Xms256m -Xmx512m"

# Update to the latest for this distro
RUN yum -y update && \
	yum -y install tomcat8 tomcat-native && \
	rm -rf /var/cache/yum /var/lib/yum && yum clean all

COPY lucee-5.1.4.19.jar /usr/share/tomcat8/lucee/

# Server context folders are in /opt/lucee/*
COPY catalina.properties server.xml web.xml tomcat8.conf /usr/share/tomcat8/conf/

# Create Lucee configs
COPY lucee-server.xml /opt/lucee/server/lucee-server/context/lucee-server.xml
COPY lucee-web.xml.cfm /opt/lucee/web/lucee-web.xml.cfm

# Provide test page
RUN mkdir -p /var/www
COPY index.cfm /var/www/
ONBUILD RUN rm -rf /var/www/*

# lucee first time startup; explodes lucee and installs bundles/extensions
RUN /usr/sbin/tomcat8 start && \
    while [ ! -f "/opt/lucee/web/logs/application.log" ] ; do sleep 2; done && \
    /usr/sbin/tomcat8 stop

# Replace the Trusted SSL Certificates packaged with Lucee with those from AmzLinux
#   ca-certificates package from the OS is the most recent authority
RUN cp -f /etc/pki/ca-trust/extracted/java/cacerts /opt/lucee/server/lucee-server/context/security/cacerts

# Expose HTTP and HTTPS ports
EXPOSE 8080
