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
$appName="1384microservices"
$location=westeurope
az group create --name $appName --location $location
```

### Create Azure Cosmos DB account
```powershell
$appName="1384microservices"
az cosmosdb create --name $appName --resource-group $appName --kind MongoDB --enable-free-tier
```

### Create Azure Service Bus namespace
```powershell
$serviceBusNamespaceName = "dotnet1384microservices"
$resourceGroupName="1384microservices"
az servicebus namespace create --name $serviceBusNamespaceName --resource-group $resourceGroupName --sku Standard

```