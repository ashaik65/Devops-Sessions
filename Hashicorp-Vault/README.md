### Deploy Hashicorp vault on kubernetes ###

Steps to deploy :- As prequsist we need Kubernetes cluster ready for this 

Adding Helm repo :- 

$ helm repo add hashicorp https://helm.releases.hashicorp.com
"hashicorp" has been added to your repositories

But for best practice we need to change thing in values.yaml and install
First search repo using command 

$ helm search repo vault --versions
```` yaml

NAME           	CHART VERSION	APP VERSION	DESCRIPTION                               
hashicorp/vault	0.23.0       	1.12.1     	Official HashiCorp Vault Chart            
hashicorp/vault	0.22.1       	1.12.0     	Official HashiCorp Vault Chart            
hashicorp/vault	0.22.0       	1.11.3     	Official HashiCorp Vault Chart            
hashicorp/vault	0.21.0       	1.11.2     	Official HashiCorp Vault Chart            
hashicorp/vault	0.20.1       	1.10.3     	Official HashiCorp Vault Chart            
hashicorp/vault	0.20.0       	1.10.3     	Official HashiCorp Vault Chart            
hashicorp/vault	0.19.0       	1.9.2      	Official HashiCorp Vault Chart            
hashicorp/vault	0.18.0       	1.9.0      	Official HashiCorp Vault Chart            
hashicorp/vault	0.17.1       	1.8.4      	Official HashiCorp Vault Chart            
hashicorp/vault	0.17.0       	1.8.4      	Official HashiCorp Vault Chart            
hashicorp/vault	0.16.1       	1.8.3      	Official HashiCorp Vault Chart            
hashicorp/vault	0.16.0       	1.8.2      	Official HashiCorp Vault Chart            
hashicorp/vault	0.15.0       	1.8.1      	Official HashiCorp Vault Chart            
hashicorp/vault	0.14.0       	1.8.0      	Official HashiCorp Vault Chart            
hashicorp/vault	0.13.0       	1.7.3      	Official HashiCorp Vault Chart            
hashicorp/vault	0.12.0       	1.7.2      	Official HashiCorp Vault Chart            
hashicorp/vault	0.11.0       	1.7.0      	Official HashiCorp Vault Chart            
hashicorp/vault	0.10.0       	1.7.0      	Official HashiCorp Vault Chart            
hashicorp/vault	0.9.1        	1.6.2      	Official HashiCorp Vault Chart            
hashicorp/vault	0.9.0        	1.6.1      	Official HashiCorp Vault Chart            
hashicorp/vault	0.8.0        	1.5.4      	Official HashiCorp Vault Chart            
hashicorp/vault	0.7.0        	1.5.2      	Official HashiCorp Vault Chart            
hashicorp/vault	0.6.0        	1.4.2      	Official HashiCorp Vault Chart            
hashicorp/vault	0.5.0        	           	Install and configure Vault on Kubernetes.
hashicorp/vault	0.4.0        	           	Install and configure Vault on Kubernetes.

you can see lots of version are there but we need to go with latest version for vault but before that we will take values.yaml for this use command

$ helm show values hashicorp/vault > values.yaml

Then now we need to install this chart

$ helm install vault hashicorp/vault -f values.yaml

NAME: vault
LAST DEPLOYED: Wed Jan 18 11:04:19 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault


now once you check resources using kubectl found that vault pod is stuck in 0/1 Running because we need to exec into the pod and unseal the keys

``` yaml

anis.shaikh@C02F60RVML7H hashicorp-vault % kubectl get all
NAME                                        READY   STATUS         RESTARTS        AGE
pod/vault-0                                 0/1     Running             0          35m
pod/vault-agent-injector-77fd4cb69f-npbl9   0/1     ContainerCreating   0          35m

NAME                               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/kubernetes                 ClusterIP   10.96.0.1        <none>        443/TCP             11d
service/vault                      ClusterIP   10.100.90.213    <none>        8200/TCP,8201/TCP   35m
service/vault-agent-injector-svc   ClusterIP   10.107.152.178   <none>        443/TCP             35m
service/vault-internal             ClusterIP   None             <none>        8200/TCP,8201/TCP   35m
service/vault-ui                   ClusterIP   10.107.61.141    <none>        8200/TCP            35m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/vault-agent-injector   1/1     1            1           35m

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/vault-agent-injector-77fd4cb69f   1         1         1       35m

NAME                     READY   AGE
statefulset.apps/vault   0/1     35m
````

### Initialising Vault ###
As in previous step we see that vault is stuck in Run
```` yaml

kubectl exec -it vault-0 -- sh

## once exec into the pod we need to fire vault commands

vault operator init ----> to initialised vault

vault operator unseal ----> to unseal the secret (we need to fire this commnad  atlist 3 times for unseal)

kubectl -n vault exec -it vault-0 -- vault status   ----> check unsealing status

### Now we will login to UI ###

kubectl vault get svc
kubectl port-forward svc/vault-ui 3000:8200  # Note:- we can expose ui via Ingress,NodePort,LoadBalancer 
````
### Enable Kubernetes Authentication ###

For the injector to be authorised to access vault, we need to enable K8s auth
````yaml

kubectl exec -it vault-0 -- sh  

vault login
Token (will be hidden) We need to put root token at the time of unseal 

Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                hvs.bjaRGRjTh0XLY075dBxoiXVR
token_accessor       DwAwvmL3Z4hTwfrlWmLHPBF8
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]

vault auth enable kubernetes ---> enable the kubernetes auth

Success! Enabled kubernetes auth method at: kubernetes/
## Now we need write auth configuration
vault write auth/kubernetes/config \
token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
kubernetes_host=https://${KUBERNETES_PORT_443_TCP_ADDR}:443 \
kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
issuer="https://kubernetes.default.svc.cluster.local"

Success! Data written to: auth/kubernetes/config

## now check in vault UI ---> Authentication Methods ----> Auth Methods ----> we can see kubrenetes auth
````

### Basic Secret Injection ###

In order for us to start using secrets in vault, we need to setup a policy.

````yaml

#Create a role for our app

kubectl -n vault exec -it vault-0 -- sh 

vault write auth/kubernetes/role/basic-secret-role \
   bound_service_account_names=basic-secret \
   bound_service_account_namespaces=example-app \
   policies=basic-secret-policy \
   ttl=1h

Success! Data written to: auth/kubernetes/role/basic-secret-role

# Now we need to go to vault UI again see this role has been created or not
The above maps our Kubernetes service account, used by our pod, to a policy. Now lets create the policy to map our service account to a bunch of secrets

kubectl -n vault exec -it vault-0 -- sh 

cat <<EOF > /home/vault/app-policy.hcl
path "secret/basic-secret/*" {
  capabilities = ["read"]
}
EOF
vault policy write basic-secret-policy /home/vault/app-policy.hcl

Success! Uploaded policy: basic-secret-policy

# now we can go to vault UI and see whether this policy is create or not

Now our service account for our pod can access all secrets under secret/basic-secret/* Lets create some secrets.

kubectl -n vault exec -it vault-0 -- sh 

vault secrets enable -path=secret/ kv

Success! Enabled the kv secrets engine at: secret/

vault kv put secret/basic-secret/helloworld username=dbuser password=sUp3rS3cUr3P@ssw0rd # in this steps we are putting secrets.

Success! Data written to: secret/basic-secret/helloworld

# Now we need to deploy our application with current requriments
Lets deploy our app and see if it works:

kubectl create ns example-app

kubectl apply -f deployment.yaml -n example-app 

kubectl -n example-app get pods

Once the pod is ready, the secret is injected into the pod at the following location:

kubectl -n example-app exec <pod-name> -- sh -c "cat /vault/secrets/helloworld"









