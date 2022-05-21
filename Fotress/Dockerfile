FROM tomcat:8.5.20

# Maintainer
MAINTAINER yz7787@nyu.edu

# Copy tomcat-users.xml
ADD tomcat-users.xml /usr/local/tomcat/conf/

# Copy catalina.sh with JAVA_OPTS
ADD catalina.sh /usr/local/tomcat/bin/

# Copy JAR
ADD fortress-realm-proxy-2.0.7.jar /usr/local/tomcat/lib/

# Copy images to tomcat path
ADD fortress-rest-2.0.7.war /usr/local/tomcat/webapps/
ADD fortress-web-2.0.7.war /usr/local/tomcat/webapps/

EXPOSE 8080

# Start the Service
RUN chmod +x /usr/local/tomcat/bin/catalina.sh
CMD ["/usr/local/tomcat/bin/catalina.sh", "run"]
