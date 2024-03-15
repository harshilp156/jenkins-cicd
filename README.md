
# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD and Kubernetes 

**Architecture**
![228301952-abc02ca2-9942-4a67-8293-f76647b6f9d8](https://github.com/harshilp156/jenkins-cicd/assets/67538347/1d47e39c-1d54-4cf4-8879-b640e4040472)

Created fully automated CI/CD pipeline to automate the process of building, testing, and deploying of Java based application using Jenkins, Maven, Sonarqube, Argo CD and Kubernetes.

1. Jenkins Setup: Used an EC2 t2.large instance running Ubuntu to host Jenkins..

2. SonarQube Integration: SonarQube for code quality analysis.



3. Docker Integration: Integrated Docker for containerization, streamlining application deployment across environments.



4. Minikube & ArgoCD Setup: Minikube and ArgoCD to deploy Kubernetes cluster management and GitOps-based deployment.



5. Pipeline Configuration: Configured Jenkins pipelines to fetch source code from GitHub, build Docker containers, perform SonarQube analysis, and deploy to Kubernetes via ArgoCD.

- Setup one EC2 t2.large ubuntu instance for the jenkins setup.

![Screenshot (584)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/15528236-2705-4a04-9c64-f44201fee2d7)

## Install Jenkins

Pre-Requisites:
- Java(sdk)

Run the below commands to 

Install Java
```bash
sudo apt update
sudo apt install openjdk-11-jre
java -version
```
Now we can proceed with jenkins.

Install Jenkins
```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

**Note** :  By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules.

Login to Jenkins using the below URL:
http://:8080 [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

After login to Jenkins, Run the command to copy the Jenkins Admin Password 
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

<img width="1291" alt="215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3" src="https://github.com/harshilp156/jenkins-cicd/assets/67538347/3db76e2c-602d-4cca-b583-b1ec08b8fa4a">

After entering the password we can now Install suggested plugins.

Wait for the Jenkins to Install suggested plugins.

Create First Admin User or Skip the step.

## Install the Docker Pipeline and SonarQube Scanner plugin in Jenkins:

Log in to Jenkins.

Go to Manage Jenkins > Manage Plugins.

In the Available tab, search for "Docker Pipeline".

Select the plugin and click the Install button.

Restart Jenkins after the plugin is installed.

## Created a free style project.

Provide the git as SCM for pipeline script along with the JenkinsFile path. 

- I have used docker conatiner as agent.

## Sonar Qube Setup.

```bash
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Now SonarQube Server is accessible on ```bash http://<ip-address>:9000 ```

- Generate the SonarQube token and add it to the jenkins.

## Docker slave Setup.

- Run the below command to Install Docker

```bash
sudo apt update
sudo apt install docker.io
```

- Grant Jenkins user and Ubuntu user permission to docker deamon.

```bash
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once we are done with the above steps, it is better to restart Jenkins.
```bash 
http://<ec2-instance-public-ip>:8080/restart
```
## Minikube setup

Minikube : Minikube quickly sets up a local Kubernetes cluster on macOS, Linux, and Windows. 

Minikube and ArgoCD to deploy Kubernetes cluster management and GitOps-based deployment.

currently I am using windows laptop so I have set up the windows subsystem for Linux.

## Install WSL command

```bash
wsl --install
```

- Now we can install minikube by following the below commands.

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
- After this we can install ArgoCD

We can refer below link for ArgoCD installation.

https://operatorhub.io/operator/argocd-operator

## Dockerhub and Github Credentials 

- Now we have to add Dockerhub and Github Credentials to the jenkins so that it can fetch and push the docker container along with Github so that it can make changes to the deployment file.

- Again it is highly recommended to restart the jenkins.

- Make the necessary changes in the deployment file particularly sonarqube url and github so that the jenkins file runs smoothly and it will change the image tag(inside deployment.yml) automatically each time the CI/CD pipeline executes and push the changes to the github.

![Screenshot (583)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/b44d2981-1552-48c6-83e9-260bb87f6995)

![Screenshot (581)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/db3310ee-a6c6-454b-a107-cc15fac87acd)

![Screenshot (582)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/4844f9ad-a504-47a5-a626-eb3ce447d507)

## ArgoCD setup

```bash
vim argocd.yml
```

```bash
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```
```bash
kubectl get all
kubectl apply -f argocd.yml
kubectl get pods
kubectl get svc
```

- Now change the argocd server's service type from cluster type to the node port so it can be accessible using ip address.

- We can get the argocd password from argocd cluster secret file, it has base64 encryption so we have to decrypt first.

After that we can create the deployment application in argoCD by providing repository URL ,path, kubernetes cluster as destination and namespace as default.

Here as we can see the entire CI/CD pipeline using Jenkins , maven, sonarqube, minikube, argoCD and K8S.

![Screenshot (585)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/1d447cdc-69b2-4e82-bd3a-5ba31520c3b0)

![Screenshot (586)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/f1490972-edd4-48f6-923c-0e159b33e51f)

![Screenshot (587)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/b7c61861-2ced-4aed-9786-b2f4e1e564b3)

![Screenshot (588)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/4d327804-88bb-4861-ad18-b483ad4d6f15)

![Screenshot (580)](https://github.com/harshilp156/jenkins-cicd/assets/67538347/119f8c51-6864-47f6-8034-f60294232f4a)

## Tech Stack

**AWS:** EC2

**Docker**

**GIT**

**Kubernetes**

**Maven**

**SonarQube**

**Minikube**

**ArgoCD**

**Java** **HTML** **CSS** **VIM**

