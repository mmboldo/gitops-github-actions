## GitHub Actions Secrets

This repo is intentionally **not runnable by default** (portfolio-safe).
To run it yourself, you must create the required AWS resources and then add secrets
to your fork or your own GitHub repository.

### Required secrets (AWS)
These are used by the GitHub Actions workflows to authenticate to AWS and run Terraform.

**Option A — Recommended (OIDC, no long-lived keys)**
If you configure GitHub → AWS OIDC, you only need:

- `AWS_ROLE_TO_ASSUME` — IAM role ARN that GitHub Actions can assume
- `AWS_REGION` — e.g. `us-west-2`

**Option B — Simpler (Access keys)**
If you don’t want to set up OIDC, use IAM access keys:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`

### Terraform remote state
If your Terraform uses an S3 backend + DynamoDB lock table, create them in AWS first
and set (if needed by your workflow):

- `TF_STATE_BUCKET` — S3 bucket name for state
- `TF_STATE_DDB_TABLE` — DynamoDB table name for state locking

### Container registry (optional)
If you build/push the app image:

- `GHCR_TOKEN` (or use `GITHUB_TOKEN` if allowed)
- or ECR-related secrets (if using ECR)

### Kubernetes / EKS access
If your workflow deploys to EKS, it must be able to run `kubectl` against your cluster.
With OIDC, the assumed role must have permissions to describe the cluster and update kubeconfig.

Common needs:
- EKS cluster name and region (either hard-coded in workflow or stored as secrets):
  - `EKS_CLUSTER_NAME`
  - `AWS_REGION`

### Where to set secrets
In GitHub:
`Repo → Settings → Secrets and variables → Actions → New repository secret`

Never commit keys into the repo. Treat secrets as compromised the moment they hit Git history.