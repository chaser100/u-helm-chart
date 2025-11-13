# Universal Helm Chart

A comprehensive and flexible Helm chart for deploying applications to Kubernetes. This chart provides a universal template that supports a wide range of deployment scenarios including deployments, services, ingress, jobs, cronjobs, and advanced features like autoscaling, sidecar containers, and custom manifests.

**Chart Version:** 0.2.0

## Features

- **Flexible Deployment Configuration** - Deploy applications with customizable replica counts, resource limits, and security contexts
- **Multiple Ingress Options** - Support for standard ingress, extra ingress, and plain ingress with automatic service creation
- **Job and CronJob Support** - Run one-time jobs and scheduled cronjobs with full configuration options
- **Autoscaling** - Horizontal Pod Autoscaler (HPA) support with CPU and memory metrics
- **Security** - Configurable security contexts, service accounts, and pod security policies
- **Volume Management** - Support for ConfigMaps, Secrets, and PersistentVolumeClaims
- **Sidecar Containers** - Deploy additional containers alongside your main application
- **Custom Manifests** - Inject raw Kubernetes manifests for advanced use cases
- **Environment Variables** - Flexible configuration for environment variables (plain values, secrets, and envFrom)

## Installation

This chart is published via GitHub Pages and can be used as an external Helm repository.

### Add the Helm Repository

```bash
helm repo add uni-chart https://chaser100.github.io/u-helm-chart
helm repo update
```

### Install the Chart

```bash
# Basic installation
helm install my-release uni-chart/application

# Installation with custom values
helm install my-release uni-chart/application -f my-values.yaml

# Installation with specific version
helm install my-release uni-chart/application --version 0.1.9
```

### Upgrade the Chart

```bash
helm upgrade my-release uni-chart/application -f my-values.yaml
```

### Uninstall the Chart

```bash
helm uninstall my-release
```

## Configuration

The following table lists the configurable parameters and their default values. You can override these values by creating a custom `values.yaml` file or using the `--set` flag during installation.

### Deployment Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas for the deployment | `1` |
| `image` | Container image name | `nginx` |
| `imageTag` | Container image tag | `latest` |
| `imagePullPolicy` | Image pull policy | `Always` |
| `imagePullSecrets` | List of image pull secrets for private registries | `[]` |
| `nameOverride` | Override the name of the chart | `""` |
| `fullnameOverride` | Override the full name of the chart | `""` |
| `deploymentAnnotations` | Additional annotations for the deployment | `[]` |
| `podAnnotations` | Additional annotations for pods | `{}` |
| `podLabels` | Additional labels for pods | `{}` |

#### Example

```yaml
replicaCount: 3
image: myregistry/myapp
imageTag: v1.2.3
imagePullPolicy: IfNotPresent
imagePullSecrets:
  - name: regcred
```

### Service Account

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Whether to create a service account | `true` |
| `serviceAccount.automount` | Automatically mount the service account token | `true` |
| `serviceAccount.annotations` | Additional annotations for the service account | `{}` |
| `serviceAccount.name` | Name of the service account (if not set, uses fullname) | `""` |

#### Example

```yaml
serviceAccount:
  create: true
  automount: true
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/my-role
  name: my-service-account
```

### Security Context

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext` | Security context for the pod | `{}` |
| `securityContext` | Security context for the container | `{}` |

#### Example

```yaml
podSecurityContext:
  fsGroup: 2000
  runAsNonRoot: true
  runAsUser: 1000

securityContext:
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
```

### Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.name` | Name of the service port | `http` |
| `service.type` | Type of service (ClusterIP, NodePort, LoadBalancer) | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.protocol` | Service protocol | `TCP` |

#### Example

```yaml
service:
  name: http
  type: LoadBalancer
  port: 8080
  protocol: TCP
```

### Ingress

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress controller class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | List of ingress hosts and paths | See values.yaml |
| `ingress.tls` | TLS configuration for ingress | `[]` |

#### Example

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: example.com
      paths:
        - path: /
          pathType: Prefix
    - host: api.example.com
      paths:
        - path: /api
          pathType: Prefix
  tls:
    - secretName: example-tls
      hosts:
        - example.com
        - api.example.com
```

### Extra Ingress

An additional ingress resource with the same structure as the main ingress.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `extra.ingress.enabled` | Enable extra ingress | `false` |
| `extra.ingress.className` | Ingress controller class name | `""` |
| `extra.ingress.annotations` | Ingress annotations | `{}` |
| `extra.ingress.hosts` | List of ingress hosts and paths | See values.yaml |
| `extra.ingress.tls` | TLS configuration for ingress | `[]` |

### Plain Ingress

A flexible feature for creating arbitrary Ingress resources with advanced configuration, including automatic service creation.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingressPlain.enabled` | Enable plain ingress | `false` |
| `ingressPlain.items` | List of ingress objects | `[]` |

Each item in `ingressPlain.items` supports:

- `name` - Ingress name (auto-generated if not specified)
- `className` - Ingress controller class
- `annotations` - Ingress annotations
- `labels` - Additional labels
- `tls` - TLS configuration:
  - `secretName` - TLS secret name
  - `hosts` - List of hosts
- `rules` - List of ingress rules:
  - `host` - Domain name
  - `paths` - List of paths:
    - `path` - Route path (e.g., `/api`)
    - `pathType` - Path type (`Prefix`, `Exact`, `ImplementationSpecific`)
    - `backend.service.name` - Backend service name
    - `backend.service.port` - Backend service port
    - `createService` - Whether to automatically create a Service for this path
    - `service` - Service configuration (optional):
      - `type` - Service type (`ClusterIP`, `NodePort`, `LoadBalancer`)
      - `port`, `targetPort`, `portName`, `protocol` - Service port configuration
      - `selector` - Pod selector labels
      - `annotations`, `labels` - Service metadata

#### Example

```yaml
ingressPlain:
  enabled: true
  items:
    - name: public-api
      className: nginx
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
        cert-manager.io/cluster-issuer: letsencrypt-prod
      rules:
        - host: api.example.com
          paths:
            - path: /
              backend:
                service:
                  name: users-service
                  port: 8080
              createService: true
              service:
                type: ClusterIP
                selector:
                  app: users
                port: 8080
                targetPort: 8080
            - path: /orders
              backend:
                service:
                  name: orders-service
                  port: 9090
              createService: false
      tls:
        - secretName: api-tls
          hosts:
            - api.example.com
```

### Health Probes

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe` | Liveness probe configuration | `{}` |
| `readinessProbe` | Readiness probe configuration | `{}` |
| `startupProbe` | Startup probe configuration | `{}` |

#### Example

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3

startupProbe:
  httpGet:
    path: /startup
    port: http
  initialDelaySeconds: 10
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 30
```

### Autoscaling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable Horizontal Pod Autoscaler | `false` |
| `autoscaling.minReplicas` | Minimum number of replicas | `1` |
| `autoscaling.maxReplicas` | Maximum number of replicas | `100` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization percentage | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | Target memory utilization percentage (optional) | `null` |

#### Example

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
```

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.requests` | Resource requests (CPU, memory) | `{}` |
| `resources.limits` | Resource limits (CPU, memory) | `{}` |

#### Example

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

### ConfigMaps

Mount ConfigMaps as volumes in the pod.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `configMaps` | List of ConfigMap configurations | `{}` |

Each ConfigMap entry supports:

- `name` - ConfigMap name
- `mountPath` - Mount path in the container
- `readOnly` - Whether the volume is read-only
- `defaultMode` - Default file mode (e.g., `0440`)
- `subPath` - Subpath within the volume (optional)
- `items` - List of items to include (optional)
- `data` - ConfigMap data

#### Example

```yaml
configMaps:
  - name: app-config
    mountPath: /etc/app-config
    readOnly: true
    defaultMode: 0440
    data:
      app.conf: |
        debug = true
        log_level = info
  - name: partial-config
    mountPath: /etc/partial
    subPath: app.conf
    readOnly: false
    items:
      - key: app.conf
        path: config-file
    data:
      app.conf: |
        key=value
```

### Volumes

Additional volumes for mounting ConfigMaps, Secrets, PVCs, etc.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `volumes` | List of volume definitions | `{}` |
| `volumeMounts` | List of volume mount definitions | `{}` |

#### Example

```yaml
volumes:
  - name: custom-secret
    secret:
      secretName: mysecret
      optional: false
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: my-pvc

volumeMounts:
  - name: custom-secret
    mountPath: "/etc/secrets"
    readOnly: true
  - name: persistent-storage
    mountPath: "/data"
```

### Pod Scheduling

| Parameter | Description | Default |
|-----------|-------------|---------|
| `nodeSelector` | Node selector labels | `[]` |
| `tolerations` | Pod tolerations | `{}` |
| `affinity` | Pod affinity/anti-affinity rules | `{}` |

#### Example

```yaml
nodeSelector:
  node-role.kubernetes.io/app: ""

tolerations:
  - key: "node-role.kubernetes.io/app"
    operator: "Exists"
    effect: "NoSchedule"

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - application
          topologyKey: kubernetes.io/hostname
```

### Sidecar Containers

Deploy additional containers alongside the main application container.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `extraContainers.enabled` | Enable sidecar containers | `false` |
| `extraContainers.containers` | List of sidecar container definitions | `[]` |

#### Example

```yaml
extraContainers:
  enabled: true
  containers:
    - name: sidecar
      image: busybox:latest
      command: ["sleep", "3600"]
      env:
        - name: SIDECAR_ENV
          value: production
      envFromSecrets:
        - name: BAR
          secretName: example-external-secret
          secretKey: BAR
      volumeMounts:
        - name: my-config
          mountPath: /mnt/config
      resources:
        requests:
          cpu: 50m
          memory: 64Mi
        limits:
          cpu: 100m
          memory: 128Mi
```

**Note:** Each sidecar container can have its own `env` variables. Global `env` variables from the root level are also inherited by all sidecar containers.

### Environment Variables

The chart supports three ways to configure environment variables:
1. **Plain environment variables** - Simple key-value pairs
2. **Environment variables from secrets** - Load specific keys from Kubernetes secrets
3. **Environment variables from secrets (envFrom)** - Load all keys from a secret

#### Plain Environment Variables

| Parameter | Description | Default |
|-----------|-------------|---------|
| `env` | List of plain environment variables | `[]` |

This is useful for non-sensitive configuration values or when using tools like Bank Vaults webhooks that will mutate the deployment and add secret values later.

#### Example

```yaml
env:
  - name: ENVIRONMENT
    value: production
  - name: LOG_LEVEL
    value: info
  - name: APP_NAME
    value: my-app
  - name: API_HOST
    value: api.example.com
```

#### Environment Variables from Secrets

| Parameter | Description | Default |
|-----------|-------------|---------|
| `envFromSecrets.enableEnvFrom` | Enable loading all keys from a secret as env vars | `false` |
| `envFromSecrets.secretName` | Secret name for envFrom | `""` |
| `envSecrets.enableEnv` | Enable loading specific keys from secrets | `false` |
| `envSecrets.envs` | List of environment variable definitions from secrets | `[]` |

#### Example with Bank Vaults Webhooks

When using Bank Vaults webhooks, you can define plain environment variables in values.yaml and add annotations. Bank Vaults will mutate the deployment after creation and inject secret values:

```yaml
podAnnotations:
  vault.security.banzaicloud.io/vault-addr: "https://vault.example.com:8200"
  vault.security.banzaicloud.io/vault-role: "my-app-role"
  vault.security.banzaicloud.io/vault-path: "kubernetes"
  vault.security.banzaicloud.io/vault-skip-verify: "false"

env:
  - name: DATABASE_URL
    value: ""  # Will be injected by Bank Vaults
  - name: API_KEY
    value: ""  # Will be injected by Bank Vaults
  - name: ENVIRONMENT
    value: production  # Plain value, not from vault
```

#### Example with Secrets

```yaml
env:
  - name: ENVIRONMENT
    value: production

envFromSecrets:
  enableEnvFrom: true
  secretName: example-external-secret

envSecrets:
  enableEnv: true
  envs:
    - name: API_KEY
      secretName: my-secret
      secretKey: API_KEY
    - name: DATABASE_URL
      secretName: db-secret
      secretKey: connection-string
```

### Job

Run a one-time Kubernetes Job.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `job.enabled` | Enable the job | `false` |
| `job.name` | Job name | `my-job` |
| `job.image` | Container image | `busybox` |
| `job.imageTag` | Container image tag | `latest` |
| `job.imagePullPolicy` | Image pull policy | `IfNotPresent` |
| `job.restartPolicy` | Restart policy | `OnFailure` |
| `job.backoffLimit` | Number of retries before marking as failed | `2` |
| `job.command` | Container command | `["/bin/sh", "-c"]` |
| `job.args` | Container arguments | `[]` |
| `job.env` | Plain environment variables | `[]` |
| `job.envFromSecrets` | Environment variables from secrets | `[]` |
| `job.resources` | Resource requests and limits | `{}` |

#### Example

```yaml
job:
  enabled: true
  name: migration-job
  image: postgres:14
  imageTag: latest
  imagePullPolicy: IfNotPresent
  restartPolicy: OnFailure
  backoffLimit: 3
  command: ["/bin/bash", "-c"]
  args: ["psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f /migrations/init.sql"]
  env:
    - name: MIGRATION_ENV
      value: production
  envFromSecrets:
    - name: DB_HOST
      secretName: db-secret
      secretKey: host
    - name: DB_USER
      secretName: db-secret
      secretKey: user
    - name: DB_NAME
      secretName: db-secret
      secretKey: database
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 256Mi
```

### CronJob

Run scheduled Kubernetes CronJobs.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cronjob.enabled` | Enable the cronjob | `false` |
| `cronjob.name` | CronJob name | `my-cronjob` |
| `cronjob.image` | Container image | `busybox` |
| `cronjob.imageTag` | Container image tag | `latest` |
| `cronjob.imagePullPolicy` | Image pull policy | `IfNotPresent` |
| `cronjob.schedule` | Cron schedule expression | `"*/5 * * * *"` |
| `cronjob.restartPolicy` | Restart policy | `OnFailure` |
| `cronjob.backoffLimit` | Number of retries | `1` |
| `cronjob.successfulJobsHistoryLimit` | Number of successful jobs to keep | `2` |
| `cronjob.failedJobsHistoryLimit` | Number of failed jobs to keep | `1` |
| `cronjob.command` | Container command | `["/bin/sh", "-c"]` |
| `cronjob.args` | Container arguments | `[]` |
| `cronjob.env` | Plain environment variables | `[]` |
| `cronjob.envFromSecrets` | Environment variables from secrets | `[]` |
| `cronjob.resources` | Resource requests and limits | `{}` |

#### Example

```yaml
cronjob:
  enabled: true
  name: backup-cronjob
  image: postgres:14
  imageTag: latest
  imagePullPolicy: IfNotPresent
  schedule: "0 2 * * *"  # Daily at 2 AM
  restartPolicy: OnFailure
  backoffLimit: 2
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 2
  command: ["/bin/bash", "-c"]
  args: ["pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME > /backups/backup-$(date +%Y%m%d).sql"]
  env:
    - name: BACKUP_ENV
      value: production
  envFromSecrets:
    - name: DB_HOST
      secretName: db-secret
      secretKey: host
    - name: DB_USER
      secretName: db-secret
      secretKey: user
    - name: DB_NAME
      secretName: db-secret
      secretKey: database
  resources:
    requests:
      cpu: 200m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 512Mi
```

### Extra Manifests

Inject raw Kubernetes manifests for advanced use cases (e.g., ExternalSecrets, custom CRDs).

| Parameter | Description | Default |
|-----------|-------------|---------|
| `extraManifests` | List of Kubernetes manifest objects | `{}` |

#### Example

```yaml
extraManifests:
  - apiVersion: external-secrets.io/v1beta1
    kind: ExternalSecret
    metadata:
      name: example-external-secret
      namespace: default
    spec:
      refreshInterval: 10s
      secretStoreRef:
        name: vault-cluster-backend
        kind: ClusterSecretStore
      target:
        name: example-external-secret
        template:
          type: Opaque
      data:
        - secretKey: FOO
          remoteRef:
            key: path/keys
            property: FOO
        - secretKey: BAR
          remoteRef:
            key: path/keys
            property: BAR
```

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

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
