# Known Issues & Troubleshooting

This document lists known issues encountered during development of this
GitOps platform blueprint, along with their root causes and resolutions.

The goal is to document **real-world failure scenarios**, not hide them.

---

## 1. Terraform State Lock Conflicts

### Description

Terraform workflows may fail with state lock errors when multiple pipeline
runs attempt to modify infrastructure simultaneously.

### Root Cause

- Concurrent `terraform apply` executions
- Missing or misconfigured state locking

### Resolution

- Use a DynamoDB table for Terraform state locking
- Restrict `apply` to protected branches (e.g. `main`)
- Allow `plan` only on pull requests

### Lesson Learned

Infrastructure pipelines must enforce **serialization** for state changes.

---

## 2. GitHub Actions Missing Secrets

### Description

Workflows fail during authentication or deployment steps.

### Root Cause

- Required GitHub Actions secrets not configured
- Incorrect secret names
- Secrets scoped incorrectly (repo vs org)

### Resolution

- Verify all required secrets are configured
- Match secret names exactly as referenced in workflows
- Prefer repository-scoped secrets for isolation

### Lesson Learned

CI/CD pipelines should **fail fast** and clearly when secrets are missing.

---

## 3. EKS Authentication Errors

### Description

Kubernetes deployment steps fail with authentication or authorization errors.

### Root Cause

- IAM role not mapped in the EKS `aws-auth` ConfigMap
- Insufficient IAM permissions for GitHub Actions user

### Resolution

- Update `aws-auth` ConfigMap to include CI/CD role
- Verify IAM policies include required EKS permissions

### Lesson Learned

Cloud IAM and Kubernetes RBAC must be aligned explicitly.

---

## 4. Container Image Pull Failures

### Description

Pods fail to start due to image pull errors.

### Root Cause

- ECR repository does not exist
- Incorrect image tag
- Missing ECR permissions

### Resolution

- Create ECR repository manually or via Terraform
- Ensure GitHub Actions pushes images before deployment
- Verify EKS node role has ECR pull permissions

### Lesson Learned

Image build, publish, and deploy steps must be tightly ordered.

---

## 5. Cost-Related Issues

### Description

Unexpected AWS charges incurred during testing.

### Root Cause

- EKS clusters left running
- Load balancers not destroyed
- ECR images accumulating

### Resolution

- Destroy Terraform-managed resources after testing
- Periodically clean unused ECR images
- Monitor AWS billing dashboard

### Lesson Learned

Cost management is a **first-class concern** in infrastructure projects.

---

## Summary

These issues reflect common challenges in real-world GitOps pipelines.

Documenting them demonstrates:
- Operational awareness
- Troubleshooting ability
- Production-minded thinking
