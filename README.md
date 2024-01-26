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

Install Minikube, kubectl tool as per below link

https://www.linuxbuzz.com/install-minikube-on-ubuntu/

### Install and Configure ArgoCD

https://argo-cd.readthedocs.io/en/stable/getting_started/

```
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
```
kubectl get pods -n argocd (wait until all pods starts running)
```

![1](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/71712148-0c97-496c-a158-bdd2f6c8c1f4)

```
kubectl get svc -n argocd 
```
Once all the pods running, change the service type from "ClusterIP" to "Nodeport" for the SVC - "argocd-server" to access Argo CD UI

```
kubectl edit svc argocd-server -n argocd
kubectl get svc -n argocd 
```

![2](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/37203a7c-d0b6-4f0e-9c07-1730a805aee1)

```
minikube service list -n argocd
minikube service argocd-server -n argocd
```
![3](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/6c155ad8-832b-4f68-9374-2daefe7e902e)

As we are using Nodeport service we can access ArgoCD UI in browser
Now copy the ip-address of "argocd-server" and paste it in the browser

![4](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/39b616df-2630-4e04-854c-76e66f384515)

Username is 'admin' and for password

```
kubectl get secret
kubectl edit secret example-argocd-cluster
```
Copy the encoded secret and decode it with below command and use it as password

```
echo <encoded-secret> | base64 -d
```


![5](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/60f253be-eba2-4501-9319-61326b002630)


Create Application 

![6](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/f4e3d0fb-481e-4c2a-bc88-9e98c8191139)


Now Build the Jenkins Job and wait for the build process to complete.
Monitor the pipeline stages and fix any issues that arise.

![jenkins-job-build-history](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/d0416fbe-d3dc-4776-9765-b8a5a1a6536d)

Once the build is successful, go to the sonarqube server and check the projects results.

![Sonarqube](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/57e687e4-ace0-48b5-8038-d6f79afea5b4)

Now go to Docker Hub Repository and check for the pushed image

![7](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/80c538c1-90a4-4cd3-b338-181fb91a56b7)

Now go to Argo CD server, refresh the page and check deployments.

![Argocd](https://github.com/Jyothidk/Jenkins-to-Kubernetes/assets/127189060/c66e92f3-4b55-43d6-9ff5-5a76e647bc1a)









