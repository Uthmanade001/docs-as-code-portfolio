# GitHub Actions OIDC → AWS

## Why
Instead of long-lived AWS keys in GitHub, I used **OIDC** so workflows assume a role securely.

## Steps
1. Create IAM role with OIDC trust:
   - Provider: `token.actions.githubusercontent.com`
   - Condition: `repo: Uthmanade001/docs-as-code-portfolio:ref:refs/heads/main`

2. Attach permissions for ECR/EKS.

3. In GitHub Actions:
```yaml
permissions:
  id-token: write
  contents: read

- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::<acct>:role/GitHubOIDCDeployRole
    aws-region: eu-west-2