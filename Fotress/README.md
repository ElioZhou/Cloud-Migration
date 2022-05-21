# Tomcatdocker
A custom build for to install Fortress-WEB and Fortress-REST on Docker - Be sure to **edit fortress.properties!**
The website is

http://54.145.198.173:8080/fortress-web-2.0.7/

`sudo docker build -t fortress:v1.0 .`

`sudo docker run -itd --name=fortress -p 8080:8080 fortress:v1.1`


# Step1: Config ApacheDS using Docker
-------------------------------------------------------------------------------
## SECTION 1. Prerequisites

Minimum software requirements:
 * Linux Machine with docker-engine
 * Java SDK >= 8
 * Apache Maven >= 3

___________________________________________________________________________________
## SECTION 2. ApacheDS Docker Image

1. Get the Apache Fortress Core package:

a. clone git latest: 

```bash
git clone https://gitbox.apache.org/repos/asf/directory-fortress-core.git
cd directory-fortress-core
```

b. or clone git release: 

```bash
git clone --branch 2.0.7  https://gitbox.apache.org/repos/asf/directory-fortress-core.git
cd directory-fortress-core
```

c. or download source package from Apache: 

```bash
wget https://www.apache.org/dist/directory/fortress/dist/2.0.7/fortress-core-2.0.7-source-release.zip
unzip fortress-core-2.0.7-source-release.zip
cd fortress-core-2.0.7
```

2. Prepare the ApacheDS docker image

a. build the ApacheDS docker image (trailing dot matters):

```bash
docker build -t apachedirectory/apacheds-for-apache-fortress-tests -f src/docker/apacheds-for-apache-fortress-tests/Dockerfile .
```

b. or pull the latest prebuilt image from Dockerhub:

```bash
docker pull apachedirectory/apacheds-for-apache-fortress-tests
```

3. Run the ApacheDS docker image:

```bash
docker run --name=apacheds -d -p 10389:10389 -P apachedirectory/apacheds-for-apache-fortress-tests
```

 * Here we're mapping the internal port for running image to what the host machine uses, w/ apacheds' default of 10389
 * Setting name=apacheds for managing image.
 
4. Verify the image started successfully:

```bash
root@localhost:/opt/fortress/directory-fortress-core# docker ps
CONTAINER ID   IMAGE                                                COMMAND                  CREATED          STATUS          PORTS                                           NAMES
7df7ab967737   apachedirectory/apacheds-for-apache-fortress-tests   "sh -c 'service ${AP…"   10 minutes ago   Up 10 minutes   0.0.0.0:10389->10389/tcp, :::10389->10389/tcp   apacheds
```

5. Manage ApacheDS Image:

a. start 

```bash
docker run --name=apacheds -d -p 10389:10389 -P apachedirectory/apacheds-for-apache-fortress-tests
```

b. stop 

```bash
docker stop apacheds
```

c. remove 

```bash
docker rm apacheds
```

d. view the logs 

```bash
docker logs apacheds
```

e. inspect 

```bash
docker inspect apacheds
```

f. connect via bash 

```bash
docker exec -it apacheds bash
```

___________________________________________________________________________________
## SECTION 3. Apache Fortress Setup

1. Prepare your terminal for execution of maven commands.

```bash
#!/bin/bash
export M2_HOME=...
export JAVA_HOME=...
export PATH=$PATH:$M2_HOME/bin
```

2. Prepare the package:

```bash
cp build.properties.example build.properties
```

 * Seeds the fortress properties with defaults for ApacheDS usage.
 * [build.properties.example](build.properties.example) contains the default config for the apacheds docker image.
 * Learn how the fortress config subsystem works: [README-CONFIG](README-CONFIG.md).

3. Run the maven install to build fortress and initialize config settings:

```bash
mvn clean install
```
___________________________________________________________________________________
## SECTION 4. Apache Fortress Core Integration Test

1. From fortress core base folder, enter the following commands:

   a. To use the config from earlier:
```bash
mvn install -Dload.file=./ldap/setup/refreshLDAPData.xml
```

2. Next, enter the following command:

```bash
mvn -Dtest=FortressJUnitTest test
```

 More about this step: 
  * Tests the APIs against your LDAP server.

3. Verify the tests worked:

```bash
NUMBER OF ADDS: 1146
NUMBER OF BINDS: 1130
NUMBER OF DELETES: 0
NUMBER OF MODS: 4247
NUMBER OF READS: 20176
NUMBER OF SEARCHES: 7045
Tests run: 128, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 335.906 sec - in org.apache.directory.fortress.core.impl.FortressJUnitTest

Results :

Tests run: 128, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- maven-antrun-plugin:1.8:run (fortress-load) @ fortress-core ---
[INFO] Executing tasks

fortress-load:
[INFO] Executed tasks
[INFO] 
[INFO] --- maven-antrun-plugin:1.8:run (fortress-load-debug) @ fortress-core ---
[INFO] Executing tasks

fortress-load-debug:
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  05:58 min
[INFO] Finished at: 2022-01-02T15:23:11Z
[INFO] ------------------------------------------------------------------------
```

4. Rerun the tests to verify teardown APIs work:

```bash
mvn -Dtest=FortressJUnitTest test
```

5. Verify that worked also:

```bash
NUMBER OF ADDS: 993
NUMBER OF BINDS: 1130
NUMBER OF DELETES: 1355
NUMBER OF MODS: 8389
NUMBER OF READS: 27331
NUMBER OF SEARCHES: 9582
Tests run: 162, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 378.525 sec - in org.apache.directory.fortress.core.impl.FortressJUnitTest

Results :

Tests run: 162, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- maven-antrun-plugin:1.8:run (fortress-load) @ fortress-core ---
[INFO] Executing tasks

fortress-load:
[INFO] Executed tasks
[INFO] 
[INFO] --- maven-antrun-plugin:1.8:run (fortress-load-debug) @ fortress-core ---
[INFO] Executing tasks

fortress-load-debug:
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  06:43 min
[INFO] Finished at: 2022-01-02T15:30:15Z
[INFO] ------------------------------------------------------------------------
```
 * More tests ran this time vs the first time, due to teardown.

 Test Notes:
  * If tests complete without errors, Apache Fortress works with ApacheDS server running in Docker image.
  * These tests load thousands of objects into the target ldap server.
  * Warning messages are negative tests in action.

6. Optional sections in the [README](README.md) file:

 * SECTION 11. Instructions to run the Apache Fortress Command Line Interpreter (CLI).
 * SECTION 12. Instructions to run the Apache Fortress Command Console.
 * SECTION 13. Instructions to build and test the Apache Fortress samples.
 * SECTION 14. Instructions to performance test.

___________________________________________________________________________________
## SECTION 5. Docker Command Reference

Here are some common commands needed to manage the Docker image.

#### Build image

```bash
docker build -t apachedirectory/apacheds-for-apache-fortress-tests -f src/docker/apacheds-for-apache-fortress-tests/Dockerfile .
```

 * trailing dot matters

 Or just to be sure don't use cached layers:

```bash
docker build --no-cache=true -t apachedirectory/apacheds-for-apache-fortress-tests -f src/docker/apacheds-for-apache-fortress-tests/Dockerfile .
```

#### Run container

```bash
docker run --name=apacheds -d -p 10389:10389 -P apachedirectory/apacheds-for-apache-fortress-tests
```

#### Go into the container

```bash
docker exec -it apacheds bash
```

#### Restart container

```bash
docker restart apacheds
```

#### Stop and delete container

```
docker stop apacheds
docker rm apacheds
```

____________________________________________________________________________________
#### END OF README-QUICKSTART-DOCKER-APACHEDS
# Step2: Config Fortress App
___________________________________________________________________________________
## SECTION 5. Apache Tomcat Setup

During this section, you will be asked to setup Apache Tomcat 8 and prepare for usage with Apache Fortress

1. Download and prepare the package:

 ```
 wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.0.30/bin/apache-tomcat-8.0.30.tar.gz
 tar -xvf apache-tomcat-8.0.30.tar.gz
 sudo mv apache-tomcat-8.0.30 /usr/local/tomcat8
 ```
 *Change the tomcat version as neeeded - v7 and beyond are ok.*
 *For BSD variants (i.e. Mac) append /* to the folder name above on mv command.*

2. Download the fortress realm proxy jar into tomcat/lib folder:

  ```
  sudo wget https://repo.maven.apache.org/maven2/org/apache/directory/fortress/fortress-realm-proxy/2.0.7/fortress-realm-proxy-2.0.7.jar -P /usr/local/tomcat8/lib
  ```

3. Prepare tomcat fortress usage:

 ```
 sudo vi /usr/local/tomcat8/conf/tomcat-users.xml
 ```

4. Add tomcat user to deploy fortress:

 ```
 <role rolename="manager-script"/>
 <role rolename="manager-gui"/>
 <user username="tcmanager" password="m@nager123" roles="manager-script"/>
 <user username="tcmanagergui" password="m@nager123" roles="manager-gui"/>
 ```

5. Save and exit tomcat-users.xml file

6. Configure Tomcat as a service (optional)

 a. Edit the config file:

 ```
 vi /etc/init.d/tomcat
 ```

 b. Add the following:

 ```
 #!/bin/bash
 # description: Tomcat Start Stop Restart
 # processname: tomcat
 # chkconfig: 234 20 80
 CATALINA_HOME=/usr/local/tomcat8
 case $1 in
 start)
 sh $CATALINA_HOME/bin/startup.sh
 ;;
 stop)
 sh $CATALINA_HOME/bin/shutdown.sh
 ;;
 restart)
 sh $CATALINA_HOME/bin/shutdown.sh
 sh $CATALINA_HOME/bin/startup.sh
 ;;
 esac
 exit 0
 ```

 c. Add the init script to startup for run level 2, 3 and 4:

 ```
 cd /etc/init.d
 chmod 755 tomcat
 chkconfig --add tomcat
 chkconfig --level 234 tomcat on
 ```

7. Start tomcat server:

 a. If running Tomcat as a service:

 ```
 service tomcat start
 ```

 b. Else

 ```
 sudo /usr/local/tomcat8/bin/startup.sh
 ```

8.  Verify clean logs after startup:

 ```
 tail -f -n10000 /usr/local/tomcat8/logs/catalina.out
 ```

9.  Verify setup by signing onto the Tomcat Manager app with credentials userId: tcmanagergui, password: m@nager123

 ```
 http://hostname:8080/manager
 ```
___________________________________________________________________________________
## SECTION 6. Apache Fortress Rest Setup

During this section, you will be asked to setup Apache Fortress Rest Application

1. Download the package:

 a. from git:
 ```
 git clone --branch 2.0.7  https://gitbox.apache.org/repos/asf/directory-fortress-enmasse.git
 cd directory-fortress-enmasse
 ```

 b. or download package:
 ```
 wget https://www.apache.org/dist/directory/fortress/dist/2.0.7/fortress-rest-2.0.7-source-release.zip
 unzip fortress-rest-2.0.7-source-release.zip
 cd fortress-rest-2.0.7
 ```

2. Prepare:

 ```
 cp ../[FORTRESS-CORE-HOME]/config/fortress.properties src/main/resources
 ```
 *Make sure to change the host!
 *where FORTRESS-CORE-HOME is package location on your machine*

3. Build, perform fortress rest test policy load and deploy to Tomcat:

 ```
 mvn clean install -Dload.file=./src/main/resources/FortressRestServerPolicy.xml tomcat:deploy
 ```

4. Redeploy (if need be):

 ```
 mvn tomcat:redeploy
 ```

5. Smoke test:

 ```
 mvn test -Dtest=EmTest
 ```

___________________________________________________________________________________
## SECTION 7. Apache Fortress Web Setup

During this section, you will be asked to setup Apache Fortress Web Application

1. Download the package:

 a. from git:
 ```
 git clone --branch 2.0.7  https://gitbox.apache.org/repos/asf/directory-fortress-commander.git
 cd directory-fortress-commander
 ```

 b. or download package:
 ```
 wget https://www.apache.org/dist/directory/fortress/dist/2.0.7/fortress-web-2.0.7-source-release.zip
 unzip fortress-web-2.0.7-source-release.zip
 cd fortress-web-2.0.7
 ```

2. Prepare:

 ```
 cp ../[FORTRESS-CORE-HOME]/config/fortress.properties src/main/resources
 ```
 *Make sure to change the host!
 *where FORTRESS-CORE-HOME is package location on your machine*

3. Build, perform fortress web test policy load and deploy to Tomcat:

 ```
 mvn clean install -Dload.file=./src/main/resources/FortressWebDemoUsers.xml tomcat:deploy
 ```

4. Redeploy (if need be):

 ```
 mvn tomcat:redeploy
 ```

5. Open browser and test (creds: test/password):

 ```
 http://hostname:8080/fortress-web
 ```

6. Click on the links, to pull up various views on the data stored in the directory.

7. Run the Selenium Web driver integration tests with Firefox (default):

 ```
 mvn test -Dtest=FortressWebSeleniumITCase
 ```

8. Run the tests using Chrome:

 ```
 mvn test -Dtest=FortressWebSeleniumITCase -Dweb.driver=chrome
 ```

 Note: The Selenium tests require that:
 * Either Firefox or Chrome installed to target machine.
 * **FORTRESS_CORE_HOME**/*FortressJUnitTest* successfully run.  This will load some test data to grind on.
 * [FortressWebDemoUsers](./src/main/resources/FortressWebDemoUsers.xml) policy loaded into target LDAP server.

___________________________________________________________________________________
#### END OF README-QUICKSTART-APACHEDS
