# Timeoff Management Application

![Gitlab pipeline status](https://img.shields.io/gitlab/pipeline-status/uditrpanchal/timeoff-management-application?branch=master&style=plastic)

## Table of Contents
- [Description](#description)
- [Tasks list](#tasks)
- [Hostnames](#hostname)
- [Initial Setup](#initial-setup-step-by-step)
- [Subnets](#subnets)
- [Route Tables](#route-tables)
- [Terraform configs](#terraform-configuration-structure)
- [Kubernetes configs](#kubernetes-configuration-structure-kustomize-configuration-management)
- [CI/CD Flow](#cicd-flow)

### Description
Welcome aboard fellow engineers. This is where you will find all the application code, scripts and configurations for IaC. This document will include the architecture diagram and details of infrastructure provisioning through Terraform. Gitlab CI/CD was used to setup autonomous build and deployment of application to AWS EKS cluster.

### Tasks
- [x] Architecture Diagram for the solution
- [x] Infrastructure running in AWS and deploy using IaC
- [x] Application must be deployed in fully autonomous way (CI/CD)
- [x] Application needs to be Highly Available and load balanced in atleast 2 AZ
- [ ] Application should be served using HTTPS and HTTP should be redirected to HTTPS

##### Hostname
- [x] http://aa79ca04ac3904183b874ae6adfede49-498e336b853077ec.elb.us-east-1.amazonaws.com 
- [ ] https://timeoff-management.net [ DNS not resolving, Domain registered with Route53, Certificate (ACM) ]

<br />

![aws_architecture_diagram](/public/img/AWS_Architecture.drawio.png "aws_architecture_diagram")

<p align="center">
<b>AWS EKS Cluster Architecture</b>
</p>

<br />

#### Initial Setup (Step-by-step) :
- Fork https://github.com/timeoff-management/timeoff-management-application to your hosted Git repository (Gitlab, Github, Bitbucket etc.)
- Resolve application build issues
- Containerize the timeoff-management application (Dockerfile)
- Deploy and test the application locally (Docker/Kubernetes). 
- For best practice, don't use your root user for AWS work. Create group in IAM and add user (eg: gitlab) to the group with programmatic access to perform all tasks.
- Register the domain and update the A record in hosted DNS zone for hostname redirecting to the application.
- Setup EKS cluster manually and deploy the container image.
- Create terraform configuration for each AWS component and services to setup the AWS Infrastructure.
- Create Kubernetes objects and configuration (Kustomize - configuration management).
- Create CI/CD pipeline (.gitlab-ci.yml). Store all required environment variables in Gitlab before executing the pipeline.
- Test out end-to-end flow.

<br />


#### Subnets

| Name   | Type     | CIDR Block    |
| ------------- | ------------- | -------- |
| public-us-east-1a         | public        | 192.168.0.0/18  |
| public-us-east-1b           | public         | 192.168.64.0/18  |
| private-us-east-1a         | private        | 192.168.128.0/18  |
| private-us-east-1b           | private         | 192.168.192.0/18  |

<br />

#### Route Tables

##### public ( 2 subnets )
- public-us-east-1b [192.168.64.0/18]
- public-us-east-1a [192.168.0.0/18]

##### private1
- private-us-east-1a [192.168.128.0/18]


##### private2
- private-us-east-1b [192.168.192.0/18]

<br />

#### Terraform (Provisioning AWS Cluster)

##### Terraform configuration structure
Each terraform file contains the link to the resource and comments for block


```bash
terraform/
 ├── provider.tf
 ├── vpc.tf
 ├── internet-gateway.tf
 ├── subnets.tf
 ├── nat-gateways.tf
 ├── routing-tables.tf
 ├── route-table-association.tf
 ├── eips.tf
 ├── eks.tf
 ├── eks-node-groups.tf
 └── terraform.tfstate

```
<br />

```bash
# Steps to provision AWS EKS Cluster

$ terraform init          # Downloads and install plugins for required providers
$ terraform plan          # Previews the changes (like a Dry-run)
$ terraform apply         # Applies the changes [ --auto-approve (to execute without user prompt for 'yes' confirmation)]


# Step to destroy AWS EKS Cluster

$ terraform destroy       # Terminates resources managed by terraform project

```

##### Kubernetes configuration structure [Kustomize Configuration Management]

```bash
kubernetes/
├── base
│   ├── deployment.yaml
│   ├── external-nlb-service.yaml
│   ├── internal-nlb-service.yaml
│   └── kustomization.yaml
└── overlays
    ├── dev
    │   ├── kustomization.yaml
    │   └── namespace.yaml
    └── prod
        ├── kustomization.yaml
        ├── namespace.yaml
        ├── nlb-cert-arn.yaml
        └── replicas.yaml

```
<br />

```bash
# Steps to setup the project

# This command creates prod-project namespace and deploys /base components with applied overlays for (prod) environment

kubectl apply -k /overlays/dev 
kubectl apply -k /overlays/prod  (Triggers .gitlab-ci.yml [last-step] after the merge from develop to master branch)        
```
<br />

#### CI/CD Flow

![git_flow_diagram](/public/img/git_flow_drawio.png "gitflow_diagram")
<p align="center">
<b>CI/CD Flow</b>
</p>

#### Challenges faced
- Application Build :
    - As application was old. Build was failing initially, Updated Node.js and node-sass library version.
    - Due to old libraries, issue arised with package-lock.json. Removed the file and application build was successful.
- AWS Infrastructure :
    - Transitioning from creating EKS cluster manually to automatic provisioning through Terraform took enormous time due steep learning curve and understanding each AWS component and services.
    - Performed domain registration with Route53 and requested certificate through ACM (AWS Certificate Manager). Created Route53 hosted DNS Zone. Assigned A record for domain **timeoff-management.net**. Updated NS (DNS Name servers) as well after checking using DNS tools Whois lookup, nslookup, dig etc. Currently, **timeoff-management.net** not being resolved. Based on search, it might take 2-3 days to update the DNS.
- CI/CD Pipeline :
    - Last stage of pipeline almost took the whole day to resolve the issue of logging into EKS cluster. Due to lack of clear documentation on locating AWS profile inside the build-agent container. Logging to EKS cluster was not successful. Eventually, aws configure set command helped to set access-key, secret and KUBECONFIG to access the EKS cluster. Before that, applying those values through environment variable was not working. 

<br />

#### Future Improvements
- To solve DNS resolution issue.
- Use self-signed certificate using cert-manager controller in kubernetes cluster.
- Terraform state management through AWS S3 Bucket or Gitlab-managed Terraform state
- Deep dive and study AWS components, services and network extensively.
