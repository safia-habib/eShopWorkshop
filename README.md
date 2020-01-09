# eShop Workshop
## Summary
This workshop is built with the intention of demonstrating the use of Dynatrace during a migration of a .NET monolith application into a .NET microservice application.  
The exercises in this workshop are making use of the existing Microsoft demo application named eShop packaged in a monolith [eShopOnWeb](https://github.com/peterhack/eShopOnWeb/) and containerized microservice architecture [eShopOnContainers](https://github.com/peterhack/eShopOnContainers/).  In addition the workshop is run within Azure.  
## Pre-Requisites
- Azure account with ability to create and manage virtual machines (VMs) as well as a 3-node Azure Kubernetes Cluster (AKS)
- Ability to run Azure Resource Manager (ARM) templates from within the Azure CLI

## Step 0 - Environment validation and navigation
### Azure
log into Azure Portal 
- https://portal.azure.com
- open Azure Bash CLI [show icon]
  > run command: /> git clone https://github.com/peterhack/eShopOnWeb.git
### Dynatrace
Create a Dynatrace trial tenant (if don't already have one)
 > [https://dynatrace.com/trial](https://dynatrace.com/trial)
- log into Dynatrace tenant
  > [collect tenant URL/ID]
  >
  > [get API token]
  >
  > [get PaaS token]
## Step 1 - The Monolith
### deploying a VM
Using the ARM template to deploy eShopOnWeb Monolithic .NET application with Dynatrace Extension applied
- this template will request a password for the eshopadmin user and the MSSQL DB admin
  - the DB Admin user will require 8 alpha chars, 1+ number, 1+ special chars.
- this ARM template will request your Dynatrace tenant URL, API token, & PaaS token
- in Bash CLI:
    > cd eShopOnWeb
    >
    > az group deployment create --name "eshopvm" --resource-group "eshopworkshop" --template-file "./src/VM/azuredeploy.json"

This takes ~10 mins to complete

-  SSH to VM using eshopadmin@[vm-ip-address]
  > tail /var/log/cloud-init-output.log 
- last line should be
  > "Cloud-init v. 19.3-41-gc4735dd3-0ubuntu1~18.04.1 finished..."

### Reviewing eShopOnWeb App
- confirm eShopOnWeb is running
  > docker ps 
-  open browser and navigate to: 
    > https://[vm-ip-address]:5106

### Reviewing eShopOnWeb App in Dynatrace
navigate around Dynatrace to review app

### Add front-end to eShopOnWeb App (deployed by default)
add nginx front end
  > cd /home/eshopadmin/eShopOnWeb/src/VM/
  >
  >  ./start-frontend.sh

### Review eShopOnWeb App in Dynatrace to see RUM
[add content exercises here]

### Create a custom service to expose 'orders'
as this app wasn't designed to strangle out orders, this is where we make the leap... 

restart the MVC container, run some transactions and investigate the service flow including the new 'orders' custom service

## Step 2 - Microservices
### creating an AKS deployment
> az group create --name eshopworkshop --location eastus

> az aks create --resource-group eshopworkshop --name eshopaks --node-count 3 --enable-addons monitoring,http_application_routing --generate-ssh-keys
### Deploy the OneAgent Operator in AKS
https://www.dynatrace.com/support/help/shortlink/kubernetes-deploy#install-oneagent-operator

> kubectl create namespace dynatrace
> LATEST_RELEASE=$(curl -s https://api.github.com/repos/dynatrace/dynatrace-oneagent-operator/releases/latest | grep tag_name | cut -d '"' -f 4)
> kubectl create -f https://raw.githubusercontent.com/Dynatrace/dynatrace-oneagent-operator/$LATEST_RELEASE/deploy/kubernetes.yaml
> kubectl -n dynatrace logs -f deployment/dynatrace-oneagent-operator
>kubectl -n dynatrace create secret generic oneagent --from-literal="apiToken=[API_TOKEN](https://www.dynatrace.com/support/help/reference/dynatrace-concepts/what-is-an-access-token/)" --from-literal="paasToken=[PAAS_TOKEN](https://www.dynatrace.com/support/help/technology-support/cloud-platforms/kubernetes/installation-and-operation/full-stack/deploy-oneagent-on-kubernetes/#expand-1380how-to-get-your-paas-token)"
### Deploy the eShopOnContainers App
- clone [eShopOnContainers](https://github.com/peterhack/eShopOnContainers) to Azure CLI dir

> cd ./eShopOnContainers/deploy/k8s/helm/
>
> kubectl create namespace eshop
>
> ./deploy-all.sh --aks-name eshopaks --aks-rg eshopworkshop --tag dev --dns aks --namespace eshop
>
> kubectl apply -f aks-httpaddon-cfg.yaml
>
> kubectl delete pod $(kubectl get pod -l app=addon-http-application-routing-nginx-ingress -n kube-system -o jsonpath="{.items[0].metadata.name}") -n kube-system

you can get the DNS name of your aks cluster with the following:
> kubectl get ing -n eshop

 You can view the MVC client at http://[dns]/webmvc





