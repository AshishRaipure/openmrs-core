# OpenMRS Core - Production Readiness & Test Runbook

This document captures how to validate OpenMRS core for production and how to deploy it on Kubernetes.

## 1) Build and Test

Run from repository root:

```bash
mvn clean verify -DskipITs
```

If the full build cannot complete in your environment, check:

- Java version compatibility (OpenMRS 2.5.x typically targets Java 8/11 runtime compatibility).
- Maven repository access from your CI/build network.
- Credentials or proxy settings for `https://mavenrepo.openmrs.org`.

Recommended CI pipeline stages:

1. `mvn -B -DskipTests dependency:go-offline`
2. `mvn -B clean verify`
3. Container image build + vulnerability scan.
4. Deploy to staging and run smoke tests (`/openmrs/login.htm`).

## 2) Production Readiness Checklist

- [ ] Build is reproducible in CI with pinned Maven and JDK versions.
- [ ] Unit/integration tests are green.
- [ ] DB migrations are validated in a staging database.
- [ ] Secrets are externally managed (not stored in git).
- [ ] TLS termination enabled at ingress/load balancer.
- [ ] Monitoring and alerting configured.
- [ ] Backups and disaster recovery tested.
- [ ] Rollback procedure documented and tested.

## 3) Kubernetes Deployment

Use manifests in [`deploy/kubernetes`](deploy/kubernetes/README.md).

High-level commands:

```bash
kubectl apply -f deploy/kubernetes/namespace.yaml
kubectl apply -f deploy/kubernetes/configmap.yaml
kubectl apply -f deploy/kubernetes/secret.example.yaml
kubectl apply -f deploy/kubernetes/deployment.yaml
kubectl apply -f deploy/kubernetes/service.yaml
kubectl apply -f deploy/kubernetes/ingress.yaml
```

## 4) Post-Deploy Smoke Test

```bash
kubectl -n openmrs rollout status deploy/openmrs
kubectl -n openmrs get pods
kubectl -n openmrs logs deploy/openmrs --tail=200
```

Then open the ingress host and verify the login page loads.
