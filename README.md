# k8s-manifests

## 📁 Project Structure

Here is an overview of the directory structure:
```
.
|-- apps
|   |-- base
|   |-- dev
|   |-- production
|   `-- staging
|-- clusters
|   |-- dev
|   |-- production
|   `-- staging
`-- infrastructure
    |-- base
    |-- dev
    |-- production
    `-- staging
```
### 📁 Directory Explanations

This repository uses a GitOps structure, leveraging Kustomize's "base and overlay" pattern.

* **`/apps`**: Contains all manifests for your deployable applications (e.g., microservices, frontends, APIs).
    * **`/apps/base`**: Holds the common, environment-agnostic YAML "templates" for all applications.
    * **`/apps/dev`**: Contains Kustomize overlays for the **development** environment. These patch the `base` manifests with `dev`-specific settings (e.g., lower replica counts, debug flags).
    * **`/apps/staging`**: Overlays for the **staging** environment (e.g., testing database URLs, UAT configurations).
    * **`/apps/production`**: Overlays for the **production** environment (e.g., high replica counts, production resource limits, stable image tags).

* **`/infrastructure`**: Contains manifests for cluster-wide, third-party services that support your applications.
    * **Examples**: NGINX Ingress Controller, Prometheus, Grafana, cert-manager.
    * **`/infrastructure/base`**: Default manifests for these infrastructure tools.
    * **`/infrastructure/[dev|staging|production]`**: Overlays to customize these tools for each specific environment (e.g., different `values.yaml` for a Helm chart).

* **`/clusters`**: This directory connects everything. It holds the Argo CD `Application` resources that define what to deploy and where.
    * **`/clusters/dev`**: Contains Argo CD manifests that point to `apps/dev` and `infrastructure/dev` and sync them to your **development cluster**.
    * **`/clusters/staging`**: Points to the `staging` overlays for deployment to the **staging cluster**.
    * **`/clusters/production`**: Points to the `production` overlays for deployment to the **production cluster**.
