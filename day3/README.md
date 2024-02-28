# Revision 

<img src="rev1.png">

### azure devops users and groups

<img src="rev2.png">

### IAC using terraform 
### to configure aws creds in azure devops agent machine run below command with that user which has azdevops agent running

```
azureuser@linux-agent:~/poc$ aws  configure 
AWS Access Key ID [****************ABOD]: 
AWS Secret Access Key [****************iMG4]: 
Default region name [us-east-1]: 
Default output format [None]: 
azureuser@linux-agent:~/poc$ 
azureuser@linux-agent:~/poc$ whoami
azureuser
azureuser@linux-agent:~/poc$ ls  /home/azureuser/.aws/
config  credentials
azureuser@linux-agent:~/poc$ 

```

<img src="terraform.png">

### more info about terraform 

<img src="terraform1.png">

### azure pipeline.yaml data

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
- stage: ashuappbuild
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

```

### adding second stage there 

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

- stage: createinfraonAWSCloud  # trying to create VM in AWS cloud platform using terraform
  jobs:
  - job: testing_terraform_installation
    steps:
    - script: | 
        echo 'checking terraform version'
        terraform -v


```

### terraform connect with aws cloud to provision resources

<img src="res.png">

### terraforce to create public and private key 

```
provider "aws" {
   region = "us-east-1" # NV 
}
# creating key pair to connect cloud machine
resource "tls_private_key" "ashu-key" {
  algorithm = "RSA"
  rsa_bits  = 4096
} 
# creating key pair using above alog 
resource "aws_key_pair" "ashu_genkey" {
  key_name   = "ashu_private_key"
  public_key = tls_private_key.ashu-key.public_key_openssh
  provisioner "local-exec" { # Create "myKey.pem" to your computer!!
    command = "echo '${tls_private_key.ashu-key.private_key_pem}' > ./ashukey.pem"
  }
}
```

### adding pipeline changes 

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

- stage: createinfraonAWSCloud  # trying to create VM in AWS cloud platform using terraform
  jobs:
  - job: testing_terraform_installation
    steps:
    - script: | 
        sudo snap install terraform --classic # incase we want to install from pipeline 
        echo 'checking terraform version'
        terraform -v
        echo 'Now lets run terraform code'
        cd terraform_code 
        terraform init # it will download module of terraform 
        terraform plan  # will let you know what is going to happen 
        sleep 2
        terraform apply --auto-approve  # create resources in aws cloud 

```

### key location on agent side 

```
azureuser@linux-agent:~/agent$ find ~/ -iname *.pem 
/home/azureuser/agent/_work/9/s/terraform_code/suhaskey.pem
/home/azureuser/agent/_work/8/s/terraform-files/kanikapvtkey.pem
/home/azureuser/agent/_work/6/s/terraform_code/ashukey.pem
/home/azureuser/poc/walm.pem

```


### updated terraform code to create linux vm 

```
provider "aws" {
   region = "us-east-1" # NV 
}
# creating key pair to connect cloud machine
resource "tls_private_key" "ashu-key" {
  algorithm = "RSA"
  rsa_bits  = 4096
} 
# creating key pair using above alog 
resource "aws_key_pair" "ashu_genkey" {
  key_name   = "ashu_private_key"
  public_key = tls_private_key.ashu-key.public_key_openssh
  provisioner "local-exec" { # Create "myKey.pem" to your computer!!
    command = "echo '${tls_private_key.ashu-key.private_key_pem}' > ./ashukey.pem"
  }
}

# now using resource to create Virutal machine 
resource "aws_instance" "ashuvm" {

    ami = "ami-07761f3ae34c4478d"
    instance_type = "t2.micro"

    key_name = "ashu_private_key"
    tags = {
      "Name" = "ashu-linux-vm"
    }
}

```

### final azure pipeline file 

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
