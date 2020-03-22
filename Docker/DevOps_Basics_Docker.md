# DevOps Basics : Docker

### Table of Contents
**[Prerequisites](#prerequisites)**  
**[Create Docker-Host VM](#prerequisites)**  
**[Setup Docker CE](#setup-docker-ce)**  
**[Jenkins integration](#jenkins-integration)**  
**[`Deploy_on_Docker` Jenkins job](#deploy_on_docker-jenkins-job)**  
**[Tomcat server Dockerfile](#tomcat-server-dockerfile)**  
**[`Deploy_on_Container` Jenkins job](#deploy_on_container-jenkins-job)**  
**[Setup Docker Registry](#setup-docker-registry)**  

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
Source: https://docs.docker.com/install/linux/docker-ce/centos/
- Install packages
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
- Run hello-world container
    ```
    docker pull hello-world:latest
    docker run --name hello-world-container hello-world:latest
    docker ps -a
    docker rm hello-world-container
    ```
- Run tomcat container
    ```
    docker pull tomcat:latest
    docker image list
    docker run --name tomcat-container -p 8080:8080 tomcat:latest
    docker ps -a
    ```
- Explore tomcat container from inside
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
### Setup Docker Registry
Source: https://www.centlinux.com/2019/03/configure-private-docker-registry-centos-7.html
- Login to Docker-Host VM
    ```
    vagrant ssh
    sudo su - root
    ```
- Generate a self-signed digital certificate
    ```
    mkdir -p /var/lib/docker/containers/docker-registry/certs
    openssl req \
    -newkey rsa:2048 \
    -nodes -sha256 \
    -x509 -days 365 \
    -keyout /var/lib/docker/containers/docker-registry/certs/docker-registry.key \
    -out /var/lib/docker/containers/docker-registry/certs/docker-registry.crt

    ###
    Country Name (2 letter code) [XX]:UA
    State or Province Name (full name) []:Kyiv
    Locality Name (eg, city) [Default City]:Kyiv
    Organization Name (eg, company) [Default Company Ltd]:Global DevOps Basics
    Organizational Unit Name (eg, section) []:ITLAB
    Common Name (eg, your name or your server's hostname) []:docker-registry.example.com
    Email Address []:root@docker-01.example.com
    ###
    ```
- Configure Basic HTTP Authentication for Private Docker Registry
    ```
    mkdir -p /var/lib/docker/containers/docker-registry/auth
    docker run \
    --entrypoint htpasswd \
    registry -Bbn docker_user 123 > /var/lib/docker/containers/docker-registry/auth/htpasswd
    ```
- Create a Directory to persist Private Docker Registry Data
    ```
    mkdir /var/lib/docker/containers/docker-registry/registry
    ```
- Pull registry image from Docker Hub
    ```
    docker pull registry
    ```
- Create a container for Private Docker Registry
    ```
    docker run -d \
    --name docker-registry \
    --restart=always \
    -p 5000:5000 \
    -v /var/lib/docker/containers/docker-registry/registry:/var/lib/registry \
    -v /var/lib/docker/containers/docker-registry/auth:/auth \
    -e "REGISTRY_AUTH=htpasswd" \
    -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
    -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
    -v /var/lib/docker/containers/docker-registry/certs:/certs \
    -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/docker-registry.crt \
    -e REGISTRY_HTTP_TLS_KEY=/certs/docker-registry.key \
    registry
    ```
- Add IP Address of Private Docker Registry to Local DNS resolver of Docker host
    ```
    echo '127.0.0.1 docker-registry.example.com docker-registry' >> /etc/hosts
    ```
- Install digital security certificate on Docker host
    ```
    mkdir -p /etc/docker/certs.d/docker-registry.example.com:5000
    cp /var/lib/docker/containers/docker-registry/certs/docker-registry.crt /etc/docker/certs.d/docker-registry.example.com:5000/ca.crt
    ```
- Pull an image from Docker Hub. We will later push this image to our Private Docker Registry
    ```
    docker pull busybox
    ```
- Create another tag for busybox image, so we can push it into our Private Docker Registry
    ```
    docker tag busybox:latest docker-registry.example.com:5000/busybox
    ```
- Login to docker-registry.example.com using docker command
    ```
    docker login docker-registry.example.com:5000

    ###
    Username: docker_user
    Password: 123
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json. Configure a credential helper to remove this warning. See https://docs.docker.com/engine/reference/commandline/login/#credentials-store
    Login Succeeded
    ###
    ```
- Push busybox image to Private Docker Registry
    ```
    docker push docker-registry.example.com:5000/busybox
    ```
- Observe result
    ```
    ls -la /var/lib/docker/containers/docker-registry/registry
    ```
