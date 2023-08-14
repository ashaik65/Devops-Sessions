pipeline {
  agent {
    kubernetes {
      yaml """
kind: Pod
spec:
  containers:
  - name: smarttech-ci
    image: python:slim
    command:
    - cat
    tty: true
  - name: smarttech-cd
    image: python:slim
    command:
    - cat
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
      - name: jenkins-docker-cfg
        mountPath: /kaniko/.docker
  volumes:
  - name: jenkins-docker-cfg
    projected:
      sources:
      - secret:
          name: docker-credentials
          items:
            - key: .dockerconfigjson
              path: config.json
"""
    }
  }
  stages {
    stage('Build with Kaniko') {
      steps {


       script {
                container('smarttech-ci') {
                    withCredentials([
               string(credentialsId: 'REPO_CREDS_ID', variable: 'REPO_PRIVATE_TOKEN'),
              ])
              {
               sh '''
                  echo "Login to CI Repo..."
                  echo ${BUILD_NUMBER}
                  echo $REPO_PRIVATE_TOKEN
                  apt update -y && apt install -y git
                  git clone https://oauth2:${REPO_PRIVATE_TOKEN}@github.com/ashaik65/jenkins-kaniko.git
                  ls

               '''
              }
            }


       }






        container(name: 'kaniko', shell: '/busybox/sh') {
          sh '''#!/busybox/sh
             ls
             pwd
             cd jenkins-kaniko/jenkins-k8-cicd-main
             ls
            /kaniko/executor --context `pwd` --destination ashaik65/kaniko:${BUILD_NUMBER}
          '''
        }



      }
    }


       stage('Deploy With Helm') {
      steps {


       script {
                container('smarttech-cd') {
                    withCredentials([
               string(credentialsId: 'HELM_REPO_CREDS_ID', variable: 'HELM_REPO_PRIVATE_TOKEN'),
               string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
               string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY'),
               string(credentialsId: 'AWS_DEFAULT_REGION', variable: 'AWS_DEFAULT_REGION'),
              ])
              {
               sh '''
                  echo "Login to CD Helm Repo..."
                  echo ${AWS_ACCESS_KEY_ID}
                  echo ${AWS_SECRET_ACCESS_KEY}
                  echo ${AWS_DEFAULT_REGION}
                  export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                  export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                  export AWS_DEFAULT_REGION=${AWS_DEFAULT_REGION}
                  echo ${BUILD_NUMBER}
                  echo $HELM_REPO_PRIVATE_TOKEN
                  apt update -y && apt install -y git curl zip unzip sudo
                  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                  unzip awscliv2.zip
                  sudo ./aws/install
                  aws --version
                  aws eks update-kubeconfig --name demo-cluster --region ${AWS_DEFAULT_REGION}
                  echo "Kubectl Installation"
                  curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
                  chmod +x ./kubectl
                  mv ./kubectl /usr/local/bin/kubectl
                  kubectl version --client
                  kubectl get nodes
                  echo "Helm Installation"
                  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
                  chmod 700 get_helm.sh
                  ./get_helm.sh
                  git clone https://oauth2:${HELM_REPO_PRIVATE_TOKEN}@github.com/ashaik65/k8-helm.git
                  ls
                  cd k8-helm/k8-helm-main
                  helm upgrade --install hello -f charts/hello/values.yaml --set image.repository=ashaik65/kaniko:${BUILD_NUMBER} --namespace hello charts/hello/ --atomic --timeout 1m25s --cleanup-on-fail
               '''
              }
            }


       }






  



      }
    }






















  }










}
