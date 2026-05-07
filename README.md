# AWS EKS Cluster Architecture

![Alt text](https://github.com/nawaz-ch/ingress-https-for-retailstore-application/blob/23ac132e8be1f1dc88f477db7af8fabf7780ea7d/aws-vpc-architecture.png)


# Project Structure

| File | Description |
|-----------|-------|
| c1_versions.tf | Required Terraform + AWS provider versions |
| c2_variables.tf | Remote backend for Terraform state (S3 + DynamoDB) |
|c4_datasources_and_locals.tf | 	AWS data sources and local values |
| c5_eks_tags.tf | 	Common tags for resources |
| c6_eks_cluster_iamrole.tf | IAM role for EKS control plane |
| c7_eks_cluster.tf | EKS cluster resource definition |
| c8_eks_nodegroup_iamrole.tf | 	IAM role for EKS worker node groups |
| c9_eks_nodegroup_private.tf | 	Private node group configuration |
| c10_eks_outputs.tf | 	Useful Terraform outputs (kubeconfig, cluster details) |
| data-lbc.tf | get IAM policy for the loadbalancer Controller via http	 |
| iam-policy-for-lbc.tf | creates iam role with iam policy and attached to lbc service account  |


# Steps to provision

```bash
# Terraform initalize
terraform init

# Terraform validate
terraform validate

# Terraform Plan
terraform plan

# Terraform Apply
terraform apply -auto-approve

```

# Configure kubectl CLI to access EKS Cluster

```
# EKS KubeConfig
aws eks update-kubeconfig --name <cluster_name> --region <aws_region>

# get all nodes
kubectl get nodes

# get all pods
kubectl get pods

```

# AWS LoadBalancer Controller on EKS(with pod identity)




