# Install Jenkins on CentOS 7
Jenkins is a self-contained Java-based program, ready to run out-of-the-box, with packages for Windows, Mac OS X and other Unix-like operating systems. As an extensible automation server, Jenkins can be used as a simple CI server or turned into the continuous delivery hub for any project.

### Prerequisites
1. Installed VirualBox
1. Installed Vagrant

## Prepare Jenkins Server Vagrant Project
1. Create Project Folder
2. Init Vagrantfile  
`vagrant init`
3. Copy Vagrantfile Content
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
    config.vm.define 'jenkins_server' do |js|
        js.vm.box = 'centos/7'
        js.vm.hostname = 'jenkins-server'
        js.vm.provider 'virtualbox' do |vb|
            vb.name = 'jenkins_server'
            vb.memory = '4096'
            vb.cpus = '2'
        end
        js.vm.network 'private_network', ip: '192.168.40.10'
        js.vm.provision 'shell', path: 'jenkins_server_bootstrap.sh'
    end
end
```

4. Create `jenkins_server_bootstrap.sh` file
```
#!/usr/bin/env bash

echo 'Hello! I am a Jenkins Server...'
# Update system
# yum update -y

# Install git
yum install git -y

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

# Install Maven
wget https://apache.ip-connect.vn.ua/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar -xvzf apache-maven-3.6.3-bin.tar.gz
mv apache-maven-3.6.3 /opt/maven

# Setup Maven variables
FILE=/etc/profile.d/maven.sh
if [ -f "$FILE" ]; then
    echo "$FILE exist"
else
    touch $FILE
    echo '#!/bin/sh' > $FILE
    echo 'export M2_HOME=/opt/maven' >> $FILE
    echo 'export M2=/opt/maven/bin' >> $FILE
    echo 'export PATH=$PATH:$M2_HOME:$M2' >> $FILE
fi

# Add Jenkins repo
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Jenkins
yum install jenkins -y

# Start and enable Jenkins service
systemctl start jenkins
systemctl enable jenkins

# Writing useful info
echo 'JAVA_HOME'
echo $JAVA_HOME_TEMP
```

5. Start Jenkins Server VM  
`vagrant up`  

## Configure Jenkins
- The default Username is `admin`
- Grab the default password 
- Password Location:`/var/lib/jenkins/secrets/initialAdminPassword`
- `Skip` Plugin Installation; _We can do it later_
- Change admin password
   - `Admin` > `Configure` > `Password`
- Configure `java` path
  - `Manage Jenkins` > `Global Tool Configuration` > `JDK`  
  Name `JAVA_HOME`  
  JAVA_HOME `/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64`  

### Test Jenkins Jobs
1. Create “new item”
1. Enter an item name – `My-First-Project`
   - Chose `Freestyle` project
1. Under the Build section
	Execute shell: echo "Welcome to Jenkins Demo"
1. Save your job 
1. Build job
1. Check "console output"
