# DevOps Basics : Ansible

### Table of Contents
**[Prerequisites](#prerequisites)**  
**[Create Ansible Server VM](#create-ansible-server-vm)**  
**[Setup Ansible Server](#setup-ansible-server)**  
**[Create Docker project on Ansible Server](#create-docker-project-on-ansible-server)**  
**[Setup Ansible Server and Docker Registry communication](#setup-ansible-server-and-docker-registry-communication)**  
**[Jenkins integration](#jenkins-integration)**  
**[`Deploy_on_Docker_Container_using_Ansible_Playbooks` Jenkins job](#deploy_on_docker_container_using_ansible_playbooks-jenkins-job)**  

### Prerequisites
- VirtualBox
- Vagrant
- Jenkins
- Docker Host
- Docker Registry
### Create Ansible Server VM
- Create Vagrant project
    ```
    mkdir ./ansible_server
    cd ./ansible_server
    touch Vagrantfile
    ```
- Set Vagrantfile content
    ```
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure('2') do |config|
        config.vm.define 'ansible_server' do |as|
            as.vm.box = 'centos/7'
            as.vm.hostname = 'ansible-server'
            as.vm.provider 'virtualbox' do |vb|
                vb.name = 'ansible_server'
                vb.memory = '2048'
                vb.cpus = '1'
            end
            as.vm.network 'private_network', ip: '192.168.40.16'
        end
    end
    ```
- Turn on VM
    ```
    vagrant up
    vagrant ssh
    ```
### Setup Ansible Server
- Install packages
    ```
    sudo su -
    yum install epel-release
    yum install ansible
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
- Create `ansadmin` on Ansible Server
    ```
    useradd ansadmin
    passwd ansadmin
    echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
    usermod -aG docker ansadmin
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    systemctl restart sshd

    su - ansadmin
    ssh-keygen
    ```
- Create `ansadmin` on Docker Host
    ```
    sudo su -
    useradd ansadmin
    passwd ansadmin
    usermod -aG docker ansadmin
    echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
    ```
- Add SSH key of `ansadmin` from Ansible Server to Docker Host and localhost
    ```
    su - ansadmin
    ssh-copy-id ansadmin@192.168.40.15
    ssh-copy-id localhost
    ```
- Setup Ansible inventory
    - Create `hosts` file
        ```
        sudo vi /etc/ansible/hosts
        ```
    - Set `hosts` file content
        ```
        [local]
        127.0.0.1

        [docker]
        192.168.40.15
        ```
- Perform test run with `ping` module 
    ```
    ansible all -m ping
    ```
### Create Docker project on Ansible Server
- Create project folder
    ```
    sudo su -
    mkdir /opt/docker
    chown ansadmin:ansadmin /opt/docker
    su - ansadmin
    cd /opt/docker
    ```
- Create `Dockerfile`
    ```
    vi Dockerfile
    ```
- Set `Dockerfile` content
    ```
    FROM tomcat:latest

    MAINTAINER mykhailo.shabalov@gmail.com

    COPY ./webapp.war /usr/local/tomcat/webapps/
    ```
- Create `create-simple-devops-image` playbook file
    ```
    vi create-simple-devops-image.yml
    ```
- Set `create-simple-devops-image` playbook file content
    ```
    ---
    - hosts: local
      become: true

      tasks:

      - name: create docker image using war file
        command: docker build -t simple-devops-image:latest .
        args:
          chdir: /opt/docker

      - name: create tag to image
        command: docker tag simple-devops-image docker-registry.example.com:5000/simple-devops-image

      - name: push image to docker registry
        command: docker push docker-registry.example.com:5000/simple-devops-image

      - name: remove docker image from ansible server
        command: docker rmi simple-devops-image:latest docker-registry.example.com:5000/simple-devops-image
        ignore_errors: yes
    ```
- Create `create-simple-devops-project` playbook file
    ```
    vi create-simple-devops-project.yml
    ```
- Set `create-simple-devops-project` playbook file content
    ```
    ---
    - hosts: docker
      become: true

      tasks:

      - name: stop current running container
        command: docker stop simple-devops-container
        ignore_errors: yes

      - name: remove stopped container
        command: docker rm simple-devops-container
        ignore_errors: yes

      - name: remove docker image
        command: docker rmi docker-registry.example.com:5000/simple-devops-image:latest
        ignore_errors: yes

      - name: pull docker image from registry
        command: docker pull docker-registry.example.com:5000/simple-devops-image:latest

      - name: create docker container using new image
        command: docker run -d --name simple-devops-container -p 8080:8080 docker-registry.example.com:5000/simple-devops-image:latest
    ```

### Setup Ansible Server and Docker Registry communication
- Copy certificate file from Docker Host
    ```
    sudo su -
    scp /var/lib/docker/containers/docker-registry/certs/docker-registry.crt ansadmin@192.168.40.16:/opt/docker
    ```
- Move certificate on Ansible Server
    ```
    sudo su -
    echo '192.168.40.15 docker-registry.example.com docker-registry' >> /etc/hosts
    mkdir -p /etc/docker/certs.d/docker-registry.example.com:5000
    mv /opt/docker/docker-registry.crt /etc/docker/certs.d/docker-registry.example.com:5000/ca.crt
    ```
- Authenticate `root` and `ansadmin` on Docker Registry
    ```
    docker login docker-registry.example.com:5000
    # docker_user : 123

    su - ansadmin
    docker login docker-registry.example.com:5000
    # docker_user : 123
    ```
### Jenkins integration
- Check Ansible playbooks manually
    ```
    cd /opt/docker
    # Test run
    ansible-playbook create-simple-devops-image.yml --check
    ansible-playbook create-simple-devops-project.yml --check
    # Actual run
    ansible-playbook create-simple-devops-image.yml
    ansible-playbook create-simple-devops-project.yml
    ```
- Configure `Publish Over SSH` plugin
    - `Manage Jenkins` > `Configure System` > `Publish Over SSH` > `SSH Servers`
        ```
        Name: ansible-server
        Hostname: 192.168.40.16
        Username: ansadmin
            # Advanced...
            (/) Use password authentication, or use a different key
            Passphrase / Password: ansadmin
                # Test Configuration
        ```
### `Deploy_on_Docker_Container_using_Ansible_Playbooks` Jenkins job
- Create new Maven project `Deploy_on_Docker_Container_using_Ansible_Playbooks`
    ```
    Description: Deploy on Docker Container using Ansible Playbooks
    Git:
        Repository URL: https://github.com/michaelshabalov/hello-world.git
        Branch Specifier: */master
    Enable Poll SCM: * * * * *
    Build:
        Root POM: pom.xml
        Goals and options: clean install package
    Post-build Actions:
        Send build artifacts over SSH:
            Name: ansible-server
            Source files: webapp/target/webapp.war
            Remove prefix: webapp/target
            Remote directory: //opt//docker
            Exec command:   ansible-playbook /opt/docker/create-simple-devops-image.yml;
                            ansible-playbook /opt/docker/create-simple-devops-project.yml;
    ```
