markdown
# Runbook — Kubernetes On-Call

## Symptoms
- Pod CrashLoopBackOff / 502 from ingress / rollout stuck

## Quick Checks
```bash
kubectl get pods -A
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous
kubectl get events -n <ns> --sort-by=.lastTimestamp | tail -20
Common Fixes
Image pull error → check ECR creds/tag

Readiness probe failing → verify /healthz & container port

OOMKilled → increase memory limits

Stuck rollout:

bash
Copy
Edit
kubectl rollout status deploy/<name> -n <ns>
kubectl rollout undo deploy/<name> -n <ns>