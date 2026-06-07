# CI Pipelines — Reusable GitHub Actions Workflows

Central repository for shared CI/CD workflows used by the voting-app microservices.

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `reusable-build.yml` | `workflow_call` | Build multi-arch Docker → Push ECR → Update GitOps |

## How Service Repos Use This

```yaml
jobs:
  build-and-deploy:
    uses: mochthebest-byte/ci-pipelines/.github/workflows/reusable-build.yml@main
    with:
      image_name: vote
      ecr_repository: my-app/vote
      service_name: vote
      aws_region: us-east-1
      ci_role_arn: arn:aws:iam::428156589409:role/my-app-eks-github-ci
    secrets:
      gitops_token: ${{ secrets.GITOPS_PAT }}
```

## Pipeline Steps

1. **Checkout** — clones the service repository
2. **OIDC Auth** — authenticates to AWS via GitHub OIDC (no static keys)
3. **ECR Login** — authenticates to Amazon ECR
4. **Buildx Setup** — prepares Docker Buildx for multi-arch builds
5. **Build & Push** — builds images for `linux/amd64` and `linux/arm64`, pushes to ECR
6. **GitOps Update** — commits the new image tag to the GitOps repo (ArgoCD auto-syncs)

## Features

- **No static AWS keys** — uses OIDC Workload Identity Federation
- **Multi-arch** — builds for both amd64 and arm64 in a single shot
- **BuildKit caching** — inline cache for faster rebuilds
- **Auto-cleanup** — ECR lifecycle policies (keep 10, expire >90d)
- **Graceful errors** — readable messages for missing manifests or config issues

## Required Secrets

| Secret | Purpose |
|--------|---------|
| `GITOPS_PAT` | GitHub token with push access to the GitOps repo |
