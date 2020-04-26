## Dev Server

This repository contains installer that allows you to set up the system for software development.  
Dev Server includes:
- Atlassian Jira
- Atlassian Bitbucket
- Atlassian Confluence
- Jenkins
- Nexus Repository Manager
- SonarQube

**Note**: You must have licenses for Atlassian products!  
Jenkins, Nexus and SonarQube are in community versions.

#### Guide to install the software

1. Prepare server with Ubuntu (tested with Ubuntu 18.04)
2. Connect via SSH to your server as root user
3. Install Ansible
   ```bash
   apt-get update
   apt-get install python3-pip python3-setuptools
   pip3 install ansible
   ```
4. Clone the GIT repository (master branch)
   ```bash
   git clone https://github.com/przemyslawsikora/dev-server.git
   ```
5. Create private and public key for your user from which you can login to the system 
   via SSH (use PuttyGen on Windows or ssh-keygen on Linux, MacOS)
6. Copy the public key into the <code>installer/roles/users/files</code> as <code>admin_key.pub</code>  
   Ensure the content is similar to
   ```bash
   ssh-rsa AAAAB3...36== {user_name}
   ```
7. Optional but **recommended** - change the default passwords in the file <code>secret.yml</code>  
   Default vault password used to open and edit the file is <code>12345</code>
   ```bash
   ansible-vault edit installer/vars/secret.yml
   ```
   Change also the vault password
   ```bash
   ansible-vault rekey installer/vars/secret.yml
   ```
8. Open <code>installer/vars/all.yml</code> file and change the content, especially:
   - domain
   - admin_email
9. Enter directory <code>installer</code>
10. Install Ansible roles
   ```bash
   ansible-galaxy install -r requirements.yml
   ```
11. Run installer
   ```bash
   ansible-playbook installer.yml -i production --ask-vault-pass -e 'ansible_python_interpreter=/usr/bin/python3'
   ```

After installation, you should have accessible following applications:

| Application                           | Version         | Address (assuming your domain is example.com)  |
|------------------------------------   |--------------   |---------------------------------------------   |
| Atlassian Jira                        | 8.8             | https://jira.example.com                  	   |
| Atlassian Bitbucket                   | 7.1             | https://bitbucket.example.com             	   |
| Atlassian Confluence                  | 7.4             | https://confluence.example.com             	   |
| Jenkins                               | 2.233           | https://jenkins.example.com               	   |
| Nexus Repository Manager              | 3.22.1          | https://nexus.example.com                  	   |
| SonarQube                             | 8.2 (community) | https://sonar.example.com                 	   |
| Private Docker Registry (by Nexus)    | 3.22.1          | https://docker.example.com                	   |

#### Post Configuration

##### Jenkins
To retrieve default password for the admin user, use the command:
```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

##### Nexus
To retrieve default password for the admin user, use the command:
```bash
docker exec -it nexus cat /nexus-data/admin.password
```
To create Docker private registry, create new docker-hosted repository under Nexus Configuration panel and set the HTTP port in <code>Repository Connectors</code> to <code>8100</code>.  
After that, you can push your Docker images like below:
```bash
docker tag {image} docker.{your_domain}/{image}
docker login -u {nexus_user} docker.{your_domain}
docker push docker.{your_domain}/{image}
docker logout docker.{your_domain}
```
Example usage:
```bash
docker tag mongo:4.2 docker.example.com/mongo:4.2
docker login -u admin docker.example.com
docker push docker.example.com/mongo:4.2
docker logout docker.example.com
```

##### SonarQube
Default username and password is <code>admin</code> / <code>admin</code>

#### Additional configuration

1. To have Docker support in Jenkins:  
   https://getintodevops.com/blog/the-simple-way-to-run-docker-in-docker-for-ci
   ```bash
   docker exec -it -u root jenkins bash

   apt-get update && \
   apt-get -y install apt-transport-https \
       ca-certificates \
       curl \
       gnupg2 \
       software-properties-common && \
   curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
   add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) \
       stable" && \
   apt-get update && \
   apt-get -y install docker-ce
   ```

2. To have synchronization between Bitbucket and Jenkins:
  - install app "Webhook to Jenkins for Bitbucket Server" in Bitbucket
  - install plugins "Blue Ocean" and "Bitbucket Pipeline for Blue Ocean" in Jenkins

3. To have synchronization between Jira and Jenkins install app "Jenkins Integration for Jira Server" in Jira
