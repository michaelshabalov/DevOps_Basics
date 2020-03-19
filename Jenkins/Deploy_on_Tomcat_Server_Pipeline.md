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

4. Add new Jenkins slave  
`http://yallalabs.com/devops/how-to-add-linux-slave-node-agent-node-jenkins/`

5. Create simple job to check Jenkins slave  
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
