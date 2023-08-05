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

