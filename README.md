# vProfile Helm & GitOps Repository

This repository hosts the Helm chart and ArgoCD application manifests for deploying the **vProfile** multi-tier web application stack onto Kubernetes (Amazon EKS).

## Application Architecture

The stack consists of:
- **vproapp**: Java Spring MVC application running on Apache Tomcat (port 8080)
- **vprodb**: MySQL database with persistent storage on AWS EBS (port 3306)
- **vprocache01**: Memcached caching service (port 11211)
- **vpromq01**: RabbitMQ message broker (port 5672)
- **Ingress**: AWS Application Load Balancer (ALB) via AWS Load Balancer Controller with SSL termination

## Repository Structure

```
├── Chart.yaml                  # Helm Chart metadata
├── values.yaml                 # Configuration values (updated automatically by CI)
├── templates/                  # Kubernetes resource templates
│   ├── app-deployment.yaml
│   ├── db-deployment.yaml
│   ├── mc-deployment.yaml
│   ├── rmq-deployment.yaml
│   ├── services.yaml
│   ├── ingress.yaml
│   ├── pvc.yaml
│   ├── secret.yaml
│   └── dockerregistry-secret.yaml
├── argocd/                     # ArgoCD GitOps definitions
│   ├── projects/
│   │   └── vprofile-project.yaml   # ArgoCD AppProject
│   └── apps/
│       └── vprofile-app.yaml       # ArgoCD Application (syncs this repository)
└── kubedefs/                   # Raw Kubernetes manifests (reference)
```

## GitOps Workflow

1. A merge to `main` in `vprofile-app` triggers the GitHub Actions CI pipeline.
2. The pipeline builds a multi-stage Docker image, tags it with the commit SHA, and pushes it to Amazon ECR.
3. The pipeline updates `app.image` and `app.tag` in `values.yaml` in this repository and commits the change.
4. **ArgoCD** detects the new commit in this repository and automatically synchronizes the cluster to deploy the new version with zero downtime.

## Deploying via ArgoCD

Apply the ArgoCD project and application manifests:

```bash
kubectl apply -f argocd/projects/vprofile-project.yaml
kubectl apply -f argocd/apps/vprofile-app.yaml
```
