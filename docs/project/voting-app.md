# Voting App (Docker → ECR → EC2/EKS)

This page documents the **Example Voting App** I implemented end-to-end using Docker, AWS ECR, EC2 (via SSM, no SSH), and (optional) EKS.

## Overview
- **vote** (Python/Flask) → users vote A/B
- **worker** (background) → reads from Redis, writes to Postgres
- **result** (Node.js) → shows live results
- **Redis** + **Postgres** as data services
- **CI/CD** with GitHub Actions (build, push to ECR, deploy/run)

## Architecture
```mermaid
flowchart LR
  subgraph Client
    U[User Browser]
  end
  subgraph App
    V[vote (Flask)]
    R[result (Node)]
    W[worker]
    D[(Postgres)]
    C[(Redis)]
  end
  U --> V
  V --> C
  W --> C
  W --> D
  R --> D

Run Locally (Docker Compose)
# docker-compose.yml contains: vote, result, worker, redis, db
docker compose up --build
# Open http://localhost:5000 for vote, http://localhost:5001 for result


Common logs you’ll see:

worker processing messages

db confirming inserts

vote serving /

result serving /

Push Images to AWS ECR
AWS_ACCOUNT_ID=<your-account-id>
AWS_REGION=eu-west-2
REPO_PREFIX=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

aws ecr get-login-password --region $AWS_REGION \
 | docker login --username AWS --password-stdin $REPO_PREFIX

# Example: build & push 'vote'
docker build -t vote:latest ./vote
docker tag vote:latest $REPO_PREFIX/vote:latest
docker push $REPO_PREFIX/vote:latest


Repeat for result, worker.

Run on EC2 via SSM (no SSH)
INSTANCE_ID=i-xxxxxxxxxxxxxxxxx
AWS_REGION=eu-west-2
REPO_PREFIX=<acct>.dkr.ecr.<region>.amazonaws.com

aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --parameters commands='[
    "docker login -u AWS -p $(aws ecr get-login-password --region '"$AWS_REGION"') '"$REPO_PREFIX"'",
    "docker pull '"$REPO_PREFIX"'/vote:latest",
    "docker pull '"$REPO_PREFIX"'/result:latest",
    "docker pull '"$REPO_PREFIX"'/worker:latest",
    "docker network create vote || true",
    "docker run -d --name redis --network vote redis:7",
    "docker run -d --name db --network vote -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=votes -p 5432:5432 postgres:16",
    "docker run -d --name vote --network vote -p 5000:80 '"$REPO_PREFIX"'/vote:latest",
    "docker run -d --name result --network vote -p 5001:80 '"$REPO_PREFIX"'/result:latest",
    "docker run -d --name worker --network vote '"$REPO_PREFIX"'/worker:latest"
  ]' \
  --targets "Key=instanceids,Values=$INSTANCE_ID" \
  --region $AWS_REGION


(Optional) Deploy to EKS

Build/push images to ECR (as above).

Apply Kubernetes manifests or Helm chart for vote, result, worker, redis, postgres.

Verify rollout: kubectl rollout status deploy/<name> -n <ns>

CI/CD (GitHub Actions)

Build & push docker images to ECR on push

Deploy to EC2 via SSM or to EKS via kubectl

Example step (ECR login):

- uses: aws-actions/amazon-ecr-login@v2


OIDC to AWS (no long-lived keys):

permissions:
  id-token: write
  contents: read
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::<acct>:role/GitHubOIDCDeployRole
    aws-region: eu-west-2

Proof (Screenshots)

ECR repos & pushed tags


EC2 SSM console showing success


Local app pages (vote & result)