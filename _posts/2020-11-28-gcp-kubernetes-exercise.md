---
layout: post
title: GCP Kubernetes Exercise
category: [gcp]
tags: [gcp, cloud, iac, terraform, kubernetes]
date: 2020-11-28
---

![The workloads in the GKE cluster](/img/2020-11-28-gcp-kubernetes-exercise_img_1.png)

*The workloads in the GKE cluster.*

### Introduction

I have earlier used Terraform to create Kubernetes in AWS [EKS](https://aws.amazon.com/eks/) (Amazon Elastic Kubernetes Service) and Azure [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/) (Azure Kubernetes Service) and therefore I wanted to create Kubernetes also in the [GCP](https://cloud.google.com/gcp/) (Google Cloud Platform) to have some perspective on how different the Kubernetes infrastructure is to create in these three major clouds.

My earlier AWS EKS and Azure AKS blog posts are here:

- [Creating Azure Kubernetes Service (AKS) the Right Way]({% post_url 2019-01-07-creating-azure-kubernetes-service-aks-the-right-way %})
- [Creating AWS Elastic Container Service for Kubernetes (EKS) the Right Way]({% post_url 2019-01-18-creating-aws-elastic-container-service-for-kubernetes-eks-the-right-way %})

This GCP GKE Kubernetes exercise can be found in my [gcp](https://github.com/karimarttila/gcp) repo in directory [simpleserver-kube](https://github.com/karimarttila/gcp/tree/kube-exercise/simpleserver-kube). I might, later on, continue with this exercise - creating a [Helm](https://helm.sh/) chart for the Clojure simple server to be deployed to this GKE cluster.

### Initialization Scripts

You can create the solution in many ways. You could have a [GCP Project](https://cloud.google.com/storage/docs/projects) made for you and you have to use that specific project. I didn't have those kinds of restrictions when making this exercise. In this exercise, I used Admin project / Infra project pattern. I.e. I create an Admin project just for hosting the [GCP Service Account](https://cloud.google.com/iam/docs/service-accounts) - the Service Account is used by Terraform to create the actual Infra project and all resources belonging to that infra project. Let's walk through these steps (one-time initialization - later on, you just use Terraform to develop the solution).

In the [init](https://github.com/karimarttila/gcp/tree/kube-exercise/simpleserver-kube/init) directory you can find a few scripts that I used to automate the admin project infrastructure. I gathered all relevant information I needed into the [env-vars-template.sh](https://github.com/karimarttila/gcp/blob/kube-exercise/simpleserver-kube/init/env-vars-template.sh) file - one should copy-paste this template e.g. to ~/.gcp/my-kube-env-vars.sh and provide the values for the environment variables. Not all environment variables are needed, I created a bunch of them, some are for administrational purposes and used to label resources (optional), but some are needed by GCP (billing information) and to create the resources (admin / infra project id...).

Then you are ready to create the Admin project and related resources: [create-admin-project.sh](https://github.com/karimarttila/gcp/blob/kube-exercise/simpleserver-kube/init/create-admin-project.sh). This script just uses the environment variables and creates the admin project, then sets certain configuration values and creates a [gcloud](https://cloud.google.com/sdk/gcloud) configuration for the admin project. Finally, we link the billing account to this admin project and enable `container.googleapis.com` so that the Service Account that belongs to this admin project and is used by Terraform can, later on, create GKE cluster.

We are going to store Terraform state in the GCP [Cloud Storage Bucket](https://cloud.google.com/storage/docs/json_api/v1/buckets) therefore we need to create it: [create-admin-bucket.sh](https://github.com/karimarttila/gcp/blob/kube-exercise/simpleserver-kube/init/create-admin-bucket.sh).

Finally, we are ready to create the last admin project resource: the Service Account that is used by Terraform to create the resources in the infra project side: [create-service-account.sh](https://github.com/karimarttila/gcp/blob/kube-exercise/simpleserver-kube/init/create-service-account.sh). The script also binds certain roles to that Service Account - e.g. the role needed to create the infra project.

The last thing to create is the infra project gcloud configurations. Since we already know the infra project id we are going to use let's create the infra configuration right now: [create-infra-configuration.sh](https://github.com/karimarttila/gcp/blob/kube-exercise/simpleserver-kube/init/create-infra-configuration.sh). This way the gcloud infra configuration is ready when we start creating the infra resources - we can use this gcloud configuration to examine the resources with the gcloud cli.

### Terraform Solution

I have often used a kind of "Mother Module" Terraform pattern, e.g. [env-def.tf](https://github.com/karimarttila/aws/blob/master/simple-server-eks/terraform/modules/env-def/env-def.tf) in one of my previous exercises. But this autumn I and my colleague Kimmo Koskinen created an AWS Terraform solution to be used internally in Metosin cloud training and also in our AWS projects and - my colleague Kimmo Koskinen suggested an "Independent Terraform States by Module" solution which is quite nice. It provides a nice way to create independent Terraform states per module and you can create and develop the modules independently as the name suggests. We wrote a blog post regarding this work: [Terraform ECS Example](https://www.metosin.fi/blog/terraform-ecs-example/). So, I used this "Independent Terraform States by Module" solution in this GCP GKE exercise. There are basically three independent Terraform modules in the [terraform](https://github.com/karimarttila/gcp/tree/kube-exercise/simpleserver-kube/terraform) directory:

- [project](https://github.com/karimarttila/gcp/tree/kube-exercise/simpleserver-kube/terraform/project)
- [vpc](https://github.com/karimarttila/gcp/tree/kube-exercise/simpleserver-kube/terraform/vpc)
- [kube](https://github.com/karimarttila/gcp/tree/kube-exercise/simpleserver-kube/terraform/kube)

All modules comprise a `setup.tf` file that includes the Terraform google provider and the state configuration. All modules comprise also `main.tf`, `variables.tf`, and `outputs.tf` files - respectively giving configurations for the main resources, variables used, and outputs.

The `project` and `vpc` modules just create the infra project and the vpc used in this project and are more or less trivialities. Let's spend some time with the `kube` module instead.

The GKE Terraform configuration is ridiculously small, just 60 lines. First, we create the cluster itself and then the nodes used by the cluster. The simplicity and easiness of the GKE Terraform solution was a pleasant surprise. And more surprises ahead: it took just some 60 seconds for Terraform to create the GKE cluster. I remember that in our previous project creating AWS EKS using Pulumi took quite a while.

### Connecting and Testing the GKE Cluster

Use gcloud to get the credentials for the new GKE cluster:

```bash
 gcloud container clusters get-credentials YOUR-CLUSTER-NAME --region YOUR-REGION
 ```

 Then you can use [kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/) cli tool to examine the GKE cluster and its resources. Another nice tool is [Lens](https://k8slens.dev/).

 Just to make sure the cluster works properly I deployed some dummy service to it:

```bash
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
kubectl get pods --all-namespaces
kubectl expose deployment hello-server --type LoadBalancer  --port 80 --target-port 8080
kubectl get service hello-server
```

You'll get the external IP for the service and you can curl it:

```bash
λ> curl http://EXTERNAL-IP
Hello, world!
Version: 1.0.0
```

### Resource Naming

I like to name my cloud resources so that all resources have a prefix providing information regarding the project and the environment. We used this same strategy with my colleague in the project I previously mentioned. I used this same strategy in this exercise. Example:

```hcl
locals {
  workspace_name = terraform.workspace
  module_name    = "kube"
  res_prefix     = "${var.PREFIX}-${local.workspace_name}"
...

resource "google_container_cluster" "kube-cluster" {
  name                     = "${local.res_prefix}-${local.module_name}-cluster"
```

So, if the prefix is the project name (e.g. `projx`) and the Terraform workspace is e.g. `dev` all resources have a resource prefix `projx-dev`. E.g. the gke cluster name will be: `projx-dev-kube-cluster`. You can have many environments in the same GCP account using this pattern, e.g. `dev`, `qa`, `test` etc. - all environments have a dedicated Terraform state. Just to make it explicit - you should always keep your production environment in a dedicated production account.

### Patterns for Creating Environments

Since it is so easy and quick to create new environments using GKE I wouldn't simulate environments using Kubernetes namespaces in the same GKE cluster. You can do this but interacting with the other resources in the same environment would be complex (inside the Kubernetes cluster you can have a dynamic number of virtual environments but outside the Kubernetes cluster you typically have one environment, the one the cluster itself belongs to). Instead, I would just create exact copies of the environments (by parameterizing the instance sizes, of course). Read more about this pattern in my earlier blog post [Cloud Infrastructure Golden Rules](({% post_url 2019-01-07-creating-azure-kubernetes-service-aks-the-right-way %})).

### Conclusions

It was a real positive surprise how easy it is to set up a Kubernetes cluster using GCP GKE service and Terraform. Considering my experiences using Kubernetes with AWS and GCP I would recommend using the simplest solution for running containers in each cloud: with AWS, use ECS; with GCP, use GKE.


*The writer is working at Metosin using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested in getting Clojure training in Finland you can contact me by sending an email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>

