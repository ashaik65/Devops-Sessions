```yaml
Install AZ Cli:-
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
Documents Refernce:- https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt

Install Kubectl Cli using AZ Cli:-
az aks install-cli
Docuement Refernce:- https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli

For connecting to kubernetes cluster need to login using az cli command

az login ----> paste link in browser and it will ask code put code from cli and give permission then it will login sucessfully

Intstall Helm:-
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

