# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.6] - 2026-07-23

### Added
- Optional Prometheus Operator `ServiceMonitor` support via the new `serviceMonitor` values block
- Tested ServiceMonitor values example with configurable labels and scrape endpoints

### Changed
- Updated README and configuration documentation with ServiceMonitor requirements and usage

## [0.3.5] - 2026-07-22

### Added
- GitHub Actions workflow that lints the chart and renders every tested values example
- Dedicated chart positioning and usage guidance in the project documentation

### Changed
- Restructured and shortened README with tested examples, requirements, idempotent installation, and executable troubleshooting commands

### Fixed
- Fixed automatically created Plain Ingress Service names to use `service.name` or `backend.service.name`, avoiding invalid generated names and keeping Ingress backend references consistent

## [0.3.4] - 2026-05-29

### Added
- New `extraDeployments` values block for managing additional Deployment resources directly from chart values
- Optional inheritance for `extraDeployments[].image`, `extraDeployments[].imageTag`, and `extraDeployments[].imagePullPolicy` from main deployment values
- Optional `extraDeployments[].inheritEnv` flag to inherit app-level `env`/`envFrom` context only when explicitly enabled
- Optional `route.timeouts.request` and `route.timeouts.backendRequest` support for HTTPRoute rules
- Test scenario for `extraDeployments` rendering and imageTag inheritance/override

### Changed
- Updated README with `extraDeployments` documentation and usage examples

## [0.3.3] - 2026-05-26

### Added
- Optional `job.serviceAccountAnnotations` merged into chart-created ServiceAccount annotations when `job.enabled=true`

### Changed
- ServiceAccount template now renders when any of deployment/job/cronjob is enabled (with `serviceAccount.create=true`)
- Updated README and job test values for ServiceAccount annotations in Job migration scenarios

## [0.3.2] - 2026-05-18

### Added
- Optional container `lifecycle` hooks support for deployment main container (`postStart` and `preStop`)

### Changed
- Updated README with `lifecycle` parameter reference and usage example

## [0.3.1] - 2026-04-28

### Added
- `job.nameOverride` and `job.fullnameOverride` for stable/custom Job naming
- Optional `job.ttlSecondsAfterFinished` support
- Optional `job.podAnnotations` support for Job Pod template metadata
- Optional `job.envFrom` support for list-based `secretRef`/`configMapRef`
- Optional `job.extraLabels` support for Job and Job Pod template metadata
- Optional `job.serviceAccountName` override for Job Pod

### Changed
- Updated README job parameter reference and examples for new Job options
- Updated job test values scenario to validate new Job options rendering

## [0.3.0] - 2026-04-28

### Added
- Optional `job.annotations` support for adding custom annotations to Job metadata

### Changed
- Updated README job documentation and examples with annotations usage
- Updated job test values scenario to cover custom annotations rendering

## [0.2.9] - 2026-04-07

### Added
- Optional `securityContext` support for sidecar containers via `extraContainers.containers[].securityContext`

### Changed
- Updated README examples and sidecar parameter reference for `extraContainers` `securityContext`

## [0.2.7] - 2026-03-12

### Added
- ConfigMap annotations support

## [0.2.6] - 2026-03-12

### Added
- Support for Gateway API `HTTPRoute` via new `route` configuration block
- New template `templates/route.yaml` with backend bound to chart-managed Service (`application.fullname` + `service.port`)
- Test values scenario for Gateway API route rendering (`tests/values-test-route.yaml`)

### Changed
- Updated documentation for Gateway API route usage and values
- Route template now renders only when deployment is enabled to avoid referencing a non-existent chart-managed Service

## [0.2.5] - 2026-01-14

### Added
- Support for `shareProcessNamespace` parameter to enable process namespace sharing between containers in the pod

## [0.2.4] - 2025-12-03

### Added
- Support for PersistentVolumeClaims (PVC) via `persistentVolumeClaims` parameter
- Automatic volume and volumeMount configuration for PVCs in main container
- Individual PVC support for init containers via `pvc.enabled: true` in `initContainers.containers[].pvc`
- Individual PVC support for sidecar containers via `pvc.enabled: true` in `extraContainers.containers[].pvc`
- PVC template created before deployment to ensure proper resource ordering

### Changed
- Renumbered template files to maintain logical order (PVC template is now 4_pvc.yaml, placed before deployment)
- PVCs from `persistentVolumeClaims` are now mounted only in the main container (not automatically in init/sidecar containers)
- Init containers and sidecar containers require individual `pvc` configuration if persistent storage is needed

## [0.2.3] - 2025-11-28

### Added
- Support for init containers via `initContainers` parameter for pre-startup initialization tasks (database migrations, file downloads, etc.)

## [0.2.2] - 2025-11-19

### Added
- Support for custom container name via `containerName` parameter (defaults to Chart.Name if not set)
- Support for custom command in main container via `command` parameter

## [0.2.1] - 2025-11-13

### Added
- Support for plain environment variables (not only from secrets) via `env` parameter
- Automatic Helm hooks for `ExternalSecret` resources in `extraManifests` to ensure they are created before deployment
- Support for plain environment variables in Job, CronJob, and sidecar containers
- Comprehensive test suite with 12 test scenarios covering all features

### Changed
- Updated documentation to reflect new environment variables functionality
- Improved `extraManifests` template to automatically add pre-install/pre-upgrade hooks for ExternalSecret resources

### Fixed
- Fixed issue where deployment could fail with `ConfigContainerError` when using ExternalSecret resources

## [0.2.0] - 2025-11-13

### Added
- Support for disabling deployment via `deploymentDisable` parameter
- Plain Ingress feature with automatic service creation
- Enhanced ingress configuration options
- Support for multiple ingress resources

### Changed
- Chart version updated to 0.2.0
- Improved ingress template structure

## [0.1.9] - Previous

### Added
- Initial chart release
- Basic deployment, service, and ingress support
- Job and CronJob support
- ConfigMap and volume management
- Sidecar containers support
- Autoscaling (HPA) support
- Environment variables from secrets
- Extra manifests support

---

[0.3.6]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.6
[0.3.5]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.5
[0.3.4]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.4
[0.3.3]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.3
[0.3.2]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.2
[0.3.1]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.1
[0.3.0]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.3.0
[0.2.9]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.9
[0.2.6]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.6
[0.2.5]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.5
[0.2.4]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.4
[0.2.3]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.3
[0.2.2]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.2
[0.2.1]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.1
[0.2.0]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.2.0
[0.1.9]: https://github.com/chaser100/u-helm-chart/releases/tag/v0.1.9
