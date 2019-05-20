## Dev Server

Installer of applications for software development.  
It includes:
- Atlassian Jira
- Atlassian Bitbucket
- Atlassian Confluence
- Jenkins
- Nexus Repository Manager
- SonarQube

You must have licenses for Atlassian products!  
Jenkins, Nexus and SonarQube are in community version.

#### Guide to install the software

1. Prepare server with Ubuntu 18 (tested with Ubuntu 18.04)
2. Connect via SSH to your server as root user
3. Install Ansible
   ```bash
   sudo apt-get update
   sudo apt-get install software-properties-common
   sudo apt-add-repository --yes --update ppa:ansible/ansible
   sudo apt-get install ansible
   ```
4. Pull the GIT repository (master branch)
5. Create private and public key for user admin (use PuttyGen or similar software)
6. Copy the public key into the <code>installer/roles/users/files</code> as <code>admin_key.pub</code>  
   Ensure the content is similar to
   ```
   ssh-rsa AAAAB3...36== admin
   ```
7. Open <code>installer/vars/all.yml</code> file and change the content, especially:
   - domain
   - admin_email
   - database passwords
8. Enter directory <code>installer</code>
9. Install Ansible roles
   ```bash
   ansible-galaxy install -r requirements.yml
   ```
10. Run installer and set up admin password
   ```bash
   ansible-playbook site.yml -i production --extra-vars "adminpassword=<admin password here>"
   ```

After installation, you should have accessible following applications:

| Application                        	| Address                        	|
|------------------------------------	|--------------------------------	|
| Atlassian Jira                     	| https://jira.example.com       	|
| Atlassian Bitbucket                	| https://bitbucket.example.com  	|
| Atlassian Confluence               	| https://confluence.example.com 	|
| Jenkins                            	| https://jenkins.example.com    	|
| Nexus Repository Manager           	| https://nexus.example.com      	|
| SonarQube                          	| https://sonar.example.com      	|
| Private Docker Registry (by Nexus) 	| https://docker.example.com     	|


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
