# Helm Chart Engineering - Exercise 19

## Overview

This project demonstrates the creation of a reusable Helm chart for deploying a Kubernetes application. The chart supports multiple environments (Dev, QA, and Prod) and includes:

* Deployments
* Services
* ConfigMaps
* Secrets
* Ingress
* Horizontal Pod Autoscaling (HPA)
* Environment-specific configurations

---

# Prerequisites

Install Helm:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Verify installation:

```bash
helm version
```

---

# Create a New Helm Chart

```bash
helm create payment-service
```

Move into the chart directory:

```bash
cd payment-service
```

---

# Remove Unnecessary Files

Delete the default test directory:

```bash
rm -rf templates/tests
```

Optionally remove:

```bash
rm templates/httproute.yaml
```

if it is not required.

---

# Project Structure

```text
payment-service/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
├── values-prod.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── configmap.yaml
    ├── secret.yaml
    ├── ingress.yaml
    └── hpa.yaml
```

---

# Base Configuration

## values.yaml

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""

environment: dev

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

httpRoute:
  enabled: false

ingress:
  enabled: false

secret:
  dbPassword: test123
```

---

# Environment Configuration

## values-dev.yaml

```yaml
replicaCount: 1

environment: dev
```

## values-qa.yaml

```yaml
replicaCount: 2

environment: qa
```

## values-prod.yaml

```yaml
replicaCount: 5

environment: prod

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

ingress:
  enabled: true
  host: payment.example.com
```

---

# ConfigMap Template

File: `templates/configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: {{ .Release.Name }}-config

data:
  APP_ENV: {{ .Values.environment | quote }}
```

---

# Secret Template

File: `templates/secret.yaml`

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: {{ .Release.Name }}-secret

type: Opaque

stringData:
  DB_PASSWORD: {{ .Values.secret.dbPassword | quote }}
```

---

# Deployment Template

The Deployment uses:

* ConfigMap environment variables
* Resource requests and limits
* ServiceAccount integration
* Environment-specific replica counts

File: `templates/deployment.yaml`

(Insert deployment template here.)

---

# Horizontal Pod Autoscaler

File: `templates/hpa.yaml`

This template creates an HPA only when autoscaling is enabled.

Features:

* CPU-based scaling
* Configurable minimum replicas
* Configurable maximum replicas

---

# Validate the Chart

Lint the chart:

```bash
helm lint .
```

Render manifests:

```bash
helm template payment .
```

---

# Test Autoscaling

## Development Environment

```bash
helm template payment . -f values-dev.yaml
```

Expected:

* No HPA resource rendered

---

## Production Environment

```bash
helm template payment . -f values-prod.yaml
```

Expected:

```yaml
kind: HorizontalPodAutoscaler
```

---

# Deploy to Kubernetes

Create namespace:

```bash
kubectl create namespace dev
```

Install the chart:

```bash
helm install payment . \
-f values-dev.yaml \
-n dev
```

Verify deployment:

```bash
kubectl get all -n dev
```

---

# Upgrade Deployment

```bash
helm upgrade payment . \
-f values-dev.yaml \
-n dev
```

---

# Rollback Deployment

View release history:

```bash
helm history payment -n dev
```

Rollback:

```bash
helm rollback payment 1 -n dev
```

---

# Uninstall

```bash
helm uninstall payment -n dev
```

---

# Learning Outcomes

After completing this exercise, you will understand:

* Helm chart structure
* Templating using values.yaml
* Environment-specific deployments
* ConfigMaps and Secrets
* Ingress configuration
* Horizontal Pod Autoscaling
* Helm install, upgrade, rollback, and uninstall operations
* Kubernetes application packaging best practices

```
```

