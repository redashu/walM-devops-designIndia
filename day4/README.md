### COntiniuou monitoring also import 
<img src="m1.png">

### over app desing pattern need 

<img src="app1.png">

### updated azure pipeline yaml

```
# Maven
# Build your Java project and run tests with Apache Maven.
# Add steps that analyze code, save build artifacts, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/java changes

trigger:
- master

pool: Default # i want to use my own agent 
#  vmImage: ubuntu-latest

stages: 
- stage: ashuappbuild # build stage for create war file 
  jobs:
  - job: runcommand
    steps:
    - script: echo 'hello world'
      displayName: 'testing stages and jobs'
      
  - job: runmavenbuild   
    steps:
    - task: Maven@3
      inputs:
        mavenPomFile: 'pom.xml'
        mavenOptions: '-Xmx3072m'
        javaHomeOption: 'JDKVersion'
        jdkVersionOption: '1.8'
        jdkArchitectureOption: 'x64'
        publishJUnitResults: true
        testResultsFiles: '**/surefire-reports/TEST-*.xml'
        goals: 'package'
    - script: | 
        echo "current maven build"
        ls 

- stage: createinfraonAWSCloud  # trying to create VM in AWS cloud platform using terraform
  jobs:
  - job: testing_terraform_installation
    steps:
    - script: | 
        sudo snap install terraform --classic # incase we want to install from pipeline 
        echo 'checking terraform version'
        ls 
        mkdir -p /tmp/ashutf
        cp -rf terraform_code/*.tf /tmp/ashutf/
        terraform -v
        echo 'Now lets run terraform code'
        cd /tmp/ashutf
        terraform init # it will download module of terraform 
        terraform plan  # will let you know what is going to happen 
        sleep 2
        terraform apply -target=aws_key_pair.ashu_genkey --auto-approve  # create resources in aws cloud 
        terraform apply --auto-approve


```

### now ansible will install and configure java / tomcat in vm created by terraform using azdevops 

<img src="ansible.png">

### ansible playbook for tomcat install and configure 

```
---
- name: Install and Start Apache Tomcat
  hosts: all  # all machine what ever inventory has 
  remote_user: ec2-user
  become: yes

  vars:
    tomcat_version: "9.0.86"

  tasks:
    - name: Install Tomcat dependencies
      package:
        name:
          - unzip
          - wget

    - name: Download Apache Tomcat
      get_url:
        url: "https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.tar.gz"
        dest: "/tmp/apache-tomcat.tar.gz"

    - name: 
      yum:
       name: java-1.8*
       state: present
    - name: Extract Tomcat archive
      ansible.builtin.unarchive:
        src: "/tmp/apache-tomcat.tar.gz"
        dest: "/opt"
        remote_src: yes

      
    - name: Start Tomcat service
      command: "chmod 755 /opt/apache-tomcat-9.0.86/ -R"
    - name: Start Tomcat service
      command: "nohup /opt/apache-tomcat-9.0.86/bin/startup.sh"
```
