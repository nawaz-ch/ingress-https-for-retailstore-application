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
![Alt text](https://github.com/nawaz-ch/ingress-https-for-retailstore-application/blob/70a29f77e2e33ff6357a02544ac9dd05fcf8d763/loadbalancer.png)


# Install AWS LoadBalancer Controller using Helm

**Export Environment Variables**
```bash
# Replace the placeholders below with your actual values
export AWS_REGION="us-east-1"
export EKS_CLUSTER_NAME="retail-dev-eksdemo1"
export AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Confirm values
echo $AWS_REGION
echo $EKS_CLUSTER_NAME
echo $AWS_ACCOUNT_ID
```
**Add Helm repo and update**
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

**Install LoadBalancer Controller**
```bash
# Get VPC ID
VPC_ID=$(aws eks describe-cluster \
  --name ${EKS_CLUSTER_NAME} \
  --query "cluster.resourcesVpcConfig.vpcId" \
  --output text)

# Verify VPC ID
echo $VPC_ID

# Install AWS Load Balancer Controller using HELM
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=${EKS_CLUSTER_NAME} \
  --set region=${AWS_REGION} \
  --set vpcId=${VPC_ID} \
  --set serviceAccount.create=true \
  --set serviceAccount.name=aws-load-balancer-controller  
```

> [!NOTE]
> IAM ROLE AND IAM POLICY and eks pod identity association with ServiceAccount for loadbalancer controller have been done through Terraform.


**Verify Helm Release**
```bash
# list all helm releases
helm list -n kube-system
```

✅ **Example Output**
```bash
NAME                        NAMESPACE    REVISION  UPDATED        STATUS    CHART                           APP VERSION
aws-load-balancer-controller kube-system 1         2025-10-08...  deployed  aws-load-balancer-controller-1.13.0  v2.13.3
```

**Check helm status**
```bash
helm status aws-load-balancer-controller -n kube-system
```

✅ **confirm `STATUS: deployed`**

**verfiy controller deployment**
```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

✅ **Expected output**
```bash
NAME                                           READY   STATUS    RESTARTS   AGE
aws-load-balancer-controller-b7d4fdf8c-9gxjz   1/1     Running   0          1m
```

**check deployment and logs**
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

# Kubernetes Ingress - HTTPS (with ACM + Route53)

## 📋 Prerequisites

Before you begin, make sure you have the following:

To follow along with the HTTPS demo, you should already have a registered domain name in AWS Route53.
This is important because ACM (AWS Certificate Manager) requires a domain for SSL certificate validation.

---


# Creation of ACM Cert and route53 DNS

1. Create ACM cert + Route53 DNS.
2. Deploy HTTPS Ingress, test end-to-end.
3. Request Public Certificate in AWS Certificate Manager (ACM):
for eg:
```bash
retailstore.nawazshareef-kubecloud.com
```
4. Validate via DNS: ACM will show CNAME records to create in Route53 → Hosted Zone: nawazshareef-kubecloud.com.
5. Wait for certificate Status: Issued.
6. Keep the certificate in the same AWS region as your EKS/ALB.

**Update Ingress Manifests with SSL Certificate ARN in 01_ingress_https_instance_mode.yaml**
[View 01_ingress_https_instance_mode](https_retail_store_k8s_manifests/06_ingress/01_ingress_https_instance_mode.yaml)
```bash
## SSL Settings
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:339713166619:certificate/e2d65ddb-389f-48c1-a4e8-c215d56c553f
```

# Deploy Kubernetes Ingress HTTPS
```bash
# Apply all HTTPS manifests (includes Ingress with TLS annotations)
kubectl apply -R -f https_retail_store_k8s_manifests/
```

# Create DNS record (Route53)
After ALB is provisioned
```bash
kubectl get ingress 
```

**In Route53 → nawazshareef-kubecloud.com, create CNAME:**
```bash
Name: retailstore.stacksimplify.com
Type: CNAME
Value: <ALB-DNS-NAME>
TTL: 60
```

# Verify Kubernetes Ingress - HTTPS

```bash
# Ingress overall
kubectl get ingress -A

# Inspect annotations, rules, and TLS
kubectl describe ingress retail-store-https-instance-mode
kubectl describe ingress retail-store-https-ip-mode

# Test HTTPS with SNI
curl -vk https://retailstore.nawazshareef-kubecloud.com
```

✅ **Expect a valid TLS handshake (issued by ACM) and Retail Store UI over HTTPS.**




