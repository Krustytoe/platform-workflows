# platform-workflows

Reusable GitHub Actions workflows and composite actions for Terraform, security scanning, and cloud authentication. Designed for teams running on AWS (commercial + GovCloud) and Azure (commercial + Government).

## Contents

| Type | Name | What it does |
|---|---|---|
| Reusable workflow | [`terraform-plan`](#terraform-plan) | fmt → init → validate → tflint → plan; posts plan output as a PR comment |
| Reusable workflow | [`terraform-apply`](#terraform-apply) | init → apply; pair with a GitHub environment for an approval gate |
| Reusable workflow | [`trivy-scan`](#trivy-scan) | Trivy scan on filesystem/IaC or container image; uploads SARIF to GitHub Security |
| Composite action | [`aws-oidc-auth`](#aws-oidc-auth) | OIDC auth to AWS — no long-lived keys |
| Composite action | [`azure-oidc-auth`](#azure-oidc-auth) | OIDC auth to Azure — no client secrets |
| Composite action | [`terraform-setup`](#terraform-setup) | Install Terraform + tflint at consistent versions |

---

## terraform-plan

Runs `fmt -check → init → validate → tflint → plan` and posts the plan output as a collapsible PR comment. Supports AWS and Azure; leave `cloud` empty for local/offline validation.

```yaml
jobs:
  plan:
    uses: Krustytoe/platform-workflows/.github/workflows/terraform-plan.yml@main
    permissions:
      contents: read
      id-token: write
      pull-requests: write
    with:
      working_directory: deploy/terraform
      cloud: aws
      aws_region: us-gov-west-1
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

| Input | Type | Default | Description |
|---|---|---|---|
| `working_directory` | string | **required** | Path to the Terraform config, relative to repo root |
| `terraform_version` | string | `~1.10` | Version constraint |
| `cloud` | string | `""` | `aws`, `azure`, or empty |
| `aws_region` | string | `""` | Required when `cloud = aws` |

| Secret | Required | Description |
|---|---|---|
| `AWS_ROLE_ARN` | When AWS | IAM role ARN to assume via OIDC |
| `AZURE_CLIENT_ID` | When Azure | Managed identity client ID |
| `AZURE_TENANT_ID` | When Azure | Azure tenant ID |
| `AZURE_SUBSCRIPTION_ID` | When Azure | Azure subscription ID |

---

## terraform-apply

Runs `init → apply`. Call this after plan is reviewed. Pair with a [GitHub environment](https://docs.github.com/en/actions/deployment/targeting-different-environments) to require explicit approval before apply runs.

```yaml
jobs:
  apply:
    uses: Krustytoe/platform-workflows/.github/workflows/terraform-apply.yml@main
    environment: prod          # approval gate lives here
    permissions:
      contents: read
      id-token: write
    with:
      working_directory: deploy/terraform
      cloud: aws
      aws_region: us-gov-west-1
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

Inputs and secrets are identical to `terraform-plan`.

---

## trivy-scan

Runs Trivy against a filesystem/IaC path or container image and uploads the results to GitHub Security as SARIF.

```yaml
jobs:
  scan:
    uses: Krustytoe/platform-workflows/.github/workflows/trivy-scan.yml@main
    permissions:
      contents: read
      security-events: write
    with:
      scan_type: fs
      target: .
      severity: HIGH,CRITICAL
```

| Input | Type | Default | Description |
|---|---|---|---|
| `scan_type` | string | `fs` | `fs` (filesystem/IaC) or `image` |
| `target` | string | `.` | Path for `fs`; image reference for `image` |
| `severity` | string | `HIGH,CRITICAL` | Severities to report |
| `upload_sarif` | boolean | `true` | Upload SARIF to GitHub Security tab |

---

## aws-oidc-auth

```yaml
- uses: Krustytoe/platform-workflows/actions/aws-oidc-auth@main
  with:
    role_arn: ${{ secrets.AWS_ROLE_ARN }}
    region: us-gov-west-1
```

Wraps `aws-actions/configure-aws-credentials`. Requires `id-token: write` permission on the calling job.

---

## azure-oidc-auth

```yaml
- uses: Krustytoe/platform-workflows/actions/azure-oidc-auth@main
  with:
    client_id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant_id: ${{ secrets.AZURE_TENANT_ID }}
    subscription_id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

Wraps `azure/login`. Requires `id-token: write` permission on the calling job.

---

## terraform-setup

```yaml
- uses: Krustytoe/platform-workflows/actions/terraform-setup@main
  with:
    terraform_version: "~1.10"
```

Installs Terraform and tflint, then runs `tflint --init` to download configured plugins.

---

## Tying it together

A full plan-then-apply flow for a Terraform repo:

```yaml
on:
  pull_request:
  push:
    branches: [main]

jobs:
  plan:
    uses: Krustytoe/platform-workflows/.github/workflows/terraform-plan.yml@main
    permissions:
      contents: read
      id-token: write
      pull-requests: write
    with:
      working_directory: deploy/terraform
      cloud: aws
      aws_region: us-gov-west-1
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}

  apply:
    needs: plan
    if: github.ref == 'refs/heads/main'
    uses: Krustytoe/platform-workflows/.github/workflows/terraform-apply.yml@main
    environment: prod
    permissions:
      contents: read
      id-token: write
    with:
      working_directory: deploy/terraform
      cloud: aws
      aws_region: us-gov-west-1
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}
```

## License

MIT
