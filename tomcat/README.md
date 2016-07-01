# Tomcat

##Prerequisites
* installed JDK, else run `sudo apt-get install default-jdk`
* sudo rights to the machine

##Installation
1. Setup your installation directory
    ```
    mkdir $CATALINA_HOME
    cd $CATALINA_HOME
    ```
	where `CATALINA_HOME=~/opt/tomcat` or your preferred installation directory

1. Download and unpack the Tomcat8 tarball. Checkout Tomcat [downloads](http://tomcat.apache.org/download-80.cgi) for other versions.
    ```
    wget http://mirror.softaculous.com/apache/tomcat/tomcat-8/v8.5.3/bin/apache-tomcat-8.5.3.tar.gz
    tar xvf apache-tomcat-8*tar.gz -C $CATALINA_HOME --strip-components=1
    ```

1.  Create a `tomcat` user and group
    ```
    sudo groupadd tomcat
    sudo useradd -s /bin/false -g tomcat -d $CATALINA_HOME tomcat
    ```

1.  Make `tomcat` owner of the `conf/, logs/, temp/, work/` subdirectories
    ```
    cd conf/ && sudo chmod g+r * && cd ..
    dirs=$(ls | grep "conf\|logs\|temp\|work")
    for dir in ${dirs}; do sudo chgrp -R tomcat $dir; done
    sudo chown -R tomcat conf/ logs/ temp/ work/
    sudo chmod g+rwx conf
    ```

1. Configure and make tomcat
    ```
    JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    CATALINA_HOME=~/opt/tomcat
    cd $CATALINA_HOME/bin
    tar xvfz commons-daemon-native.tar.gz
    cd commons-daemon-1.0.x-native-src/unix
    ./configure --with-java=$JAVA_HOME
    make
    cp jsvc ../..
    cd ../..
    ```
    _Note: Adjust the values for `$JAVA_HOME` and `$CATALINA_HOME` according to your installation._

    **Recommended:** add an `admin` user with password to access the Manager Webapp
    and a separate `deploy` user for the manager-script role.[[1](http://stackoverflow.com/questions/32230962/mvn-tomcat7deploy-cannot-invoke-tomcat-manager-broken-pipe)]
    ```
    sudo vi $CATALINA_HOME/conf/tomcat-users.xml
    ...
    <tomcat-users xmlns="http://tomcat.apache.org/xml"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
                  version="1.0">
       <role rolename="manager-gui"/>
       <user username="admin" password="password" roles="manager-gui,admin-gui"/>
       <role rolename="manager-script"/>
       <user username="deploy" password="password" roles="manager-script"/>
    </tomcat-users>
    ```

1. Start Tomcat
    ```
    CATALINA_BASE=$CATALINA_HOME
    cd $CATALINA_HOME
    sudo ./bin/jsvc \
        -classpath $CATALINA_HOME/bin/bootstrap.jar:$CATALINA_HOME/bin/tomcat-juli.jar \
        -outfile $CATALINA_BASE/logs/catalina.out \
        -errfile $CATALINA_BASE/logs/catalina.err \
        -Dcatalina.home=$CATALINA_HOME \
        -Dcatalina.base=$CATALINA_BASE \
        -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager \
        -Djava.util.logging.config.file=$CATALINA_BASE/conf/logging.properties \
        org.apache.catalina.startup.Bootstrap
    ```
    _Note: you should be able to see the Tomcat landing page at http://localhost:8080 on your browser._

    Follow the instructions to [deploy a new app](http://tomcat.apache.org/tomcat-8.5-doc/manager-howto.html#Deploy_A_New_Application_from_a_Local_Path).