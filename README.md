# u-helm-chart

# Helm Chart Values Documentation

## Deployment

### `replicaCount`
Количество реплик приложения.

### `image`, `imageTag`, `imagePullPolicy`
Настройки контейнерного образа.

### `imagePullSecrets`
Список секретов для доступа к приватным реестрам:
- name: regcred

### `nameOverride`, `fullnameOverride`
Переопределения имён ресурсов.

---

## Service Account

### `serviceAccount.create`
Создавать ли сервис-аккаунт.

### `serviceAccount.automount`
Автоматически монтировать токен сервис-аккаунта.

### `serviceAccount.annotations`, `serviceAccount.name`
Дополнительные аннотации и имя аккаунта.

---

## Pod Metadata & Security

### `podAnnotations`, `podLabels`
Дополнительные аннотации и метки.

### `podSecurityContext`, `securityContext`
Контексты безопасности для пода и контейнера.

---

## Сервис

### `service.name`, `service.type`, `service.port`, `service.protocol`
Параметры Kubernetes-сервиса.

---

## Ingress

### `ingress.enabled`
Включить ли Ingress.

### `ingress.className`, `ingress.annotations`
Настройки контроллера и аннотации.

### `ingress.hosts`
Список хостов и путей:
- host: example.com
  - path: /

### `ingress.tls`
Настройки TLS-секретов.

---

## Extra Ingress (`extra.ingress`)
Дополнительный Ingress с аналогичной структурой.

---

## Probes

### `livenessProbe`, `readinessProbe`, `startupProbe`
Проверки состояния приложения (`httpGet`, `exec`, `tcpSocket`).

---

## Автомасштабирование

### `autoscaling.enabled`
Включить HPA.

### `autoscaling.minReplicas`, `maxReplicas`
Диапазон количества реплик.

### `autoscaling.targetCPUUtilizationPercentage`
Целевое использование CPU (%).

---

## Ресурсы

### `resources.requests`, `resources.limits`
Запросы и лимиты CPU/памяти.

---

## ConfigMaps

### `configMaps`
Описание монтируемых конфигов:
- name: my-config
  mountPath: /etc/my-config
  readOnly: true

---

## Volumes

### `volumes`
Дополнительные тома (ConfigMap, Secret, PVC и др).

### `volumeMounts`
Пути монтирования в контейнере:
- name: my-volume
  mountPath: /etc/secrets

---

## Планирование подов

### `nodeSelector`, `tolerations`, `affinity`
Node-привязка, толерантности и антиаффинность.

---

## Sidecar-контейнеры

### `extraContainers.enabled`
Включить sidecar'ы.

### `extraContainers.containers`
- name: sidecar
  image: busybox
  command: ["sleep", "3600"]

---

## Секреты в окружении

### `envFromSecrets`
Загрузка переменных окружения из Secret:
- name: API_KEY
  secretName: my-secret
  secretKey: API_KEY

---

## Дополнительные манифесты

### `extraManifests`
Сырые Kubernetes-манифесты (например, ExternalSecret).

---

## Job

### `job.enabled`
Включить одноразовую задачу Job.

### Остальные поля:
- `image`, `command`, `args`, `resources`, `envFromSecrets`

---

## CronJob

### `cronjob.enabled`
Включить CronJob.

### `cronjob.schedule`
Cron-расписание (например, `"*/5 * * * *"`).

### Остальные поля:
- `resources`, `envFromSecrets`, `args`, `backoffLimit`