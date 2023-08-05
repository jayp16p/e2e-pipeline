# e2e-pipeline
e2e-pipeline

#### What we're doing
 - We will install Jenkins with TLS for secure communication (run jenkins agent for distributed builds)
 - Install SonarQube with TLS for static code analysis and quality gate checks
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

wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list

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
- If everything worked, you should see it come up with <ip addr>:8080

- Next, we setup reverse proxy


