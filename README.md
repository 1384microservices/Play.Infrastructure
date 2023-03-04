# Play Economy infrastructure
Play Economy game development infrastructure

## About
This repository stores Play Economy game infrastrcuture build scripts and files.
It installs following services:
* [Mongo DB](https://www.mongodb.com/)
* [Rabbig MQ](https://www.rabbitmq.com/)

##
### Prerequisites
* Install [winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/)
* Install git: `winget install --id Git.Git --source winget`
* Install docker[^wsl]: `winget install --id Docker.DockerDesktop`

### Clone source
Create a project folder on your box. **D:\Projects\Play Economy** is a good idea but you can choose whatever fits your needs. For Windows boxes you have to issue this command in a powershell window: `New-Item -ItemType Directory -Path 'D:\Projects\Play Economy'`. Switch to this directory: `Set-Locatin -Path 'D:\Projects\Play Economy'`. 

Clone this repository to your box: `git clone https://github.com/1384microservices/Play.Infrastructure.git`.

### Spin-up infrastructure containers
Navigate to `./src` folder from the root path of this repository clone and issue this command: `docker-compose up`.

For the first time it will take a while to download required docker images. After all images are downloaded you will see MongoDB container logs on terminal.

To stop the infrastructure you need to press `CTRL+C`.

To start the infrastructure stack without hanging the terminal window and flood it with containers output you can build the infrastructure in detached mode: `docker-compose up -d`. If you've built the infrastructure stack in detached mod you need to issue `docker-compose down` command.