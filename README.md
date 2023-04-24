# Play Economy infrastructure
Play Economy game development infrastructure

## About
This repository stores Play Economy game infrastrcuture stack scripts which creates a [MongoDB](https://www.mongodb.com/) database service, a [RabbitMQ](https://www.rabbitmq.com/) message brocker service and a network to attach to, named **pe-network**, network that will be used by other Play Economy services to communicate with infrastructure services or the rest of Play Economy services.

## Prerequisites
* Install [winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/)
* Install git: `winget install --id Git.Git --source winget`
* Install docker[^wsl]: `winget install --id Docker.DockerDesktop`

## Clone source
First, you need to create a workspace folder and then switch to that workspace folder. **D:\Projects\PlayEconomy** can be a good idea but you should change to whatever fits your needs. Than, you have to clone this repository to the workspace you've just created: 

```powershell 
New-Item -ItemType Directory -Path 'D:\Projects\Play Economy'
Set-Locatin -Path 'D:\Projects\Play Economy'
git clone https://github.com/1384microservices/Play.Infrastructure.git
```

## Spin-up infrastructure resources
You must navigate to **src** folder within this repository clone and spin-up infrastructure services:
```powershell
cd Play.Economy\src\
docker-compose up
```
For the first time it will take a while to download required docker images. Keep in mind that this command will hang your terminal.

If you want to spin-up the stack without terminal hanging you need to start infrastructure containers in detached mode:
```powershell
cd Play.Economy\src\
docker-compose up -d
```
After all images are downloaded you will see MongoDB and RabbitMQ containers logs on terminal. 

To stop the infrastructure containers you need to press `CTRL+C`.

## Tear down infrastructure resources
Whenever possible, docker-compose tries to spin-up existing resources like containers, networks, volumes and so on. It creates new resources only when required ones are not in place. That's why, `CTRL+C` won't destroy created resources, it will only stop running containers.
```powershell
# Destroy all stack resources
docker-compose down
```

## Consume Play Economy private nuget packages
Play Economy nuget packages are stored in github repository. You need to setup a new nuget source to download these packages on your machine:
```powershell
$owner="1384microservices"
$gh_pat="[type here your PAT]"
dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

## Create Azure infrastructure
### Create Azure resource group
```powershell
$appName="playeconomy1384"
$location='northeurope'
az group create --name $appName --location $location
```

### Create Azure Cosmos DB account
```powershell
$appName="playeconomy1384"
az cosmosdb create --name $appName --resource-group $appName --kind MongoDB --enable-free-tier
```

### Create Azure Service Bus namespace
```powershell
$appName="playeconomy1384"
az servicebus namespace create --name $appName --resource-group $appName --sku Standard
```

### Create Azure Container Registry
```powershell
$appName="playeconomy1384"
az acr create --name $appName --resource-group $appName --sku Basic
```

### Publish Docker images to ACR
```powershell
$appName="playeconomy1384"
az acr login --name $appName
$repositoryUrl="${appName}.azurecr.io"
$localImages = "play.catalog", "play.identity", "play.inventory", "play.trading"
for($i = 0; $i -lt $localImages.Length; $i++) {
    $currentImageName = "$($localImages[$i]):latest"
    $taggetImageName = "${repositoryUrl}/$($localImages[$i]):latest"
    docker tag $currentImageName $taggetImageName
    docker push $taggetImageName
}
```

### Create AKS cluster
```powershell
az extension add --name aks-preview

az feature register --name "EnableWorkloadIdentityPreview" --namespace "Microsoft.ContainerService"

# Query existing features. Wait for this command to reach "Registered" state.
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableWorkloadIdentityPreview')].{Name:name,State:properties.state}"

az provider register --namespace Microsoft.ContainerService

$appName="microservices1384"
az aks create --name $appName --resource-group $appName --node-vm-size Standard_B2s --node-count 2 --attach-acr $appName --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys

az aks get-credentials --name $appName --resource-group $appName

# Stop cluster.
# This does not work, seems to be an issue but worth to keep in mind.
az aks nodepool scale --name nodepool1 --cluster-name $appName --resource-group $appName --node-count 0

# Stop cluster.
# This will take a while.
az aks stop -g $appName -n $appName

# Start cluster.
# This will take a while.
az aks start -g $appName -n $appName

# Get K8S client and server version.
# This helps you to check cluster status.
kubectl version --short
```