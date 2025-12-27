# Verification Guide

This document describes how to verify that the platform and application were deployed successfully.

---

## 1. GitHub Actions

- All workflows complete successfully
- Terraform plan/apply succeeded
- Application build and deploy completed

---

## 2. Terraform State

- Remote state exists in S3
- Locking works as expected (if enabled)

---

## 3. Kubernetes (EKS)

- Nodes are Ready
- Pods are Running
- Services are reachable

Example:
```bash
kubectl get nodes
kubectl get pods -A
```