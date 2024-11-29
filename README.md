# Building a Secure DevOps Workflow with GitOps and Kubernetes

## Project Overview

This project aims to create a secure DevOps pipeline by leveraging GitOps principles to automate deployments to Kubernetes clusters. The workflow will include security checks, compliance audits, and secrets management to ensure that security best practices are followed throughout the application deployment lifecycle.

## Project Goals

- **Automate Deployments**: Implement a GitOps workflow to automate application deployment to Kubernetes clusters using tools like ArgoCD.
- **Set Up EKS**: Create a Kubernetes cluster on EKS for deploying containerized applications.
- **Provision Infrastructure**: Use Terraform to automate AWS infrastructure provisioning, including an EKS cluster.
- **Version Control**: Ensure that all infrastructure and application changes are tracked and managed using version control, specifically Git.
- **Secrets Management**: Use Jenkins credentials to manage sensitive information like API keys and passwords securely.

## Tools Used

- **GitOps**:
  - ArgoCD : GitOps tools for continuous delivery and Kubernetes management.
- **Container Orchestration**:
  - Kubernetes: A container orchestration platform for automating deployment, scaling, and management of containerized applications.
- **Version Control**:
  - GitHub : Version control systems used to store and manage application and infrastructure code.
- **CI/CD**:
  - Jenkins : Continuous integration and continuous delivery tools to automate testing and deployment processes.

## Prerequisites

- A Kubernetes cluster (local, cloud, or managed service like GKE, EKS, or AKS).
- Access to GitHub or GitLab for version control and repository management.
- A running instance of ArgoCD for GitOps.
- Jenkins or GitLab CI setup for the CI/CD pipeline.

## Getting Started

### 1. Set up Kubernetes Cluster

Ensure you have access to a Kubernetes cluster. You can use a cloud provider like AWS, Azure, or Google Cloud, or set up a local Kubernetes environment using Minikube or kind (Kubernetes in Docker).

```bash
# Create an EKS cluster using AWS CLI
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::123456789012:role/eks-cluster-role \
  --resources-vpc-config subnetIds=subnet-678f1514,subnet-123f43fa,securityGroupIds=sg-6979fe18   
```

### 2. Install ArgoCD or Flux for GitOps
You can choose between ArgoCD or Flux for GitOps. Here’s how to install ArgoCD on your Kubernetes cluster.

```bash
# Create a namespace for ArgoCD
kubectl create namespace argocd

# Install ArgoCD in the namespace
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Expose the ArgoCD API server for access (local port forwarding)
kubectl port-forward svc/argocd-server -n argocd 8080:443
   
```
Access ArgoCD UI : 
Once ArgoCD is installed, you can access the ArgoCD dashboard at http://localhost:8080. The default username is admin, and you can retrieve the password with the following command:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
   
```

### 3. Set up Version Control (GitHub)
Create a Git repository on GitHub to manage your Kubernetes deployment manifests and application CI/CD configurations.


### 4. Configure CI/CD with Jenkins or GitLab CI
Set up a Jenkins pipeline or GitLab CI pipeline to automate the testing, building, and deployment of applications to Kubernetes.
(Refer Jenkinsfile.)


### 5. GitOps Deployment
Configure ArgoCD (or Flux) to automatically deploy changes from your Git repository to your Kubernetes cluster.

Add your repository to ArgoCD:

```bash
# Add your Git repository to ArgoCD
argocd repo add https://github.com/your-repo/application.git --username <username> --password <password>

# Create an ArgoCD application that tracks changes from your Git repository:
argocd app create my-app \
  --repo https://github.com/your-repo/application.git \
  --path k8s/ \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
   
```

### 6. Provision Infrastructure with Terraform (AWS)
Use Terraform to provision the infrastructure required for your Kubernetes cluster on AWS (e.g., EKS, VPC, subnets, etc.).
(Refer configuration files of K8s folder.)

Apply Terraform Configuration:

```bash
# Initialize Terraform
terraform init

# Review the plan
terraform plan

# Apply the Terraform configuration to create AWS resources
terraform apply
   
```
Once applied, Terraform will create the necessary AWS resources (e.g., VPC, subnets, IAM roles) and provision the EKS cluster.


## Conclusion : 
By following the above steps, you can automate the deployment process using GitOps with ArgoCD or Flux, manage infrastructure with Terraform on AWS, and ensure security through Snyk, Aqua Security, and HashiCorp Vault. This setup provides a scalable, automated, and secure way to deploy applications to Kubernetes.





