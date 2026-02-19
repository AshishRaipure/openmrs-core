# OpenMRS Kubernetes Deployment Guide

This folder contains production-oriented Kubernetes manifests for deploying OpenMRS.

## What's Included

- `namespace.yaml` - dedicated namespace.
- `configmap.yaml` - non-secret runtime settings.
- `secret.example.yaml` - template for DB credentials.
- `deployment.yaml` - OpenMRS app Deployment with probes and resource limits.
- `service.yaml` - ClusterIP service for internal traffic.
- `ingress.yaml` - optional Ingress for external HTTP routing.

## Prerequisites

1. A Kubernetes cluster (v1.24+ recommended).
2. `kubectl` configured for your cluster.
3. A reachable MariaDB/MySQL instance.
4. A container image that contains OpenMRS and starts on port `8080`.

> The sample deployment references `openmrs/openmrs-core:2.5.0`. Replace this image with your validated release image if needed.

## Deploy

```bash
kubectl apply -f deploy/kubernetes/namespace.yaml
kubectl apply -f deploy/kubernetes/configmap.yaml
kubectl apply -f deploy/kubernetes/secret.example.yaml
kubectl apply -f deploy/kubernetes/deployment.yaml
kubectl apply -f deploy/kubernetes/service.yaml
kubectl apply -f deploy/kubernetes/ingress.yaml
```

## Validate

```bash
kubectl -n openmrs get pods
kubectl -n openmrs get svc
kubectl -n openmrs get ingress
kubectl -n openmrs logs deploy/openmrs
```

Wait for pods to become `Ready`. The readiness and liveness probes use `/openmrs/login.htm`.

## Production Hardening Checklist

- Use TLS at the ingress/controller level.
- Replace plaintext secret templates with your secret manager integration.
- Pin image tags to immutable digests.
- Configure backup and restore for the OpenMRS database.
- Add PodDisruptionBudget and HPA based on your SLOs.
- Enable centralized logging/metrics (Prometheus + Grafana).

## Rollback

```bash
kubectl -n openmrs rollout undo deployment/openmrs
```

## Remove

```bash
kubectl delete -f deploy/kubernetes/ingress.yaml
kubectl delete -f deploy/kubernetes/service.yaml
kubectl delete -f deploy/kubernetes/deployment.yaml
kubectl delete -f deploy/kubernetes/secret.example.yaml
kubectl delete -f deploy/kubernetes/configmap.yaml
kubectl delete -f deploy/kubernetes/namespace.yaml
```
