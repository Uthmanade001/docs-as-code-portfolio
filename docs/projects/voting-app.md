# Voting App (Docker → ECR → EC2/EKS)

This project demonstrates how I deployed the **Example Voting App** end-to-end with Docker, AWS ECR, and EC2/EKS (via SSM).

It showcases my skills in **containerisation, CI/CD, AWS, and Kubernetes**.

---

## ⚙️ Architecture
The app has 5 services:
- **vote** → Flask app for casting A/B votes  
- **result** → Node.js app showing live results  
- **worker** → background processor  
- **Redis** → in-memory queue  
- **Postgres** → database  

```mermaid
flowchart LR
  V[vote (Flask)] --> R[(Redis)]
  W[worker] --> R
  W --> D[(Postgres)]
  Rz[result (Node.js)] --> D
![Voting App UI](../assets/img/vote-arch.png.png)

▶️ Run Locally with Docker Compose
docker compose up -d
Vote UI → http://localhost:5000

Result UI → http://localhost:5001

Check logs:

docker compose logs -f worker

☁️ Push Images to AWS ECR
1. Quick method — retag public images

Pull the official Voting App images → tag them to your ECR → push.

Bash (Git Bash / WSL):

`AWS_ACCOUNT_ID=<YOUR_AWS_ACCOUNT_ID>`
`AWS_REGION=<YOUR_REGION>`
`URI="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"`

`aws ecr get-login-password --region "$AWS_REGION" \`
 ` | docker login --username AWS --password-stdin "$URI"`

# Create repos if they don’t exist
for repo in vote result worker; do
  aws ecr describe-repositories --repository-names "$repo" \
  || aws ecr create-repository --repository-name "$repo"
done

# Pull → tag → push
for img in vote result worker; do
  docker pull "dockersamples/examplevotingapp_${img}:latest"
  docker tag  "dockersamples/examplevotingapp_${img}:latest" "$URI/${img}:latest"
  docker push "$URI/${img}:latest"
done

✅ After this, your images are available in ECR as:

$URI/vote:latest

$URI/result:latest

$URI/worker:latest

![ECR Push](../assets/img/Screenshot%202025-08-11%20120617.png)

2. Build your own images (if you have the app code locally)

docker build -t vote:latest ./vote
docker tag vote:latest $URI/vote:latest
docker push $URI/vote:latest

docker build -t result:latest ./result
docker tag result:latest $URI/result:latest
docker push $URI/result:latest

docker build -t worker:latest ./worker
docker tag worker:latest $URI/worker:latest
docker push $URI/worker:latest

💻 Deploy on EC2 via SSM (no SSH)
---
INSTANCE_ID=i-xxxxxxxx
AWS_REGION=<YOUR_REGION>
URI=<your-ecr-uri>
---
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --targets "Key=instanceIds,Values=$INSTANCE_ID" \
  --parameters 'commands=[
    "docker run -d --name vote   -p 5000:80  '"$URI"'/vote:latest",
    "docker run -d --name result -p 5001:80  '"$URI"'/result:latest",
    "docker run -d --name worker             '"$URI"'/worker:latest"
  ]' \
  --region $AWS_REGION
---

☸️ Deploy on EKS (optional)
Apply Kubernetes manifests or Helm charts
Verify rollout:
kubectl rollout status deploy/vote -n prod

🔄 CI/CD with GitHub Actions

Build & Push images to ECR

Deploy to EC2 via SSM or EKS via kubectl

Secure with OIDC (no long-lived AWS keys)

Example (ECR login step):
- uses: aws-actions/amazon-ecr-login@v2
![GitHub Actions Deploy](../assets/img/Screenshot%202025-08-15%20201605%20-%20Copy.png)

✅ Proof
ECR repos with images
EC2/SSM logs showing containers running
Vote + Result UI working in browser
![Vote UI](../assets/img/Screenshot%202025-08-14%20022857.png)
![Result UI](../assets/img/Screenshot%202025-08-14%20022939.png)

🔑 Key Learnings
Containerise and orchestrate microservices
Secure CI/CD with GitHub OIDC → AWS
Automated deployments without SSH
Importance of runbooks for debugging (CrashLoopBackOff, push errors, etc.)


---
