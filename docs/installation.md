---
title: Installation
description: >-
    Learn how to install the pipeline and its dependencies and how get started using PhyloNext for your analyses.
---

# Installation

PhyloNext was primarily developed for Linux. To run it from a command line on Windows, please use Windows Subsystem for Linux ([WSL](https://learn.microsoft.com/en-us/windows/wsl/install)).

All pipeline dependencies were encapsulated in containers (Docker and Singularity), which will be automatically downloaded with the first run of PhyloNext. Only the main dependencies must be installed manually. They include:  

- [Java](https://www.oracle.com/java/technologies/downloads/)  
- [Nextflow](https://www.nextflow.io/)  
- Container engine (either [Docker](https://docker.com/) or [Singularity](https://docs.sylabs.io/guides/latest/user-guide/))  

The following installation instruction is for Ubuntu.

## 1. Install [`Nextflow`](https://www.nextflow.io/docs/latest/getstarted.html#installation) 

Nextflow (`>=22.10.0`) requires Java 11 (or later, up to 18) to be installed.

``` bash
sudo apt-get update
sudo apt-get install default-jdk
```

Install Nextflow:

``` bash
wget -qO- https://get.nextflow.io | bash
chmod +x ./nextflow
mkdir -p ~/bin & mv ./nextflow ~/bin
```

!!! note "Conda"
    Alternatively, one can install Nextflow via [conda](https://docs.conda.io/en/latest/),
    a package and environment management system.  
    See [here](https://anaconda.org/bioconda/nextflow) for additional instructions.



## 2. Install [`Docker`](https://docs.docker.com/engine/installation/)

Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. 
Containers allow you to package an application with all of the parts it needs, such as libraries and other dependencies, 
and ship it all out as one package. This eliminates the need to worry about installing and configuring the necessary dependencies on the host system.

### Ubuntu

For more details see [the official Docker documentation](https://docs.docker.com/engine/install/ubuntu/).

``` bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo usermod -aG docker $USER
newgrp docker
```

!!! note "Docker via package manager"
    Alternatively, one can install Docker via the system's package manager (e.g. `Ubuntu Software Center`).


### Windows

It would be easier to use Docker Desktop WSL 2 backend on Windows. 
Docker Desktop is an application for Windows that provides an easy-to-use interface for managing Docker containers. 
It includes tools for building and deploying containerized applications, as well as for managing the lifecycle of containers.

To install Docker on Windows, you will need to follow these steps:  

- Download the Docker Desktop installer from the Docker website [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)  
- Double-click the installer to begin the installation process  
- Follow the prompts to complete the installation. This will include accepting the terms of the license agreement and selecting the installation location  
- Once the installation is complete, you will need to restart your computer to complete the installation process  

For more details see [the Docker Desktop documentation](https://docs.docker.com/desktop/windows/wsl/).

## 3. Download the pipeline and test it on a minimal dataset with a single command

``` bash
nextflow run vmikk/PhyloNext -r main -profile test,docker
```
