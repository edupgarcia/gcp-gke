# GCP GKE Demo Application (gceme)

A demonstration application showcasing CI/CD pipelines with Jenkins and Kubernetes on Google Kubernetes Engine (GKE). The "gceme" (Google Cloud Engine Metadata Explorer) application displays instance metadata information and demonstrates modern cloud-native deployment practices.

## Overview

This project demonstrates:
- **Cloud-native application development** with Go
- **Container orchestration** using Kubernetes
- **Continuous Integration/Continuous Deployment** with Jenkins
- **Multi-environment deployments** (dev, canary, production)
- **Load balancing** and service discovery
- **Google Cloud Platform** best practices

## Architecture

The application consists of two components:

### Backend Service
- **Port**: 8080 (default)
- **Function**: Collects and returns GCE instance metadata as JSON
- **Endpoints**:
  - `/` - Returns instance metadata
  - `/version` - Returns application version (2.0.0)
  - `/healthz` - Health check endpoint

### Frontend Service
- **Port**: 8080 (default)
- **Function**: Queries backend service and displays metadata in HTML format
- **Endpoints**:
  - `/` - Displays formatted instance information
  - `/healthz` - Health check with backend connectivity test

## Project Structure

```
.
├── main.go                 # Main application code
├── html.go                # HTML template for frontend
├── main_test.go           # Unit tests
├── Dockerfile             # Container image definition
├── Gopkg.toml            # Go dependency management
├── Gopkg.lock            # Dependency lock file
├── Jenkinsfile           # Jenkins CI/CD pipeline definition
├── k8s/                  # Kubernetes manifests
│   ├── services/         # Service definitions
│   │   ├── backend.yaml
│   │   └── frontend.yaml
│   ├── dev/              # Development environment
│   │   ├── backend-dev.yaml
│   │   ├── frontend-dev.yaml
│   │   └── default.yml
│   ├── canary/           # Canary deployment
│   │   ├── backend-canary.yaml
│   │   └── frontend-canary.yaml
│   └── production/       # Production environment
│       ├── backend-production.yaml
│       └── frontend-production.yaml
└── vendor/               # Go dependencies
```

## Getting Started

### Prerequisites

- Go 1.10+
- Docker
- Kubernetes cluster (GKE recommended)
- Jenkins (for CI/CD)
- `kubectl` configured for your cluster

### Local Development

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd gcp-gke
   ```

2. **Install dependencies**:
   ```bash
   dep ensure
   ```

3. **Run tests**:
   ```bash
   go test
   ```

4. **Run backend locally**:
   ```bash
   go run *.go -port=8080
   ```

5. **Run frontend locally**:
   ```bash
   go run *.go -frontend -port=8080 -backend-service=http://localhost:8081
   ```

### Docker Build

Build the container image:
```bash
docker build -t gceme:latest .
```

Run the container:
```bash
# Backend
docker run -p 8080:8080 gceme:latest

# Frontend
docker run -p 8080:8080 gceme:latest app -frontend -backend-service=http://backend:8080
```

## Kubernetes Deployment

### Manual Deployment

1. **Deploy services**:
   ```bash
   kubectl apply -f k8s/services/
   ```

2. **Deploy to production**:
   ```bash
   kubectl apply -f k8s/production/
   ```

3. **Deploy to canary** (for testing):
   ```bash
   kubectl apply -f k8s/canary/
   ```

### Environment-Specific Deployments

- **Production**: Uses `LoadBalancer` service type for external access
- **Canary**: Limited deployment for testing new versions
- **Development**: Uses `ClusterIP` for internal-only access

## CI/CD Pipeline

The Jenkins pipeline (`Jenkinsfile`) automates:

### Pipeline Stages

1. **Test**: Runs Go unit tests
2. **Build and Push**: Creates container image using Google Cloud Build
3. **Deploy Canary**: Deploys to canary environment (canary branch)
4. **Deploy Production**: Deploys to production (master branch)
5. **Deploy Dev**: Deploys to branch-specific namespace (feature branches)

### Branch Strategy

- **`master`**: Triggers production deployment
- **`canary`**: Triggers canary deployment for testing
- **Feature branches**: Deploy to isolated development namespaces

### Environment Variables

Configure these in your Jenkins environment:
- `PROJECT`: GCP Project ID
- `CLUSTER`: GKE cluster name
- `CLUSTER_ZONE`: GKE cluster zone
- `JENKINS_CRED`: Jenkins credentials ID for GCP

## Application Features

### Metadata Display

The application shows:
- Instance ID and name
- Project information
- Zone and hostname
- Internal/External IP addresses
- HTTP request details
- Load balancer information

### Health Monitoring

- Backend health check at `/healthz`
- Frontend health check includes backend connectivity
- Kubernetes readiness probes configured

## Configuration Options

### Command Line Flags

```bash
app [options]
  -version          Display version information
  -frontend         Run in frontend mode
  -port int         Port to bind (default 8080)
  -backend-service  Backend service URL (default "http://127.0.0.1:8081")
```

### Kubernetes Configuration

Deployments include:
- Resource limits (CPU: 100m, Memory: 500Mi)
- Readiness probes
- Rolling update strategy
- Service discovery via DNS

## Monitoring and Observability

### Health Checks

- **Readiness probes**: `/healthz` endpoint
- **Service connectivity**: Frontend tests backend availability
- **Resource monitoring**: CPU and memory limits set

### Logging

Application logs include:
- Startup mode (frontend/backend)
- Request processing information
- Error handling and reporting

## Security Considerations

- Non-root container execution
- Resource limits prevent resource exhaustion
- Service account permissions in Kubernetes
- Network policies can be applied for additional security

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Commit with descriptive messages
6. Push to your fork and create a Pull Request

## Troubleshooting

### Common Issues

1. **"Not running on GCE" error**:
   - Expected when running outside GCE
   - Metadata endpoints only available on GCE instances

2. **Backend connection failures**:
   - Check service discovery configuration
   - Verify backend service is running and healthy

3. **Image pull errors**:
   - Ensure image exists in specified registry
   - Check registry permissions

### Debug Commands

```bash
# Check pod status
kubectl get pods -l app=gceme

# View logs
kubectl logs -l app=gceme,role=backend
kubectl logs -l app=gceme,role=frontend

# Test service connectivity
kubectl exec -it <pod-name> -- curl http://gceme-backend:8080/healthz
```

## License

Copyright 2015 Google Inc. Licensed under the Apache License, Version 2.0.

## Version

Current version: **2.0.0**

---

*This project demonstrates Google Cloud Platform best practices for containerized applications, CI/CD pipelines, and Kubernetes deployments.*
