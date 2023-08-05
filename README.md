# e2e-pipeline
e2e-pipeline

#### What we're doing
 - We will install Jenkins with TLS for secure communication (run jenkins agent for distributed builds)
 - Install SonarQube with TLS for static code analysis and quality gate checks
 - Build Docker containers and push them to Dockerhub & use of image tags to help with versioning
 - Explore ArgoCD - a popular gitops tool to continuously deploy the latest Docker image

![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/4ce78e34-e222-49fe-bbb1-48bba1017032)

- You can adjust your IP's as you wish, I'm using 192.168.56.x for my homelab
- Ensure you have your DNS records point to your VM's for the reverse proxies and TLS to work
![image](https://github.com/jayp16p/e2e-pipeline/assets/106398902/18268951-75a9-4304-8a65-83b48f7d7895)

