# OIDC Authentication Reference

Federated identity — no long-lived credentials stored in GitHub secrets.

---

## Google Cloud Platform

### 1. Create Workload Identity Pool (one-time)

```bash
gcloud iam workload-identity-pools create "github-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="GitHub Actions Pool"

gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --display-name="GitHub provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```

### 2. Bind Service Account

```bash
gcloud iam service-accounts add-iam-policy-binding "sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/github-pool/attribute.repository/ORG/REPO"
```

### 3. Workflow

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: google-github-actions/auth@v2
    with:
      workload_identity_provider: projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github-pool/providers/github-provider
      service_account: sa@PROJECT_ID.iam.gserviceaccount.com

  - uses: google-github-actions/setup-gcloud@v2
```

---

## AWS

### 1. Create OIDC Provider (one-time, via Terraform or console)

```hcl
resource "aws_iam_openid_connect_provider" "github" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

resource "aws_iam_role" "github_actions" {
  name = "github-actions-role"
  assume_role_policy = jsonencode({
    Statement = [{
      Action    = "sts:AssumeRoleWithWebIdentity"
      Effect    = "Allow"
      Principal = { Federated = aws_iam_openid_connect_provider.github.arn }
      Condition = {
        StringLike = {
          "token.actions.githubusercontent.com:sub" = "repo:ORG/REPO:*"
        }
      }
    }]
  })
}
```

### 2. Workflow

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::ACCOUNT_ID:role/github-actions-role
      aws-region: eu-west-1
```

---

## Azure

```yaml
permissions:
  id-token: write
  contents: read

steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

The client-id here is a federated credential — not a secret. Store in secrets for convenience, but it carries no privilege itself.

---

## Subject Claim Patterns

Control which GitHub contexts can assume the identity:

| Claim | Restricts to |
|---|---|
| `repo:ORG/REPO:ref:refs/heads/main` | Only `main` branch |
| `repo:ORG/REPO:environment:production` | Only `production` environment |
| `repo:ORG/REPO:pull_request` | Any PR (risky — fork PRs too) |
| `repo:ORG/REPO:*` | Any ref in this repo |

Always be as specific as possible — prefer environment-scoped claims for production deploys.
