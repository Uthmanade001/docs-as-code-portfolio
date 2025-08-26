markdown
# Runbook — CI/CD Troubleshooting

## Push rejected (“fetch first”)
```bash
git push --force-with-lease
# or
git pull origin main --allow-unrelated-histories && git push