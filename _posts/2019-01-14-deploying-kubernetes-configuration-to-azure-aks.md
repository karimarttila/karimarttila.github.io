---
layout: post
title: "Deploying Kubernetes Configuration to Azure AKS"
category: [azure]
tags: [azure, kubernetes, cloud, iac]
date: 2019-01-14
---

![](/img/2019-01-14-deploying-kubernetes-configuration-to-azure-aks_img_1.png)

*The Kubernetes Deployment Configuration and the Bash Script to Tweak It.*

### Introduction

This blog post describes the second part of my Azure Kubernetes Services (AKS) studies. In the first part [Creating Azure Kubernetes Service (AKS) the Right Way]({% post_url 2019-01-07-creating-azure-kubernetes-service-aks-the-right-way %}) I created a Terraform configuration for the project infra for the Azure cloud: AKS (the Kubernetes service itself), ACR (Azure Container registry) to host the Docker images and other related cloud infra I needed for the demo application.

In this second part I describe how I created the Kubernetes deployment configuration for my Simple Server so that I’m able to use the same Kubernetes configuration to deploy both the single-node and Azure Table storage versions to both Minikube Kubernetes test cluster and to the real Azure AKS Kubernetes cluster.

You can find the project in my [Github account](https://github.com/karimarttila/kubernetes/tree/master/simple-server).

There are two dependencies for this Kubernetes exercise:

* The Simple Server Clojure project which can be found in my [Github account](https://github.com/karimarttila/clojure/tree/master/clj-ring-cljs-reagent-demo/simple-server).
* The Simple Server Docker image project which can be found in my [Github account](https://github.com/karimarttila/docker/tree/master/demo-images/simple-server/clojure).


### Kubernetes Deployment Configuration

There are two Kubernetes configuration files: “[simple-server-namespace-template.yml](https://github.com/karimarttila/kubernetes/blob/master/simple-server/simple-server-namespace-template.yml)” and “[simple-server-deployment-template.yml](https://github.com/karimarttila/kubernetes/blob/master/simple-server/simple-server-deployment-template.yml)”. The configuration files are full of “REPLACE_SOMETHING” strings so that I could use the same configuration files to deploy two different versions of the same Simple Server: the single-node test version and the table-storage version (uses Azure Table Storage as database, see my previous blog post: “[Azure Table Storage with Clojure](https://medium.com/@kari.marttila/azure-table-storage-with-clojure-12055e02985c)”). The deployment configuration files also support two Kubernetes clusters: Minikube test Kubernetes cluster and [Azure Kubernetes Service](https://azure.microsoft.com/en-us/services/kubernetes-service/) (AKS) Kubernetes cluster (which I created in my previous exercise and described in my previous blog post “[Creating Azure Kubernetes Service (AKS) the Right Way](https://medium.com/@kari.marttila/creating-azure-kubernetes-service-aks-the-right-way-9b18c665a6fa)”).

### Deployment to Minikube

[Minikube](https://kubernetes.io/docs/setup/minikube/) provides a local single node Kubernetes cluster that you can use for testing Kubernetes deployments. Install minikube using instructions in [Minikube installation instructions](https://github.com/kubernetes/minikube). You also need to install the kubectl command line tool: [kubectl installation instructions](https://kubernetes.io/docs/tasks/tools/install-kubectl/). I have earlier written some experiences using Minikube so I’m not going to reiterate those instructions again, see my blog post “[Exploring Kubernetes with Minikube](https://medium.com/@kari.marttila/exploring-kubernetes-with-minikube-c90c60b25e81)”. You can also read more detailed instructions in the project’s [README.md](https://github.com/karimarttila/kubernetes/blob/master/simple-server/README.md) file.

So, build the docker images using the ```eval $(minikube docker-env)``` command and the docker images are stored in Minikube. Then you can deploy the Kubernetes cluster to Minikube using the utility script:

```bash
./create-simple-server-deployment.sh single-node minikube 192.168.99.100 31111 0.1 dummy-acr
```

You can test the single-node version in Minikube using script [call-all-ip-port.sh](https://github.com/karimarttila/clojure/blob/master/clj-ring-cljs-reagent-demo/simple-server/scripts/call-all-ip-port.sh) (a poor man’s Robot framework — calls the various Simple Server APIs one after another):

```bash
./call-all-ip-port.sh 192.168.99.100 31111O
```

Single-node version works in Minikube. How about the table-storage version? The table-storage version uses [Azure Table Storage](https://azure.microsoft.com/en-us/services/storage/tables/) as database and needs the Table storage connection string to access the Azure Storage account. I was considering various options how to provide the connection string for the application running in a Kubernetes pod: 1. Somehow in the terraform configuration inject the secrets to AKS infra and the application reads the secrets from the environment. 2. Add the secrets to Azure Key Vault and the application reads the secrets from there. 3. Inject the secrets as Kubernetes secrets to the Kubernetes cluster and the application running in a pod reads the secrets from the Kubernetes environment. 4. Just inject the secrets from sourced environmental variables using [sed](https://www.gnu.org/software/sed/manual/sed.html) to the temporary Kubernetes deployment file which is deleted right after the deployment and the secrets do not end up into the Git repository but are safely in the home directory where they were sourced in the first place. Since this was just an exercise and my focus was in the AKS deployment and not how to handle secrets the best possible way I went with the least effort and chose option #4. After injecting the Azure Table Storage connection string into the Kubernetes deployment file and using the table-storage Docker image version the Minikube deployment went smoothly also with this Simple Server version and the test script worked just fine.

### Deployment to Azure Kubernetes Service (AKS)

Deployment to Azure AKS was pretty much the same as with Minikube, except that you need to tag the Docker images and push them to the [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) (ACR) so that AKS can pull the images from there. First login to the ACR so that you are able to push to it:

```bash
az acr login --name YOURACRNAME
#Then you need to tag the Docker images using that ACR name:
docker images # => Check the initial tags first.  
# Tag the base image first:  
docker tag karimarttila/debian-openjdk11:0.1 YOURACRNAME.azurecr.io/karimarttila/debian-openjdk11:0.1  
# And then the demo image:  
docker tag karimarttila/simple-server-clojure-single-node:0.1 YOURACRNAME.azurecr.io/karimarttila/simple-server-clojure-single-node:0.1  
docker images # => Check that you have the ACR tagged images.Then just push the images to ACR registry:
docker images # => Check the images you are about to push.  
docker push YOURACRNAME.azurecr.io/karimarttila/debian-openjdk11:0.1  
docker push YOURACRNAME.azurecr.io/karimarttila/simple-server-clojure-single-node:0.1  
# => Check that the images are safely in your Azure ACR registry.  
az acr repository list --name YOURACRNAME --output table
```

 You can easily automate this part e.g. in Jenkins.

Then you can deploy the Kubernetes configuration to Azure AKS (first set the kubectl context):

```bash
kubectl config use-context YOUR-AZURE-AKS-CONTEXT  
./create-simple-server-deployment.sh single-node azure YOUR-PUBLIC-IP 31111 0.1 kari2ssaksdevacrdemo
```

You need to use the public ip that was created in my earlier exercise for the single-node version. Use the script call-all-ip-port.sh to test the deployment:

```bash
./call-all-ip-port.sh YOUR-PUBLIC-IP 3045O
# The single-node version works. Then deploy the table-storage version:
./create-simple-server-deployment.sh azure-table-storage azure YOUR-PUBLIC-IP 31112 0.1 kari2ssaksdevacrdemoAnd use the script call-all-ip-port.sh to test the deployment again, this time using the table-storage version public IP.
```

### Kubernetes Debugging Tricks

How to get an interactive shell to a running Kubernetes pod when the pod crashes and won’t start? I had to use this trick when I was wondering why the application couldn’t find one environmental variable in the pod — and the application crashed because of that and the pod crashed as well. So, first you have to override the Docker entrypoint in the Kubernetes deployment, use e.g. the following command right after image definition in the Kubernetes deployment yml file:

```bash
command: ["/bin/sh"]  
args: ["-c", "while true; do echo hello; sleep 10;done"]
```

The idea is to override the application entrypoint defined in the Docker image with some dummy entrypoint which leaves the pod doing something and not exiting immediately (that’s why the indefinite bash while loop).

Then check the pod identifier and use the pod identifier to get an interactive bash to the pod:

```bash
kubectl get pods --namespace $MY_NS # => Get pod identifier, and use it in the next command:  
kubectl exec -it kari-ss-table-storage-deployment-86b6d498ff-zdqq2 --namespace kari-ss-table-storage-ns /bin/bash
```

Voila! You get the pod running and you can get an interactive bash session in the pod. Now you can examine the environmental variables to see what is the problem (or examine what ever you need to examine there).

### Conclusions

With little bash/sed tweaking you are able to create one Kubernetes deployment configuration and use it to deploy different versions of your application and also target to different Kubernetes clusters.

Next I’ll do the same exercise in the AWS side using EKS and Fargate. So, if you are interested to compare Azure AKS with AWS EKS, stay tuned!

*The writer has two AWS certifications and one Azure certification and is working at the [Tieto Corporation](https://www.tieto.com/) in Application Services / Application Development / Public Cloud team designing and implementing cloud native projects. If you are interested to start a new cloud native project in Finland you can contact me by sending me email to my corporate email or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila’s Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>
