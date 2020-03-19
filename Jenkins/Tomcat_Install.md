# Tomcat installation on CentOS 7

### Prerequisites
1. Installed VirualBox
1. Installed Vagrant

## Prepare Web Server Vagrant Project
1. Create Project Folder
2. Init Vagrantfile  
`vagrant init`
3. Copy Vagrantfile Content
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
    config.vm.define 'web_server' do |js|
        js.vm.box = 'centos/7'
        js.vm.hostname = 'web-server'
        js.vm.provider 'virtualbox' do |vb|
            vb.name = 'web_server'
            vb.memory = '2048'
            vb.cpus = '1'
        end
        js.vm.network 'private_network', ip: '192.168.40.12'
        js.vm.provision 'shell', path: 'web_server_bootstrap.sh'
    end
end
```

4. Create `web_server_bootstrap.sh` file
```
#!/usr/bin/env bash

echo 'Hello! I am a Web Server...'

# Install wget
yum install wget -y

# Install java
yum install java-1.8* -y

# Set up JAVA_HOME variable
FILE=/etc/profile.d/jdk_home.sh
if [ -f "$FILE" ]; then
    echo "$FILE exist"
else
    echo "$FILE does not exist"
    touch $FILE
    JAVA_HOME_TEMP=`sudo find /usr/lib/jvm/java-1.8* | head -n 3 | tail -n 1`
    echo '#!/bin/sh' > $FILE
    echo "export JAVA_HOME=$JAVA_HOME_TEMP" >> $FILE
    echo 'export PATH=$PATH:$JAVA_HOME' >> $FILE
fi

# Install tomcat server
wget https://apache.ip-connect.vn.ua/tomcat/tomcat-8/v8.5.53/bin/apache-tomcat-8.5.53.tar.gz
tar -xvzf apache-tomcat-8.5.53.tar.gz
mv apache-tomcat-8.5.53 /opt/tomcat
# Enable login from outside
sed -i '19,20d' /opt/tomcat/webapps/host-manager/META-INF/context.xml
sed -i '19,20d' /opt/tomcat/webapps/manager/META-INF/context.xml
# Add tomcat users
mv /vagrant/tomcat/tomcat-users.xml /opt/tomcat/conf/tomcat-users.xml -f
# Start tomcat server
/opt/tomcat/bin/startup.sh
```

5. Create `tomcat/tomcat-users.xml` file
```
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
 <role rolename="manager-gui"/>
 <role rolename="manager-script"/>
 <role rolename="manager-jmx"/>
 <role rolename="manager-status"/>
 <user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
 <user username="deployer" password="deployer" roles="manager-script"/>
 <user username="tomcat" password="tomcat" roles="manager-gui"/>
</tomcat-users>
```

6. Start Tomcat Server VM  
`vagrant up`  
