

- https://blog.devstream.io/posts/devsecops-jenkins-in-k8s-with-secret-scan/

- https://github.com/GoogleContainerTools/kaniko






##### Jenkins Setup on EKS Cluster
```jenkins-eks
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
helm install jenkins --namespace jenkins --create-namespace jenkinsci/jenkins


# Set up port forwarding to the Jenkins UI from Cloud Shell
kubectl -n jenkins port-forward svc/jenkins --address 0.0.0.0 8090:8080

# Jenkins Secrets

kubectl get secrets -n jenkins 

kubectl get secrets/jenkins -n jenkins -o yaml

jenkins-admin-password: b1NuY1NpWmdnR25ZemtuNWx5Mnl0NQ==
jenkins-admin-user: YWRtaW4=

echo -n 'b1NuY1NpWmdnR25ZemtuNWx5Mnl0NQ==' | base64 -d
oSncSiZggGnYzkn5ly2yt5
```



### Jenkins Commands

##### k8 credentials
```k8-secrets
kubectl create secret docker-registry docker-credentials --docker-username=[userid] --docker-password=[Docker Hub access token] --docker-email=[user email address] --namespace jenkins
```


### Jenkins Service Account
```k8-secrets
kubectl create serviceaccount jenkins --namespace=jenkins
kubectl describe secret $(kubectl describe serviceaccount jenkins --namespace=jenkins | grep Token | awk '{print $2}') --namespace=jenkins
kubectl create rolebinding jenkins-admin-binding --clusterrole=admin --serviceaccount=jenkins:jenkins --namespace=jenkins
kubectl get serviceaccounts -n jenkins
```
- Note: jenkins:jenkins serviceaccount:namespace


##### Jenkins Kaniko CI & HELM CD Pipe
```jenkins-pipe
kubectl create ns hello
kubectl create rolebinding jenkins-admin-binding --clusterrole=admin --serviceaccount=jenkins:default --namespace=hello
```


##### HELM CD deployment to AWS EKS
```
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_DEFAULT_REGION=us-west-2
```

#####  We need to create token in git but select as (fine grained token)and assign to particular repo and provide necessary permissions
1. create secret for jenkins-kaniko repo as smarttech-ci as fine grained token and provide access to jenkins-kaniko repo and paste that credentials in manged jenkins and give named as REPO_CREDS_ID
2. create one more secret named as  smarttech-cd as fine grained token and provide access to k8-helm repo  and paste that credentials in manged jenkins and give named as HELM_REPO_CREDS_ID

#####  Now we cicd from anothe pipeline but before that we need to again configure some creds in jenkins 

but for this we need to create one more token from github

now will create tokens in jenkins as HELM_REPO_CREDS_ID, AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEY,AWS_DEFAULT_REGION

