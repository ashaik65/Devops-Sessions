```yaml
# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD and Kubernetes
In this POC we need 1 master and 2 worker machine and jenkins machine separately

Controlplane
sudo -i

bash <(curl -s https://raw.githubusercontent.com/rizwan141/k8s/main/script/cluster-setup/master.sh)

Worker-node
sudo -i

bash <(curl -s https://raw.githubusercontent.com/rizwan141/k8s/main/script/cluster-setup/worker.sh)

once done take joined command from Controlplane and paste on worker node


### Jenkins Install using shell script ####$
Install Jenkins but dont install maven because we have plan to use maven as docker container

For install jenkins we need to follow below shell script i am using ubuntu 20.04 this script also work for ubuntu 22.04

vi jenkins.sh

#! /bin/bash

sudo apt update
sudo apt install openjdk-11-jre -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins


### Install sonarqube on VM ###

apt install unzip
adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start


Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 

### additionally add all tokens and secrets in jenkins credentials for auth ###

#### Install Argocd in the kubernetes cluster ####

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

## once this is done kindly change argocd-server service from ClusterIP to NodePort to expose the argocd UI
to chnage service type either use edit command use this command 
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'

also for fetching password for argocd use this command 

 kubectl -n argocd get secret argocd-initial-admin-secret \
          -o jsonpath="{.data.password}" | base64 -d; echo


