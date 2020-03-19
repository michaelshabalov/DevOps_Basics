1. Install Pipeline plugin
2. Restart jenkins
3. Create simple Pipeline project

```
pipeline {
    agent any
    tools {
        maven 'M2_HOME'
    }
    options {
        skipDefaultCheckout()
    }
    stages {
        stage('GitHub Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/michaelshabalov/hello-world.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install package'
            }
        }
        stage('Deploy') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'deployer', path: '', url: 'http://192.168.40.12:8080/')], contextPath: null, onFailure: false, war: '**/*.war'
            }
        }
    }
}
```

4. Create Jenkins slave vagrant project
- Vagrantfile
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
    config.vm.define 'jenkins_slave' do |jsl|
        jsl.vm.box = 'centos/7'
        jsl.vm.hostname = 'jenkins-slave'
        jsl.vm.provider 'virtualbox' do |vb|
            vb.name = 'jenkins_slave'
            vb.memory = '2048'
            vb.cpus = '1'
        end
        jsl.vm.network 'private_network', ip: '192.168.40.11'
        jsl.vm.provision 'shell', path: 'jenkins_slave_bootstrap.sh'
    end
end
```

- jenkins_slave_bootstrap.sh
```
#!/usr/bin/env bash

echo 'Hello! I am a Jenkins Slave...'

# Install wget
yum install wget git -y

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
```

5. Add new Jenkins slave  
`http://yallalabs.com/devops/how-to-add-linux-slave-node-agent-node-jenkins/`

6. Create simple job to check Jenkins slave  
```
pipeline {
    agent {
        label '192.168.40.11'
    }
    stages {
        stage('Welcome') {
            steps {
                echo 'Hello world!'
            }
        }
        stage('Touch') {
            steps {
                sh 'touch 1.txt'
            }
        }
    }
}
```
