# DevOps Basics : Docker

### Table of Contents
**[Prerequisites](#prerequisites)**  
**[Create Docker-Host VM](#prerequisites)**  
**[Setup Docker CE](#setup-docker-ce)**  
**[Jenkins integration](#jenkins-integration)**  
**[`Deploy_on_Docker` Jenkins job](#deploy_on_docker-jenkins-job)**  
**[Tomcat server Dockerfile](#tomcat-server-dockerfile)**  
**[`Deploy_on_Container` Jenkins job](#deploy_on_container-jenkins-job)**  

### Prerequisites
- VirtualBox
- Vagrant
### Create Docker-Host VM
- Create Vagrant project
    ```
    mkdir ./docker_host
    cd ./docker_host
    touch Vagrantfile
    ```
- Set Vagrantfile content
    ```
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure('2') do |config|
        config.vm.define 'docker_host' do |dh|
            dh.vm.box = 'centos/7'
            dh.vm.hostname = 'docker-host'
            dh.vm.provider 'virtualbox' do |vb|
                vb.name = 'docker_host'
                vb.memory = '2048'
                vb.cpus = '1'
            end
            dh.vm.network 'private_network', ip: '192.168.40.15'
        end
    end
    ```
- Turn on VM
    ```
    vagrant up
    vagrant ssh
    ```
### Setup Docker CE
- Some topic 
    ```
    sudo su - root
    yum install -y \
        yum-utils \
        device-mapper-persistent-data \
        lvm2
    yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y \
        docker-ce \
        docker-ce-cli \
        containerd.io
    systemctl start docker
    systemctl enable docker
    ```

    ```
    docker pull hello-world:latest
    docker run --name hello-world-container hello-world:latest
    docker ps -a
    docker rm hello-world-container
    ```

    ```
    docker pull tomcat:latest
    docker image list
    docker run --name tomcat-container -p 8080:8080 tomcat:latest
    docker ps -a
    ```

    ```
    docker exec -it tomcat-container /bin/bash
    cp -R /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps/
    exit
    ```
### Jenkins integration
- Install `Publish Over SSH` plugin
- Create `dockeradmin` on Docker-Host VM
    ```
    useradd dockeradmin
    passwd dockeradmin
    cat /etc/group
    usermod -aG docker dockeradmin
    id dockeradmin
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    systemctl restart sshd
    ```
- Configure `Publish Over SSH` plugin
    - `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers`
        ```
        Name: docker-host
        Hostname: 192.168.40.15
        Username: dockeradmin
            # Advanced...
            (/) Use password authentication, or use a different key
            Passphrase / Password: dockeradmin
                # Test Configuration
        ```
### `Deploy_on_Docker` Jenkins job
- Create new Maven project `Deploy_on_Docker`
    ```
    Description: Deploy on Docker
    Git:
        Repository URL: https://github.com/michaelshabalov/hello-world.git
        Branch Specifier: */master
    Build:
        Root POM: pom.xml
        Goals and options: clean install package
    Post-build Actions:
        Send build artifacts over SSH:
            Name: docker-host
            Source files: webapp/target/webapp.war
            Remove prefix: webapp/target
            Remote directory: .
    ```
- Observe job result
    ```
    ls -la /home/dockeradmin/
    ```
### Tomcat server Dockerfile
- Create Dockerfile
    ```
    su - dockeradmin
    cd ~
    vi Dockerfile
    ```
- Set Dockerfile content
    ```
    FROM tomcat:latest

    MAINTAINER mykhailo.shabalov@gmail.com

    COPY ./webapp.war /usr/local/tomcat/webapps/
    ```
- Build `devops-tomcat-image`
    ```
    docker build -t devops-tomcat-image .
    docker image list
    ```
- Run `devops-tomcat-container`
    ```
    docker run -d --name devops-tomcat-container -p 8080:8080 devops-tomcat-image
    ```
    - Observe result in browser
        ```
        http://192.168.40.15:8080/webapp/
        ```
### `Deploy_on_Container` Jenkins job
- Create new Maven project `Deploy_on_Container`
    ```
    Description: Deploy on Container
    Git:
        Repository URL: https://github.com/michaelshabalov/hello-world.git
        Branch Specifier: */master
    Build:
        Root POM: pom.xml
        Goals and options: clean install package
    Post-build Actions:
        Send build artifacts over SSH:
            Name: docker-host
            Source files: webapp/target/webapp.war
            Remove prefix: webapp/target
            Remote directory: .
            Exec command: cd /home/dockeradmin; docker build -t devops-tomcat-image .; docker run -d --name devops-tomcat-container -p 8080:8080 devops-tomcat-image;
    ```
- Observe result in browser
    ```
    http://192.168.40.15:8080/webapp/
    ```
