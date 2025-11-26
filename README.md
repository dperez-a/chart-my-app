# chart-my-app
Modular Helm chart for deploying a full-stack application with frontend, backend, and database components. Includes production and development configurations with customizable values and persistent storage.

# my-app Helm Chart

Modular Helm chart for deploying a full-stack application with frontend, backend, and MySQL database components.

## Prerequisites

- Kubernetes cluster (Minikube, kind, or cloud provider)
- Helm 3.x installed
- kubectl configured

## Quick Start

### Install Helm (if not installed)

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows
choco install kubernetes-helm
```

### Deploy the Chart

```bash
# Default configuration
helm install my-release ./my-app

# Development environment
helm install my-release ./my-app -f my-app/values-dev.yaml

# Production environment
helm install my-release ./my-app -f my-app/values-prod.yaml
```

## Configuration

### Default Values (values.yaml)
- Frontend: 2 replicas, ClusterIP service
- Backend: 2 replicas, ClusterIP service
- Database: 1 replica, 1Gi persistent storage

### Development (values-dev.yaml)
- All components: 1 replica
- Images: dev tags with Always pull policy
- Database: No persistent storage (ephemeral)
- Nginx: NodePort 30080

### Production (values-prod.yaml)
- Frontend/Backend: 3 replicas
- Images: stable version tags
- Database: 5Gi persistent storage
- Enhanced resource limits
- Nginx: LoadBalancer service

## Components

| Component | Port | Service Type | Description |
|-----------|------|--------------|-------------|
| Frontend  | 80   | ClusterIP    | Web interface |
| Backend   | 5000 | ClusterIP    | REST API |
| Database  | 3306 | ClusterIP    | MySQL 8.0 |
| Nginx     | 80   | NodePort/LB  | Reverse proxy |

## Useful Commands

```bash
# Validate chart
helm lint ./my-app

# Dry run (see generated manifests)
helm install my-release ./my-app --dry-run --debug

# List releases
helm list

# Get release status
helm status my-release

# Upgrade release
helm upgrade my-release ./my-app

# Rollback
helm rollback my-release

# Uninstall
helm uninstall my-release
```

## Customization

Override specific values without modifying files:

```bash
# Change replica count
helm install my-release ./my-app --set frontend.replicaCount=5

# Change image tags
helm install my-release ./my-app \
  --set frontend.image.tag=v2.0 \
  --set backend.image.tag=v2.0

# Disable persistence
helm install my-release ./my-app --set database.persistence.enabled=false
```

## Database Setup

After deployment, configure the database:

```bash
# Get database pod name
kubectl get pods

# Access MySQL shell
kubectl exec -it  -- mysql -u root -p

# Run SQL commands
CREATE USER 'appuser'@'%' IDENTIFIED BY 'apppassword';
GRANT ALL PRIVILEGES ON *.* TO 'appuser'@'%';
CREATE DATABASE test_db;
USE test_db;
CREATE TABLE message (id INT AUTO_INCREMENT PRIMARY KEY, text VARCHAR(255) NOT NULL);
INSERT INTO message (text) VALUES ('Hello from Helm!');
```

## Accessing the Application

```bash
# Using NodePort (development)
minikube service my-release-nginx

# Using port-forward
kubectl port-forward svc/my-release-nginx 8080:80
# Access: http://localhost:8080

# Using LoadBalancer (production)
kubectl get svc my-release-nginx
# Use EXTERNAL-IP
```

## Troubleshooting

```bash
# Check all resources
kubectl get all

# Check specific component
kubectl get pods -l app=frontend
kubectl logs -l app=backend

# Describe resources
kubectl describe deployment my-release-frontend

# Check PVC status
kubectl get pvc
```

## Chart Structure

```
my-app/
├── Chart.yaml              # Chart metadata
├── values.yaml            # Default values
├── values-dev.yaml        # Development config
├── values-prod.yaml       # Production config
└── templates/
    ├── frontend/          # Frontend manifests
    ├── backend/           # Backend manifests
    └── database/          # Database manifests
```

## Best Practices Implemented

✅ Modular template structure
✅ Parameterized configurations via values.yaml
✅ Environment-specific value files
✅ Resource limits and requests defined
✅ Persistent storage with PVC
✅ Service discovery using DNS
✅ Labels for resource organization
✅ Helm hooks for lifecycle management