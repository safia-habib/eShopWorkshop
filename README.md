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
  - run command: /> git clone https://github.com/peterhack/eShopOnWeb.git
### Dynatrace
Create a Dynatrace trial tenant (if don't already have one)
- https://dynatrace.com/trial
- log into Dynatrace tenant
  - [collect tenant URL/ID]
  - [get API token]
  - [get PaaS token]
## Step 1 - The Monolith
### deploying a VM
Using the ARM template to deploy eShopOnWeb Monolithic .NET application with Dynatrace Extension applied
- this template will request a password for the eshopadmin user and the MSSQL DB admin
  - the DB Admin user will require 8 alpha chars, 1+ number, 1+ special chars.
- this ARM template will request your Dynatrace tenant URL, API token, & PaaS token
- in Bash CLI:
  - cd eShopOnWeb
  - az group deployment create --name "eshopvm" --resource-group "eshopworkshop" --template-file "./src/VM/azuredeploy.json"

This takes ~10 mins to complete

1. SSH to VM using eshopadmin@[vm-ip-address]
tail /var/log/cloud-init-output.log 
- last line should be
"Cloud-init v. 19.3-41-gc4735dd3-0ubuntu1~18.04.1 finished..."

### Reviewing eShopOnWeb App
1. docker ps -- confirm eShopOnWeb is running
1. open browser and navigate to https://[vm-ip-address]:5106

### Reviewing eShopOnWeb App in Dynatrace
navigate around Dynatrace to review app

### Add front-end to eShopOnWeb App
add nginx front end
- go to /home/eshopadmin/eShopOnWeb/src/VM/
- execute ./start-frontend.sh

### Review eShopOnWeb App in Dynatrace to see RUM

### Create a custom service to expose 'orders'
as this app wasn't designed to strangle out orders, this is where we make the leap... 

restart the MVC container, run some transactions and investigate the service flow including the new 'orders' custom service

## Step 2 - Microservices
### creating an AKS deployment

### Deploy the OneAgent Operator in AKS

### Deploy the eShopOnContainers App
review how Dynatrace separates the microservices



