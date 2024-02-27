# Revision 

<img src="rev.png">

### more about azure devops pipeline 

<img src="pipe1.png">

### azure devops agent 

<img src="az.png">

### for this training lets configure all the things we are going to use -- Before installing agent 

### update

```
sudo apt update
```

### JDk 11 and maven 

```
sudo apt install default-jre
sudo apt install default-jdk
```

### Installing docker 

```
sudo snap install docker
 sudo addgroup --system docker
sudo adduser $USER docker
newgrp docker
sudo snap disable docker
sudo snap enable docker
```

### Now installing agent 

```
 wget https://vstsagentpackage.azureedge.net/agent/3.234.0/vsts-agent-linux-x64-3.234.0.tar.gz
 tar xvzf vsts-agent-linux-x64-3.234.0.tar.gz
./config.sh
./run.sh 
```

### CI in azure devops with docker container 

<img src="cicd5.png">

### build output sharing with artifacts server

<img src="art.png">

### service connection in azure devops 

<img src="azd.png">


