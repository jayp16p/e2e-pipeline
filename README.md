# e2e-pipeline
e2e-pipeline

#### What we're doing
 - We will install Jenkins (run jenkins agent for distributed builds)
 - Install SonarQube for static code analysis and quality gate checks
 - Build Docker containers and push them to Dockerhub & use of image tags to help with versioning
 - Explore ArgoCD - a popular gitops tool to continuously deploy the latest Docker image

![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/4ce78e34-e222-49fe-bbb1-48bba1017032)

- You can adjust your IP's as you wish, I'm using 192.168.56.x for my homelab
- 
- Ensure you have your DNS records point to your VM's for the reverse proxies and TLS to work
![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/18268951-75a9-4304-8a65-83b48f7d7895)

- If it makes it easier, I have a Vagrantfile attached which could be used if you're not one of those guys who likes to do everything manually :-p
![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/766a8897-3939-47db-b633-9c5bccb54194)

### Install Jenkins & Java(using Adoptium)
```
sudo -i
sudo apt update
sudo apt upgrade

wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo apt-key add -
echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list


apt update
apt install temurin-17-jdk
/usr/bin/java --version
exit 

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins

sudo systemctl start jenkins

Output
● jenkins.service - LSB: Start Jenkins at boot time
   Loaded: loaded (/etc/init.d/jenkins; generated)
   Active: active (exited) since Fri 2020-06-05 21:21:46 UTC; 45s ago
     Docs: man:systemd-sysv-generator(8)
    Tasks: 0 (limit: 1137)
   CGroup: /system.slice/jenkins.service

```
- If everything worked, you should see it come up with IP ADDRESS:8080

### Setup Reverse Proxy using NGINX

```
sudo apt update
sudo apt upgrade

sudo apt install nginx

systemctl status nginx

```

#### In order for Nginx to serve this content, it’s necessary to create a server block with the correct directives.

```
sudo vi /etc/nginx/sites-available/jenkinsdev.jaycloud.live

sudo vi /etc/nginx/sites-available/jenkinsagent.jaycloud.live
```

```
#PASTE THIS for each host and change server_name as appropriate!
upstream jenkins{
    server 127.0.0.1:8080;
}

server{
    listen      80;
    server_name jenkinsdev.jaycloud.live;

    access_log  /var/log/nginx/jenkins.access.log;
    error_log   /var/log/nginx/jenkins.error.log;

    proxy_buffers 16 64k;
    proxy_buffer_size 128k;

    location / {
        proxy_pass  http://jenkins;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect off;

        proxy_set_header    Host            $host;
        proxy_set_header    X-Real-IP       $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto https;
    }

}
```

- Next, let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:
```
sudo ln -s /etc/nginx/sites-available/jenkinsdev.jaycloud.live /etc/nginx/sites-enabled/
# AND
sudo ln -s /etc/nginx/sites-available/jenkinsagent.jaycloud.live /etc/nginx/sites-enabled/
```

- Next, test to make sure that there are no syntax errors in any of your Nginx files:
```
sudo nginx -t
```

- Lastly, restart NGINX: ```sudo systemctl restart nginx```

If all worked you should see:
![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/6c445acb-ed73-4c1b-aa6e-6bde7eb5da5f)


### Setup Jenkins Agent - Similar to the previous one + Install Docker
```
sudo apt update
sudo apt upgrade

sudo adduser jenkins

#Grant Sudo Rights to Jenkins User
sudo usermod -aG sudo jenkins

wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo apt-key add -
echo "deb https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list


apt update
apt install temurin-17-jdk
/usr/bin/java --version
exit
###############################################################################################################
###DOCKER###
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

#Manage Docker as a non-root user
sudo groupadd docker
sudo usermod -aG docker jenkins # recall that we created a user called Jenkins when we started! or user $USER

#Run the following command to activate the changes to groups:
newgrp docker

#Switch to jenkins user
su - jenkins
#run docker ps to verify you are able to run without root on jenkins user!
```


### Connect to Remote SSH Agent
- From the Jenkins UI (Controller) ```ssh jenkins@192.168.56.12```
- Create private and public SSH keys. The following command creates the private key jenkinsAgent_rsa and the public key jenkinsAgent_rsa.pub.
  It is recommended to store your keys under ~/.ssh/ so we move to that directory before creating the key pair.
  ``` mkdir ~/.ssh; cd ~/.ssh/ && ssh-keygen -t rsa -m PEM -C "Jenkins agent key" -f "jenkinsAgent_rsa" ```
- Add the public SSH key to the list of authorized keys on the agent machine
  ```cat jenkinsAgent_rsa.pub >> ~/.ssh/authorized_keys```
- Copy the private SSH key (~/.ssh/jenkinsAgent_rsa) from the agent machine to your OS clipboard
  ```cat ~/.ssh/jenkinsAgent_rsa```
  
- Go to the jenkinsagent VM and SSH to Jenkins UI VM
- Switch to Jenkins user ```su - jenkins``` and run ```cd /var/lib/jenkins/```
- Now SSH to JenkinsAgent ```ssh jenkins@192.168.56.12``` - this will help in creating the SSH files in this dir and will help in connecting UI and agent

- Go to JenkinsUI(192.168.56.11) -> Manage Jenkins -> Nodes&Clouds -> Configure the Built in Node-> Set Number of executors to 0
  ![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/9d6e28cf-3422-4127-be52-846f7d0c5def)

- Go back and add new node
  ![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/805a3aef-5667-4d49-a59e-76ba7e4d099a)

  Set as:
  ![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/9c84042f-b447-4a5e-a87e-e9a9d45fe6e2)
  
  Add credential and set as #this is where we use the copied private key from Jenkins agent
  ![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/a41d27e9-f41e-48c4-ad85-3b38a2eb69b1)
  
  USE THOSE CREDENTIALs
  ![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/98abb867-89a5-4615-aff7-42a85c38e0a0)

  If everthing worked well, you should see the agent connected to the UI
  ![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/fb11d55b-2da6-4faa-87f6-8cdad65abb69)


