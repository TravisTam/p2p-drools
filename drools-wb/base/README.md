Drools Workbench Docker image
==============================

JBoss Drools Workbench [Docker](http://docker.io/) image.

Table of contents
------------------

* Introduction
* Usage
* Users and roles
* Logging
* Extending this image
* Experimenting
* Notes

Introduction
------------

The image contains:       
        
* JBoss Wildfly 8.1.0.Final             
* Drools Workbench 6.2.0.Final            

This image provides the Drools Workbench but it's intended to be extended so you can add  your custom configurations such as users, roles, etc.                 

If you don't want to extend this image and you just want to try Drools Workbench take a look at the Docker image `jboss/drools-workbench-showcase:6.2.0.Final`, it contains default users and roles and allows using the application with no custom configurations.                   


Usage
-----

To run a container:
    
    docker run -P -d --name drools-workbench jboss/drools-workbench:6.2.0.Final

Once container and web applications started, you can navigate to it:              

**Using local host binding port**

If you have run the container using `-P` flag in the `docker` command, the port `8080` has been bind to an available port on your host.                 

So you have to discover your host's bind port, that can be done by running the command:          

    docker ps -a

Example of the above command response:                   

    CONTAINER ID        IMAGE                                COMMAND                CREATED              STATUS              PORTS                                              NAMES
    2a55fbe771c0        jboss/drools-workbench:6.2.0.Final   ./standalone.sh -b 0   About a minute ago   Up About a minute   0.0.0.0:49159->8080/tcp, 0.0.0.0:49160->9990/tcp   drools-workbench      

As you can see, the bind port to use for container's port `8080` is `49159`, so you can navigate to:

    http://localhost:49159/drools-wb

**No bind port for localhost**

In case you run the container without using `-P` flag in the `docker` command, you can navigate to the application at:

    http://<container_ip_address>:8080/drools-wb
    
You can discover the IP address of your running container by:

    docker inspect --format '{{ .NetworkSettings.IPAddress }}' drools-workbench


Users and roles
----------------

The application have no users or roles configured, so you cannot not access it by default,               

In order to use it, at least you have to create an application user in JBoss Wildfly with role `admin`.                  

If you are looking for a Drools Workbench image that does not require to add custom configurations, try our `jboss/drools-workbench-showcase:6.2.0.Final` Docker image.                   

If you want to create your custom configuration and users, role, etc, you can take a look at section `Extending this image`    


Logging
-------

You can see all logs generated by the `standalone` binary running:

    docker logs [-f] <container_id>
    
You can attach the container by running:

    docker attach <container_id>

The Drools Workbench web application logs can be found inside the container at path:

    /opt/jboss/wildfly/standalone/log/server.log

    Example:
    sudo nsenter -t $(docker inspect --format '{{ .State.Pid }}' $(docker ps -lq)) -m -u -i -n -p -w
    -bash-4.2# tail -f /opt/jboss/wildfly/standalone/log/server.log

Extending this image
--------------------

You can extend this image and add your custom layers in order to add custom configurations, users, roles, etc.                  
 
In order to extend this image, your Dockerfile must inherit from:

    FROM jboss/drools-workbench:6.2.0.Final
    
**Configuring Wildfly**

* The Wildfly configuration files are located at `/opt/jboss/wildfly/standalone/configuration`                   
* In this file you can modify all Wildfly's subsystem configurations                           
* Drools Workbench requires running Wildfly using `full` profile, so custom modifications should be done in `standalone-full.xml` configuration file                      
* It's recommended if there are several modifications to do, to create a copy of the configuration file, rename it and use it to start Wildfly. If you do that, your Dockerfile must run Wildfly using this configuration file, so your `CMD` command should be as:                         
    
        CMD ["./standalone.sh", "-b", "0.0.0.0", "--server-config=your-standalone-full.xml"]

**Users and roles**

* By default this image does not provide users and roles for Drools Workbench                      

* The available roles for Drools Workbench examples are:                       

        ROLE        DESCRIPTION
        *************************************************
        admin       The administrator
        analyst     The analyst
        developer   The developer
        manager     The manager
        user        The end user
        kiemgmt     KIE management user
        Accounting  Accounting role
        PM          Project manager role
        HR          Human resources role
        sales       Sales role
        IT          IT role

These are the steps to create your custom users and roles by using realm files in Widlfly:                  

1.- Create a realm properties file for users and deploy it in `/opt/jboss/wildfly/standalone/configuration`:                 
 
        drools-users.properties
        ---------------------
        admin=admin
        analyst=analyst
        developer=developer
        manager=manager
        user=user
        
2.- Create a realm properties file for roles and deploy it in `/opt/jboss/wildfly/standalone/configuration`:                 
 
        drools-roles.properties
        ---------------------
        admin=admin
        analyst=analyst
        developer=developer
        manager=manager
        user=user

3.- Modify your `standalone-full.xml` in order to:                
        
3.1 - In the `management` section, modify default the security-realm for the `ApplicationRealm` as:                   

        <security-realm name="ApplicationRealm">
              <authentication>
                <local default-user="$local" allowed-users="*" skip-group-loading="true"/>
                <properties path="drools-users.properties" relative-to="jboss.server.config.dir"/>
              </authentication>
              <authorization>
                <properties path="drools-roles.properties" relative-to="jboss.server.config.dir"/>
              </authorization>
          </security-realm>
          
3.2 - In the `security` subsystem, modify default the `other` security-domain for as:                         

        <security-domain name="other" cache-type="default">
          <authentication>
            <login-module code="UsersRoles" flag="required">
              <module-option name="usersProperties" value="${jboss.server.config.dir}/drools-users.properties"/>
              <module-option name="rolesProperties" value="${jboss.server.config.dir}/drools-roles.properties"/>
            </login-module>
          </authentication>
        </security-domain>

You can find an example by looking at the Dockerfile for `jboss/drools-workbench-showcase:6.2.0.Final` image.      

Experimenting
-------------

To spin up a shell in one of the containers try:

    docker run -t -i -P jboss/drools-workbench:6.2.0.Final /bin/bash

You can then noodle around the container and run stuff & look at files etc.

You can run the Drools Workbench web application by running command:

    /opt/jboss/wildfly/bin/standalone.sh -b 0.0.0.0 --server-config=standalone-full.xml


Notes
-----

* Drools Workbench version is `6.2.0.Final`               
* Drools Workbench requires running JBoss Wildfly using `full` profile                        
* No users or roles are configured by default               
* No support for clustering                
* Use of embedded H2 database server by default               
* No support for Wildfly domain mode, just standalone mode                    
* The context path for Drools Workbench web application is `drools-wb`                  
