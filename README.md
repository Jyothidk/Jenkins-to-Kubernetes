# Jenkins Pipeline for Java based application using Maven, SonarQube, Docker, Argo CD and Kubernetes

![Screenshot 2023-03-28 at 9 38 09 PM](https://user-images.githubusercontent.com/43399466/228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8.png)



Prerequisites:

   -  Java application code hosted on a Git repository
   -  Jenkins server
   -  Sonarqube
   -  Kubernetes cluster
   -  Argo CD

Tools Required:

   -  Java, Git, Maven, Jenkins, Docker, SonarQube, Kubernetes Cluster, ArgoCD

     
Install Jenkins, configure Docker as agent, set up cicd, deploy applications to k8s 

- Go to AWS Console and create EC2 instance t2.large

### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins

Install Java

```
sudo apt update
sudo apt install openjdk-11-jre
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

### Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

Jenkins Installation is Successful. You can now start using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">


Install the Required plugins in Jenkins

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for ´Docker Pipeliner´, ´SonarQube Scanner´
   - Select the plugins and click the Install button.
   - Restart Jenkins after the plugin is installed. (`http://<ec2-instance-public-ip-address>:8080/restart` )
   

![pipeline-plugin](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/fe9cf02f-b16d-426f-861d-56137a24a5c8)


Wait for the Jenkins to be restarted.


## Docker Slave Configuration

Run the below command to Install Docker

```
sudo apt update
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

### Configuring SonarQube server

```
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```
**Note: ** By default, SonarQube will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 9000 in the inbound traffic rules.

Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 

After login at the right top corner click profile icon -> My Account -> Security, Under Generate Token give a name and click Generate and copy the Token.

![Screenshot (202)](https://user-images.githubusercontent.com/129657174/230658495-a4ee14e9-df19-4bfa-8cec-0b9ccc3abb76.png)

### Configuring Credentials on Jenkins

For SonarQube

   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Secret text
   -  Copy the Sonarqube Token in Secret box and give name as sonarqube in ID
   -  Click Save

For Git

   -  Now go to your GitHub Account > Settings > Developer Settings > Personal access tokens > Tokens(classic) > Generate new token (classic)
   -  Give a name, Select all check boxes, Click Generate token and Copy the token for future use
   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Secret text
   -  Copy the Git Secret Token in Secret box and give name as github in ID
   -  Click Save

For DockerHub

   -  Now go to [DockerHub](https://hub.docker.com/) create a user and create a new repository with name 'spring-boot-app'
   -  Go to Manage Jenkins > Manage Credentials > System > global > Add Credentials
   -  Select Kind as Username and Password
   -  Give the username and password and give name as docker-cred in ID
   -  Click Save

### Minikube Installation 

Install and configure Minikube as per below link

https://www.linuxbuzz.com/install-minikube-on-ubuntu/





