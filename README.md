# Universal Helm Chart

[![Helm Chart Version](https://img.shields.io/github/v/release/chaser100/u-helm-chart?label=Helm%20chart)](https://github.com/chaser100/u-helm-chart/releases)
[![Helm Tests](https://github.com/chaser100/u-helm-chart/actions/workflows/helm-tests.yml/badge.svg)](https://github.com/chaser100/u-helm-chart/actions/workflows/helm-tests.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-compatible-blue)](https://kubernetes.io/)

🌐 **Website:** https://helm.networkcat89.com

📖 **Documentation:** https://helm.networkcat89.com/docs

🎮 **Playground:** https://helm.networkcat89.com/playground

⭐ **Examples:** https://helm.networkcat89.com/examples

A reusable, production-ready universal Helm chart for deploying applications to Kubernetes.

Universal Helm Chart is a generic Helm chart and reusable application Helm chart for Kubernetes. Deploy different containerized applications with one configurable chart instead of maintaining a separate chart for every service.

It supports Deployments, Services, Ingress, Gateway API HTTPRoutes, Prometheus Operator ServiceMonitors, Jobs, CronJobs, autoscaling, ConfigMaps, Secrets, persistent volumes, init containers, sidecars, custom Kubernetes manifests, and more.

## Quick Start

```bash
helm repo add universal https://chaser100.github.io/u-helm-chart
helm repo update

helm upgrade --install my-app universal/application \
  --namespace my-app \
  --create-namespace \
  --set image=nginx \
  --set imageTag=1.27
```

## Why Use This Chart?

Instead of maintaining one Helm chart per application, maintain one reusable chart and configure deployments entirely through `values.yaml`.

- Use one Helm chart across multiple applications
- Reduce duplicated Kubernetes templates between services
- Keep application-specific configuration in `values.yaml`
- Support both simple workloads and advanced Kubernetes configurations
- Integrate with Helm, Argo CD, Flux, and other GitOps workflows
- Deploy applications without creating a new application chart each time

See [Why This Chart?](docs/why-this-chart.md) for design goals, trade-offs, and cases where a dedicated chart is a better fit.

## Tested Examples

The [`tests/` directory](helm-charts/application/tests/) contains tested, ready-to-use `values.yaml` examples and is the source of truth for supported configurations. Every example is linted and rendered in CI.

- [Basic application](helm-charts/application/tests/values-test-basic.yaml)
- [Ingress](helm-charts/application/tests/values-test-ingress.yaml) and [plain Ingress](helm-charts/application/tests/values-test-ingress-plain.yaml)
- [Gateway API HTTPRoute](helm-charts/application/tests/values-test-route.yaml)
- [Prometheus Operator ServiceMonitor](helm-charts/application/tests/values-test-servicemonitor.yaml)
- [Jobs](helm-charts/application/tests/values-test-job.yaml) and [CronJobs](helm-charts/application/tests/values-test-cronjob.yaml)
- [Storage and full configuration](helm-charts/application/tests/values-test-full.yaml)
- [Sidecars](helm-charts/application/tests/values-test-sidecar.yaml)

See all tested configurations in [`helm-charts/application/tests/`](helm-charts/application/tests/).

## Documentation

- [Why this chart?](docs/why-this-chart.md)
- [Configuration reference](docs/configuration.md)
- [Ingress](docs/configuration.md#ingress)
- [Gateway API](docs/configuration.md#gateway-api-route)
- [Jobs](docs/configuration.md#job)
- [CronJobs](docs/configuration.md#cronjob)
- [Persistent storage](docs/configuration.md#persistentvolumeclaims)
- [Sidecars](docs/configuration.md#sidecar-containers)
- [Init containers](docs/configuration.md#init-containers)
- [Extra deployments](docs/configuration.md#extra-deployments)

## Features

- **Flexible deployments** - Configure replicas, commands, probes, resources, lifecycle hooks, and scheduling
- **Ingress and Gateway API** - Support standard Ingress, extra Ingress, plain Ingress, and Gateway API HTTPRoute
- **Prometheus monitoring** - Optionally create a Prometheus Operator ServiceMonitor for the chart-managed Service
- **Jobs and CronJobs** - Run one-time and scheduled workloads with dedicated configuration
- **Autoscaling** - Scale Deployments with `autoscaling/v2` HorizontalPodAutoscaler resources
- **Security configuration** - Configure security contexts, service accounts, and Pod Security Standards-compatible settings
- **Storage** - Mount ConfigMaps, Secrets, arbitrary volumes, and PersistentVolumeClaims
- **Init and sidecar containers** - Run supporting containers alongside the main application
- **Custom resources** - Add raw Kubernetes manifests and extra Deployments when needed
- **GitOps workflows** - Use the same chart with Helm, Argo CD, Flux, and other deployment tools

## Requirements

- Helm 3
- Kubernetes 1.23+ for the complete built-in API set used by the chart, including `autoscaling/v2`
- Gateway API CRDs and a compatible Gateway controller when `route.enabled=true`
- Prometheus Operator ServiceMonitor CRD when `serviceMonitor.enabled=true`

Gateway API and Prometheus Operator CRDs are not installed by this chart. Install and manage the required CRDs separately before enabling HTTPRoute or ServiceMonitor resources.

## Installation

This chart is published through GitHub Pages as a Helm repository.

### Add the Helm Repository

```bash
helm repo add universal https://chaser100.github.io/u-helm-chart
helm repo update
```

### Create Application Values

```yaml
# values.yaml
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

### Install or Upgrade

Use the same idempotent command for the initial installation and later upgrades:

```bash
helm upgrade --install my-app universal/application \
  --namespace my-app \
  --create-namespace \
  -f values.yaml
```

To pin a chart version in CI/CD, list available versions and pass the selected version explicitly:

```bash
helm search repo universal/application --versions

helm upgrade --install my-app universal/application \
  --version <VERSION> \
  --namespace my-app \
  --create-namespace \
  -f values.yaml
```

### Uninstall

```bash
helm uninstall my-app --namespace my-app
```

## Configuration

The complete parameter reference and examples live in [Configuration](docs/configuration.md). Prefer the tested files in [`helm-charts/application/tests/`](helm-charts/application/tests/) when starting a new deployment.

## Troubleshooting

The commands below assume the release name and namespace are both `my-app`.

### Check Workload Status

```bash
kubectl get pods \
  --namespace my-app \
  --selector app.kubernetes.io/instance=my-app
```

### Describe Pods and Events

```bash
kubectl describe pods \
  --namespace my-app \
  --selector app.kubernetes.io/instance=my-app
```

### View Logs

```bash
kubectl logs \
  --namespace my-app \
  --selector app.kubernetes.io/instance=my-app \
  --all-containers=true
```

### Check Services and Routes

```bash
kubectl get services,endpoints,ingresses,httproutes \
  --namespace my-app \
  --selector app.kubernetes.io/instance=my-app
```

### Test Ingress

```bash
kubectl get ingress --namespace my-app
curl -H "Host: app.example.com" http://<INGRESS_IP>/
```

## Contributing

Contributions are welcome. Please open an issue or submit a pull request with a tested `values.yaml` example for user-visible chart behavior.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for release history.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
