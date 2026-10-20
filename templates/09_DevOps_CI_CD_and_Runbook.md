# 🚀 DevOps CI/CD Pipeline & Production Deployment Runbook

> **Document Owner:** Lead DevOps / Site Reliability Engineer (SRE)  
> **System Scope:** Backend Microservices, Mobile App Builds, Cloud Infrastructure  
> **Target Cloud:** AWS (ap-south-1 Mumbai) / Kubernetes (EKS)  
> **Version:** 1.0  

---

## 1. Git Branching & Promotion Strategy

Trunk-based development with protected release branches:

```
[feature/add-live-tracking]  ──(PR + Review + Tests)──>  [develop]
                                                            │
                                                     (Automated CI)
                                                            ▼
                                                       [staging] ──> Deploys to Staging Cluster
                                                            │
                                                  (QA Signoff & Tag)
                                                            ▼
                                                        [main]   ──> Blue-Green Deploy to Production
```

---

## 2. Automated CI/CD Pipeline (GitHub Actions Sample)

Har PR aur main branch push par automated pipeline execute hota hai:

```yaml
# .github/workflows/deploy-production.yml
name: Production Deployment Pipeline

on:
  push:
    branches: [ main ]

jobs:
  test_and_lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - name: Code Quality & Linting
        run: npm run lint
      - name: Run Unit & Integration Tests
        run: npm run test:coverage

  docker_build_and_push:
    needs: test_and_lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      - name: Build & Push Docker Image to ECR
        run: |
          docker build -t quickbite-api:${{ github.sha }} .
          docker tag quickbite-api:${{ github.sha }} ${{ secrets.ECR_REGISTRY }}/quickbite-api:latest
          docker push ${{ secrets.ECR_REGISTRY }}/quickbite-api:latest

  deploy_kubernetes:
    needs: docker_build_and_push
    runs-on: ubuntu-latest
    steps:
      - name: Rolling Update on Kubernetes
        run: |
          kubectl set image deployment/order-service-deployment order-service=${{ secrets.ECR_REGISTRY }}/quickbite-api:${{ github.sha }} -n production
          kubectl rollout status deployment/order-service-deployment -n production --timeout=180s
```

---

## 3. Deployment Runbook (Standard Operating Procedure - SOP)

Production deployment se pehle aur baad ke zaroori steps:

### Pre-Deployment Checklist
- [ ] Database migrations reviewed for backward compatibility (Zero downtime migrations).
- [ ] Environment variables / secrets updated in AWS Secrets Manager.
- [ ] Ops team and Customer Support alerted on internal Slack channel `#deployments`.
- [ ] Peak traffic time na ho (Deployments scheduled for off-peak: 3:00 PM - 5:00 PM).

### Deployment Execution Steps
1. Verify Staging cluster health check: `curl -I https://staging-api.quickbite.com/health`.
2. Trigger the production deployment workflow.
3. Watch Kubernetes pods rollout: `kubectl get pods -n production -w`.
4. Run automated post-deploy smoke tests.

---

## 4. Rollback Strategy (Emergency Fallback)

Agar deployment ke baad error rate spike ho ya critical issue detect ho, to **3 minute ke andar rollback execute karein**:

```bash
# Step 1: Immediate Kubernetes deployment rollback to previous replica revision
kubectl rollout undo deployment/order-service-deployment -n production

# Step 2: Verify pods restored to healthy status
kubectl rollout status deployment/order-service-deployment -n production

# Step 3: Check live error logs in Sentry / Datadog
kubectl logs -f -l app=order-service -n production --tail=100
```

---

## 5. System Health Monitoring & Alerting Matrix

| Metric | Normal Range | Alert Warning Threshold | PagerDuty Critical Alarm | Action |
|--------|--------------|-------------------------|--------------------------|--------|
| **HTTP 5xx Error Rate** | < 0.05% | > 1.0% for 3 mins | > 5.0% for 1 min | Auto-triggers on-call SRE call |
| **API Response Time (p95)**| < 120ms | > 350ms | > 800ms | Scales up Kubernetes replicas |
| **Database CPU Usage** | 20% - 40% | > 70% | > 85% for 5 mins | Alerts Lead DBA for query spike |
