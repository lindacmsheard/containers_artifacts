# TEAM 9 Fork


# Challenge 2: Approach:

a) Boot up the SQL container

Default user might be SA
```
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>" \
   -p 1433:1433 --name sql1 -h sql1 \
   -d \
   mcr.microsoft.com/mssql/server:2017-latest
```
Test connection without going into the container - directly to exposed SQLDB

```sh
#sqlcmd -S localhost:1433 -U SA -P "<YourStrong@Passw0rd>"  ## doesn't work
sqlcmd -S 127.0.0.1 -U SA -P "<YourStrong@Passw0rd>"  ## this works!
```
> Warning: turns out that the port specification requires a komma sytnax: `-S 127.0.0.1,1433` also works. Appears that the default port that sqlcmd will look for is 1433.

b) create database

```
docker exec -it sql1 "bash"
```

then in the container
```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "WelcomeAzure23!"
```
create the database:
```
CREATE DATABASE mydrivingDB;
```
---
or

```sh
sqlcmd -S 127.0.0.1 -U SA -P "<YourStrong@Passw0rd>"
CREATE DATABASE mydrivingDB;
GO

# check database tables after data load
USE mydrivingDB;
GO

SELECT NAME from sys.tables;
GO
```
c) Data loading:
uses an open? image called data-load v1

docker network ls

```
docker run --network host -e SQLFQDN=127.0.0.1 -e SQLUSER=sa -e SQLPASS="<YourStrong@Passw0rd>" -e SQLDB=mydrivingDB openhack/data-load:v1
```

or 
```
docker inspect sql1
```
to get the *bridge* network IP Address
```
docker run --network bridge -e SQLFQDN=172.17.0.2 -e SQLUSER=sa -e SQLPASS="<YourStrong@Passw0rd>" -e SQLDB=mydrivingDB openhack/data-load:v1
```


d) build POI container
first edit Dockerfile to give the right connection info

build
```
docker build --pull --rm -f "src/poi/Dockerfile" -t poi-linda:latest "src/poi"
```

run
```
docker run -d -p 8080:80 --name poi -e "SQL_PASSWORD=<YourStrong@Passw0rd>" -e "SQL_SERVER=172.17.0.2" -e "ASPNETCORE_ENVIRONMENT=Local" poi-linda:latest
```

verify we have the image
```
docker image ls
-> 
REPOSITORY                       TAG                 IMAGE ID            CREATED             SIZE
tripinsights/userprofile         1.0                 639efbdcf8d6        33 seconds ago      210MB
```

e) login to be able to push to container registry
```
az login
-> login to hacker account
```

verify with 
```
az account show
```

login to acr:
```
az acr login --name registryhhm0245
```

label the image wih the fully qualified name so docker knows where to push to
```
docker tag tripinsights/trips:1.0 registryhhm0245.azurecr.io/trips:1.0
```
push the image
```
docker push registryhhm0245.azurecr.io/trips:1.0
```

# Challenge 3 Approach

Create an AKS cluster
- name: aks-oh
- location ukwest
- default kube version 1.20.9
- DS2 machines
- enable virtual nodes is disabled
- autoscale
- Kubenet vs Azure CNI
  - select the vnet there rg-aks-oh-vnet - new vnet - we can do private link later if the sql is in another vnet 
- Enable HTTP application routing
- Enable container monitoring
- new default log analytics workspace. 
- Policy is a separate feature we can enable afterwards
- container registry is integrated. 


Create a kubernetes secret
```sh
export RG=rg-aks-oh
export AKS_NAME=aks-oh
az aks get-credentials --resource-group $RG --name $AKS_NAME

kubectl get nodes

kubectl create namespace --help
kubectl create namespace tripinsight


#get credentials 
az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME

# get nodes
kubectl get nodes 

#Start K9s
k9s
 
# Create namespace
kubectl create namespace tripinsight

#Create secret
kubectl create secret generic mssqlpassw --namespace tripinsight --from-literal=MSSQLPASSW="xxxxx"
kubectl create secret generic mssqluser --namespace tripinsight --from-literal=MSSQLUSER="xxxxxxxxxxxxx"



#     [--from-literal=key1=value1]
```


learnings from pre-work - tripviewer deployment

```
kubectl apply -f deployment.yaml --namespace=tripinsight
```
(we started this with Tripviewer)

--or--

use the azure portal for a quick deployment

(we did this for POI and then migrating that? )

--
First test of a deployment:

connect a terminal using the vscode integration, and run the curl test, but using port 80 configured in the deployment

```bash
curl -i -X GET 'http://localhost:80/api/user' 
```


# how to test a running container
use the kubernetes extension in VSCode to attach a terminal to a running container (e.g. poi)


# CH4

a) removing secrets
```
kubectl delete secret mssqlpassw -n api
kubectl delete secret mssqluser -n api
```

b) deploying a keyvault
```
[see https://medium.com/swlh/integrate-azure-key-vault-with-azure-kubernetes-service-1a8740429bea]
az keyvault create  --tname team9-keyvault--resource-group teamResources --location ukwest
az keyvautl secret set --vault-name team9-keyvault --name mssqluser --value sqladminhHm0245
az keyvault secret set --vault-name team9-keyvault --name mssqlpassw --value uL6nj2Qo1
```


c) install helm on the bastion host
```
helm repo add ... comand from blog
helm install csi-secrets ...

helm repo add-pod identity ...

```

d) Finally crate Azure Managed Identity
we may not need this because we have already a managed identity for aks and cluster nodes. 
but this is one for the pods specifically
```
az identity list
az identity create -g teamResources -n aks2kvIdentity
```

e)  give the managed identity read access on the keyvault. 

```
clientId=`az identity show --name aks2kvIdentity --resource-group teamResources |jq -r .clientId`
principalId=`az identity show --name aks2kvIdentity --resource-group teamResources |jq -r .principalId`
subId=`az account show | jq -r .id`
az role assignment create --role "Reader" --assignee $principalId --scope /subscriptions/$subId/resourceGroups/teamResources/providers/Microsoft.KeyVault/vaults/myk8skv
az keyvault set-policy -n team9-keyvault --secret-permissions get --spn $clientId
az keyvault set-policy -n team9-keyvault --key-permissions get --spn $clientId
```



# Containers 2.0 Openhack

<!-- 
Guidelines on README format: https://review.docs.microsoft.com/help/onboard/admin/samples/concepts/readme-template?branch=master

Guidance on onboarding samples to docs.microsoft.com/samples: https://review.docs.microsoft.com/help/onboard/admin/samples/process/onboarding?branch=master

Taxonomies for products and languages: https://review.docs.microsoft.com/new-hope/information-architecture/metadata/taxonomies?branch=master
-->

This repo houses the source code and dockerfiles for the Containers OpenHack event.

The application used for this event is a heavily modified and recreated version of the original [My Driving application](https://github.com/Azure-Samples/MyDriving).

## Contents

| File/folder       | Description                                |
|-------------------|--------------------------------------------|
| `.devcontainer`   | VS Code [development container](https://code.visualstudio.com/docs/remote/containers) with useful utils (Azure CLI, Kubectl, Helm, etc.)   |
| `dockerfiles`     | Dockerfiles for source code                |
| `src`             | Sample source code for POI, Trips, User (Java), UserProfile (Node.JS), and TripViewer                     |
| `.gitignore`      | Define what to ignore at commit time.      |
| `CODE_OF_CONDUCT.md` | Code of conduct.                        |
| `LICENSE`         | The license for the sample.                |

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
