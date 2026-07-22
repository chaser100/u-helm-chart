# Universal Helm Chart

[![Helm Chart Version](https://img.shields.io/github/v/release/chaser100/u-helm-chart?label=Helm%20chart)](https://github.com/chaser100/u-helm-chart/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-compatible-blue)](https://kubernetes.io/)

A reusable, production-ready universal Helm chart for Kubernetes applications.

Universal Helm Chart is a generic Helm chart and reusable Helm chart for Kubernetes applications. Deploy different containerized applications using a single configurable Helm chart instead of maintaining a separate chart for every service.

It supports Deployments, Services, Ingress, Gateway API HTTPRoutes, Jobs, CronJobs, autoscaling, ConfigMaps, Secrets, persistent volumes, init containers, sidecars, custom Kubernetes manifests, and more.

## Quick Start

```bash
helm repo add universal https://chaser100.github.io/u-helm-chart
helm repo update

helm install my-app universal/application \
  --set image=nginx \
  --set imageTag=latest
```

## Why Use This Chart?

Instead of maintaining one Helm chart per application, maintain one reusable chart and configure deployments entirely through `values.yaml`.

- Use one Helm chart across multiple applications
- Configure deployments entirely through `values.yaml`
- Reduce duplicated Helm templates between services
- Support simple workloads and advanced Kubernetes configurations
- Integrate with Helm, Argo CD, Flux, and other GitOps workflows
- Deploy applications with Helm without creating a new application chart each time

## Examples

Real-world deployment examples are available in the [`helm-charts/application/tests/`](helm-charts/application/tests/) directory.

- [Basic application](helm-charts/application/tests/values-test-basic.yaml)
- [Ingress](helm-charts/application/tests/values-test-ingress.yaml)
- [Gateway API](helm-charts/application/tests/values-test-route.yaml)
- [Jobs](helm-charts/application/tests/values-test-job.yaml)
- [CronJobs](helm-charts/application/tests/values-test-cronjob.yaml)
- [Persistent volumes](helm-charts/application/tests/values-test-full.yaml)
- [Sidecars](helm-charts/application/tests/values-test-sidecar.yaml)
- [Init containers](helm-charts/application/tests/values-test-full.yaml)
- [ConfigMaps and Secrets](helm-charts/application/tests/values-test-env-secrets.yaml)

## Documentation

- [Installation](#installation)
- [Why this chart?](docs/why-this-chart.md)
- [Configuration](docs/configuration.md)
- [Ingress](docs/configuration.md#ingress)
- [Gateway API](docs/configuration.md#gateway-api-route)
- [Jobs](docs/configuration.md#job)
- [CronJobs](docs/configuration.md#cronjob)
- [Persistent storage](docs/configuration.md#persistentvolumeclaims)
- [Usage examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

## Features

- **Flexible Deployment Configuration** - Deploy applications with customizable replica counts, resource limits, and security contexts
- **Ingress and Gateway API Routing** - Support for standard ingress, extra ingress, plain ingress with automatic service creation, and Gateway API HTTPRoute
- **Job and CronJob Support** - Run one-time jobs and scheduled cronjobs with full configuration options
- **Autoscaling** - Horizontal Pod Autoscaler (HPA) support with CPU and memory metrics
- **Security** - Configurable security contexts, service accounts, and pod security policies
- **Volume Management** - Support for ConfigMaps, Secrets, and PersistentVolumeClaims
- **Sidecar Containers** - Deploy additional containers alongside your main application
- **Custom Manifests** - Inject raw Kubernetes manifests for advanced use cases
- **Extra Deployments** - Define additional Deployment resources with optional image/imageTag inheritance from the main deployment
- **Environment Variables** - Flexible configuration for environment variables (plain values, secrets, and envFrom)
- **Lifecycle Hooks** - Optional `postStart` and `preStop` container lifecycle hooks for deployment

## Installation

This chart is published via GitHub Pages and can be used as an external Helm repository.

### Add the Helm Repository

```bash
helm repo add universal https://chaser100.github.io/u-helm-chart
helm repo update
```

### Install the Chart

```bash
# Basic installation
helm install my-release universal/application
```

Minimal `values.yaml` for a Kubernetes application:

```yaml
image: ghcr.io/example/my-app
imageTag: "1.0.0"

service:
  port: 8080

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
```

```bash
# Installation with custom values
helm install my-release universal/application -f my-values.yaml

# Installation with specific version
helm install my-release universal/application --version 0.3.4
```

### Upgrade the Chart

```bash
helm upgrade my-release universal/application -f my-values.yaml
```

### Uninstall the Chart

```bash
helm uninstall my-release
```

## Configuration

Detailed parameter reference lives in [Configuration](docs/configuration.md).

- [Deployment configuration](docs/configuration.md#deployment-configuration)
- [Service](docs/configuration.md#service)
- [Ingress](docs/configuration.md#ingress)
- [Gateway API](docs/configuration.md#gateway-api-route)
- [Jobs](docs/configuration.md#job)
- [CronJobs](docs/configuration.md#cronjob)
- [Persistent storage](docs/configuration.md#persistentvolumeclaims)
- [Sidecars](docs/configuration.md#sidecar-containers)
- [Init containers](docs/configuration.md#init-containers)
- [Extra deployments](docs/configuration.md#extra-deployments)

## Usage Examples

### Basic Web Application

```yaml
replicaCount: 2
image: nginx
imageTag: 1.21
service:
  type: ClusterIP
  port: 80
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

### Application with Database Migration Job

```yaml
replicaCount: 3
image: myapp
imageTag: v1.0.0
job:
  enabled: true
  name: db-migration
  image: myapp
  imageTag: v1.0.0
  command: ["/bin/sh", "-c"]
  args: ["npm run migrate"]
  envFromSecrets:
    - name: DATABASE_URL
      secretName: db-secret
      secretKey: url
```

### Application with Autoscaling

```yaml
replicaCount: 2
image: myapp
imageTag: v1.0.0
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 512Mi
```

### Application with Init Container and Individual PVC

```yaml
replicaCount: 2
image: myapp
imageTag: v1.0.0
persistentVolumeClaims:
  - name: data-storage
    size: 10Gi
    mountPath: /data

initContainers:
  enabled: true
  containers:
    - name: init-migration
      image: myapp
      imageTag: v1.0.0
      command: ["/bin/sh", "-c"]
      args: ["python manage.py migrate"]
      envFromSecrets:
        - name: DATABASE_URL
          secretName: db-secret
          secretKey: url
      pvc:
        enabled: true
        name: init-migration-pvc
        size: 5Gi
        storageClassName: standard
        accessModes:
          - ReadWriteOnce
        mountPath: /mnt/init
```

### Application with PersistentVolumeClaim

```yaml
replicaCount: 2
image: myapp
imageTag: v1.0.0
persistentVolumeClaims:
  - name: data-storage
    size: 10Gi
    storageClassName: fast-ssd
    accessModes:
      - ReadWriteOnce
    mountPath: /data
  - name: logs-storage
    size: 5Gi
    storageClassName: standard
    accessModes:
      - ReadWriteOnce
    mountPath: /var/log
```

### Application with PVC in Init Container

```yaml
replicaCount: 2
image: myapp
imageTag: v1.0.0
persistentVolumeClaims:
  - name: data-storage
    size: 10Gi
    mountPath: /data
initContainers:
  enabled: true
  containers:
    - name: init-migration
      image: myapp
      imageTag: v1.0.0
      command: ["/bin/sh", "-c"]
      args: ["python manage.py migrate"]
      volumeMounts:
        - name: data-storage
          mountPath: /data
```

### Application with PVC in Sidecar Container

```yaml
replicaCount: 1
image: myapp
imageTag: v1.0.0
persistentVolumeClaims:
  - name: logs-storage
    size: 5Gi
    mountPath: /var/log
extraContainers:
  enabled: true
  containers:
    - name: log-collector
      image: fluent/fluent-bit:latest
      volumeMounts:
        - name: logs-storage
          mountPath: /var/log
```

### Application with Sidecar Container

```yaml
replicaCount: 1
image: myapp
imageTag: v1.0.0
extraContainers:
  enabled: true
  containers:
    - name: log-collector
      image: fluent/fluent-bit:latest
      volumeMounts:
        - name: varlog
          mountPath: /var/log
volumes:
  - name: varlog
    hostPath:
      path: /var/log
```

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -l app.kubernetes.io/name=application
```

### View Pod Logs

```bash
kubectl logs -l app.kubernetes.io/name=application
```

### Describe Pod for Events

```bash
kubectl describe pod <pod-name>
```

### Check Service Endpoints

```bash
kubectl get endpoints <service-name>
```

### Test Ingress

```bash
kubectl get ingress
curl -H "Host: example.com" http://<ingress-ip>/
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for a list of changes and version history.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
