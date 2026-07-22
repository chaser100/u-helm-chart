# Why This Chart?

Universal Helm Chart is a reusable universal Helm chart that deploys different Kubernetes applications through `values.yaml`.

Instead of maintaining one Helm chart per application, maintain one generic Helm chart and configure each service with values. This keeps Kubernetes deployment templates consistent across applications while still allowing each application to define its own image, service, ingress, Gateway API route, jobs, cronjobs, storage, sidecars, init containers, and custom manifests.

Use this chart when you want to:

- Deploy applications with Helm without creating a separate chart for every service
- Standardize Kubernetes application Helm chart templates across teams
- Keep application-specific configuration in `values.yaml`
- Reuse the same Helm chart in Helm, Argo CD, Flux, and other GitOps workflows
- Support both simple web services and advanced Kubernetes workloads

This chart is designed for application teams that need a practical reusable Helm chart, not a framework that hides Kubernetes. You still configure standard Kubernetes concepts directly, but you avoid duplicating the same Helm templates in every repository.
