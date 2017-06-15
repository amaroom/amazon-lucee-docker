FROM amazonlinux:2017.03.0.20170401

MAINTAINER OmarAmaro <amaroom@gmail.com>
# reference https://github.com/lucee/lucee-dockerfiles

ENV LUCEE_JAVA_OPTS "-Xms256m -Xmx512m"

# Update to the latest for this distro
# we add the EPEL repo so that we can install supervisord
RUN yum -y update && \
	yum -y install tomcat8 tomcat-native nginx epel-release && \
	yum-config-manager --enable epel && \
	yum -y install supervisor && \
	rm -rf /var/cache/yum /var/lib/yum && yum clean all

COPY lucee-5.1.4.19.jar /usr/share/tomcat8/lucee/

# Server context folders are in /opt/lucee/*
COPY catalina.properties server.xml web.xml tomcat8.conf /usr/share/tomcat8/conf/

# Create Lucee configs
COPY lucee-server.xml /opt/lucee/server/lucee-server/context/lucee-server.xml
COPY lucee-web.xml.cfm /opt/lucee/web/lucee-web.xml.cfm

# supervisord only plays well with python2.6
COPY supervisord /usr/bin/
COPY supervisord.conf /etc/
RUN chmod a+x /usr/bin/supervisord

# tomcat8 startup wrapper for supervisord
COPY supervisor-tomcat-wrapper.sh /usr/sbin/
RUN chmod a+x /usr/sbin/supervisor-tomcat-wrapper.sh

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

# Copy default nginx config files
COPY nginx.conf /etc/nginx/
COPY default.conf /etc/nginx/conf.d/

# Expose HTTP and HTTPS ports
EXPOSE 80 443

# Engage
CMD ["supervisord", "-c", "/etc/supervisord.conf"]
