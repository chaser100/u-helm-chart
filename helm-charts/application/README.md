# Universal Helm Chart

A reusable universal Helm chart for deploying containerized applications to Kubernetes without maintaining a separate chart for every service.

## Features

- Deployments, Services, Ingress, and Gateway API HTTPRoutes
- Jobs, CronJobs, and Horizontal Pod Autoscaling
- ConfigMaps, Secrets, arbitrary volumes, and PersistentVolumeClaims
- Init containers, sidecars, lifecycle hooks, and custom manifests
- Optional Prometheus Operator ServiceMonitor
- Argo CD, Flux, and other GitOps workflows

## Requirements

- Helm 3
- Kubernetes 1.23+ for the complete built-in API set
- Gateway API CRDs when `route.enabled=true`
- Prometheus Operator CRDs when `serviceMonitor.enabled=true`

Optional CRDs are not installed by this chart.

## Install

Add the repository:

```bash
helm repo add universal https://chaser100.github.io/u-helm-chart
helm repo update
```

Create a minimal `values.yaml`:

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

Install or upgrade the application:

```bash
helm upgrade --install my-app universal/application \
  --namespace my-app \
  --create-namespace \
  -f values.yaml
```

Pin a chart version in CI/CD:

```bash
helm search repo universal/application --versions

helm upgrade --install my-app universal/application \
  --version <VERSION> \
  --namespace my-app \
  --create-namespace \
  -f values.yaml
```

## Connect an AI Agent with MCP

The project provides a hosted MCP knowledge service for AI-assisted chart configuration. It does not install anything into your Kubernetes cluster. The chart source of truth remains [chaser100/u-helm-chart](https://github.com/chaser100/u-helm-chart).

Server details:

- Endpoint: `https://helm.networkcat89.com/mcp`
- Transport: MCP Streamable HTTP
- Protocol version: `2025-06-18`
- Server name: `helm-networkcat89`
- Authentication: anonymous read access; write tools require an operator-provided Bearer token

Cursor and compatible MCP clients can use:

```json
{
  "mcpServers": {
    "helm-networkcat89": {
      "url": "https://helm.networkcat89.com/mcp"
    }
  }
}
```

Check index freshness and list the available tools:

```bash
curl -sS https://helm.networkcat89.com/healthz

curl -sS https://helm.networkcat89.com/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

Anonymous read tools include:

- `search_docs` for chart documentation
- `get_values_schema` and `explain_value` for values configuration
- `list_examples` and `get_example` for published examples
- `get_checklist` for the production checklist
- `list_shapes` and `get_shape` for workload shapes
- `resolve_resource` for `helm://...` resources

MCP resources are available through `resources/list` and `resources/read`. Authenticated write tools include `propose_improvement` and `open_feature_branch_pr`, which create GitHub issues or `mcp/**` pull requests.

Discovery metadata is published at [`/llms.txt`](https://helm.networkcat89.com/llms.txt). Non-MCP clients can use the OpenAPI read interface described by [`/openapi.json`](https://helm.networkcat89.com/openapi.json); its `operationId` values match the MCP tool names.

See [Connect an agent](https://helm.networkcat89.com/docs/connect-agent/) for the handshake example and setup instructions for Cursor, Claude, and generic HTTP MCP clients.

## Common Use Cases

### Deployment Strategy

The chart uses the Kubernetes default rollout behavior unless `deploymentStrategy` is set.

```yaml
deploymentStrategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Use `type: Recreate` for workloads that cannot run old and new Pods at the same time.

### Main Container Arguments and Multiple Ports

```yaml
args:
  - --config=/etc/application/config.yaml

revisionHistoryLimit: 2

containerPorts:
  - name: metrics
    containerPort: 8080
  - name: health
    containerPort: 8081

service:
  enabled: true
  ports:
    - name: metrics
      port: 8080
      targetPort: metrics
    - name: health
      port: 8081
      targetPort: health
```

Set `service.enabled: false` for workloads that do not need a chart-managed Service. Legacy single-port `service.name`, `service.port`, `service.targetPort`, `service.protocol`, and `service.appProtocol` values remain supported.

### Existing Service Account

Disable ServiceAccount creation and reference an account managed outside this chart:

```yaml
serviceAccount:
  create: false
  name: existing-service-account
```

When `name` is empty, the chart omits `serviceAccountName` from the Pod spec. Kubernetes then assigns the namespace `default` ServiceAccount.

### Gateway API HTTPRoute

Use the simple fields for a single path routed to the chart-managed Service:

```yaml
service:
  port: 8080

route:
  enabled: true
  gateway: public-gateway
  gatewayNamespace: gateway-system
  hostname: app.example.com
  path: /
```

For multiple matches, filters, traffic splitting, or other Gateway API options, set a complete `HTTPRoute.spec` under `route.spec`:

```yaml
fullnameOverride: my-app

service:
  port: 8080

route:
  enabled: true
  name: my-app
  spec:
    parentRefs:
      - name: public-gateway
        namespace: gateway-system
        sectionName: https
    hostnames:
      - app.example.com
    rules:
      - matches:
          - path:
              type: Exact
              value: /health
          - path:
              type: PathPrefix
              value: /api
        backendRefs:
          - name: my-app
            port: 8080
```

When `route.spec` is non-empty, it replaces the spec generated from `route.gateway`, `route.hostname`, `route.path`, and the other simple route fields. HTTPRoute metadata still comes from `route.name`, `route.labels`, and `route.annotations`.

### Service Application Protocol

The chart-managed Service accepts an optional application protocol hint:

```yaml
service:
  name: http
  port: 9005
  targetPort: http
  appProtocol: grpc
```

Services created for `ingressPlain` paths support the same field:

```yaml
ingressPlain:
  enabled: true
  items:
    - rules:
        - host: grpc.example.com
          paths:
            - path: /
              backend:
                service:
                  name: grpc-api
                  port: 9005
              createService: true
              service:
                port: 9005
                targetPort: http
                appProtocol: grpc
```

The chart omits `appProtocol` when it is not configured for either Service type.

### ExternalSecret Lifecycle

`ExternalSecret` objects in `extraManifests` are ordinary resources by default. The chart does not add Helm hooks automatically.

For Argo CD ordering, add a sync wave directly to the manifest:

```yaml
extraManifests:
  - apiVersion: external-secrets.io/v1beta1
    kind: ExternalSecret
    metadata:
      name: app-secrets
      annotations:
        argocd.argoproj.io/sync-wave: "-5"
    spec:
      # ExternalSecret spec
```

Set `externalSecretHooks.enabled: true` only when the legacy automatic Helm pre-install/pre-upgrade hook behavior is required.

### Horizontal Autoscaling

```yaml
replicaCount: 2

resources:
  requests:
    cpu: 200m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

### Database Migration Job

```yaml
job:
  enabled: true
  name: database-migration
  image: ghcr.io/example/my-app
  imageTag: "1.0.0"
  command: ["/bin/sh", "-c"]
  args: ["./app migrate"]
```

### Persistent Storage

```yaml
persistentVolumeClaims:
  - name: application-data
    size: 10Gi
    storageClassName: standard
    accessModes:
      - ReadWriteOnce
    mountPath: /var/lib/application
```

### Sidecar Container

```yaml
extraContainers:
  enabled: true
  containers:
    - name: log-collector
      image: fluent/fluent-bit:latest
      volumeMounts:
        - name: application-logs
          mountPath: /var/log/application

volumes:
  - name: application-logs
    emptyDir: {}
```

## Prometheus ServiceMonitor

The ServiceMonitor is disabled by default and does not affect clusters without the Prometheus Operator.

```yaml
service:
  name: http
  port: 9113

serviceMonitor:
  enabled: true
  labels:
    release: kube-prometheus-stack
  endpoints:
    - port: http
      path: /metrics
      interval: 30s
```

## Tested Configurations

- [Basic application](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-basic.yaml)
- [Container arguments and multiple ports](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-multi-port.yaml)
- [Deployment without Service](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-service-disabled.yaml)
- [Deployment with an existing ServiceAccount](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-service-account-disabled.yaml)
- [Deployment with the implicit namespace default ServiceAccount](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-service-account-implicit-default.yaml)
- [Deployment strategy](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-deployment-strategy.yaml)
- [Ingress](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-ingress.yaml)
- [Plain Ingress](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-ingress-plain.yaml)
- [Extra Ingress](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-ingress-extra.yaml)
- [Gateway API HTTPRoute](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-route.yaml) and [advanced HTTPRoute](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-route-advanced.yaml)
- [Autoscaling](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-autoscaling.yaml)
- [Jobs](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-job.yaml) and [CronJobs](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-cronjob.yaml)
- [Persistent storage and full configuration](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-full.yaml)
- [Sidecars](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-sidecar.yaml)
- [ServiceMonitor](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-servicemonitor.yaml)
- [ExternalSecret lifecycle](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-extra-manifests.yaml) and [opt-in Helm hooks](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-external-secret-hooks.yaml)
- [Extra Deployments](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-extra-deployments.yaml)
- [Custom manifests](https://github.com/chaser100/u-helm-chart/blob/main/helm-charts/application/tests/values-test-extra-manifests.yaml)

## Documentation

- [Configuration reference](https://helm.networkcat89.com/docs)
- [Connect an AI agent via MCP](https://helm.networkcat89.com/docs/connect-agent/)
- [Tested examples](https://helm.networkcat89.com/examples)
- [Interactive playground](https://helm.networkcat89.com/playground)
- [Source repository](https://github.com/chaser100/u-helm-chart)

## Uninstall

```bash
helm uninstall my-app --namespace my-app
```

## License

MIT
