This is your main project case study.

# Voting App (Docker → ECR → EC2/EKS)

This project demonstrates how I deployed the **Example Voting App** end-to-end with Docker, AWS ECR, and EC2/EKS (via SSM).

## Architecture
- **vote** (Flask app) — users cast A/B votes
- **result** (Node app) — shows live results
- **worker** (background processor)
- **Redis** (queue) + **Postgres** (database)

```mermaid
flowchart LR
  V[vote] --> R[(Redis)]
  W[worker] --> R
  W --> D[(Postgres)]
  Rz[result] --> D

Run Locally
docker compose up -d
# Vote UI:   http://localhost:5000
# Result UI: http://localhost:5001

Push Images to AWS ECR
AWS_ACCOUNT_ID=<your-id>
AWS_REGION=eu-west-2
URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

aws ecr get-login-password --region $AWS_REGION \
 | docker login --username AWS --password-stdin $URI

docker build -t vote ./vote
docker tag vote:latest $URI/vote:latest
docker push $URI/vote:latest

Run on EC2 (via SSM)
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets "Key=instanceIds,Values=i-xxxxxxxx" \
  --parameters 'commands=["docker run -d -p 5000:80 <ECR_URI>/vote:latest"]'

(Optional) Run on EKS

Apply Kubernetes manifests (Deployments + Services)

Rollout: kubectl rollout status deployment/vote

![Voting App UI](../assets/img/vote-arch.png.png)