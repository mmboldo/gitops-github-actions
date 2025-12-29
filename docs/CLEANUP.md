# CLEANUP.md — AWS Cost Avoidance / Teardown Guide

This document explains how to safely remove all AWS and Kubernetes
resources created by this project to avoid ongoing costs.

---

## What can generate AWS costs?

Resources created by this project may include:

- EKS cluster (control plane)
- EC2 worker nodes / node groups
- AWS Load Balancers (created by Ingress or Service type LoadBalancer)
- NAT Gateway
- EBS volumes
- ECR repositories
- S3 bucket used for Terraform remote state
- CloudWatch logs

---

## 0. Pre-flight: confirm AWS account and region

Before destroying anything, verify which AWS account and region
you are connected to:

    aws sts get-caller-identity
    aws configure get region

If your GitHub Actions workflow used a different region,
switch to that region first.

---

## 1. Remove Kubernetes workloads first

Ingress and Service type=LoadBalancer often create AWS load balancers.
These should be removed before running Terraform destroy.

---

### 1.1 Update kubeconfig (if needed)

    aws eks --region <AWS_REGION> update-kubeconfig --name <EKS_CLUSTER_NAME>
    kubectl get nodes

You should see worker nodes listed.

---

### 1.2 Delete application manifests

If you deployed using Kustomize:

    kubectl delete -k app/kubernetes/overlays/dev
    kubectl delete -k app/kubernetes/overlays/prod

If you applied raw manifests:

    kubectl delete -f <path-to-manifests>

---

### 1.3 Delete NGINX Ingress Controller

The workflow installs ingress-nginx using the AWS provider manifest.

Remove it with:

    kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.1/deploy/static/provider/aws/deploy.yaml

---

### 1.4 Verify Load Balancers are gone

    kubectl get svc -A | grep LoadBalancer || true
    kubectl get ingress -A

Check AWS side as well:

    aws elbv2 describe-load-balancers --region <AWS_REGION>

---

## 2. Destroy AWS infrastructure with Terraform

Run Terraform from the same folder used by the workflow
(e.g. infra/terraform).

---

### 2.1 Initialize Terraform backend

    terraform init \
      -backend-config="bucket=<TF_STATE_BUCKET>" \
      -backend-config="region=<AWS_REGION>" \
      -backend-config="key=terraform.tfstate"

---

### 2.2 Destroy infrastructure

    terraform destroy -auto-approve

If destroy fails due to dependencies:
- Re-check Kubernetes resources are deleted
- Wait a few minutes
- Re-run terraform destroy

---

## 3. Remove Terraform remote state bucket (S3)

Terraform may not delete the S3 bucket automatically.

---

### 3.1 Empty the bucket

    aws s3 rm s3://<TF_STATE_BUCKET> --recursive

If versioning is enabled, delete all object versions
(using AWS Console is easiest).

---

### 3.2 Delete the bucket (optional)

    aws s3api delete-bucket --bucket <TF_STATE_BUCKET> --region <AWS_REGION>

---

## 4. Remove ECR repositories (if used)

List repositories:

    aws ecr describe-repositories --region <AWS_REGION>

Delete a repository:

    aws ecr delete-repository \
      --repository-name <REPO_NAME> \
      --force \
      --region <AWS_REGION>

---

## 5. Clean up GitHub Actions IAM credentials

If you created an IAM user for GitHub Actions:

- Delete access keys
- Remove the IAM user if no longer needed
- Remove secrets from GitHub repository settings

List access keys:

    aws iam list-access-keys --user-name <IAM_USER>

Delete an access key:

    aws iam delete-access-key \
      --user-name <IAM_USER> \
      --access-key-id <KEY_ID>

---

## 6. Final verification checklist

Ensure nothing billable remains.

### EKS clusters

    aws eks list-clusters --region <AWS_REGION>

### EC2 instances

    aws ec2 describe-instances --region <AWS_REGION>

### Load balancers

    aws elbv2 describe-load-balancers --region <AWS_REGION>

### NAT Gateways

    aws ec2 describe-nat-gateways --region <AWS_REGION>

### EBS volumes

    aws ec2 describe-volumes --region <AWS_REGION>

---

## Notes

- Always remove Kubernetes resources before Terraform destroy.
- Prefer Terraform destroy over manual deletions.
- For portfolio purposes, documenting teardown is sufficient —
  resources do not need to be created.