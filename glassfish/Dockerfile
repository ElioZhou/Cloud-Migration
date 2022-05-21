FROM store/oracle/serverjre:8

# Maintainer
MAINTAINER pw2271@nyu.edu

COPY /adf-essentials /glassfish5/glassfish/domains/domain1/lib

# Ports being exposed
EXPOSE 4848 8080 8181

# Start asadmin console and the domain
CMD ["asadmin", "start-domain", "-v"]
