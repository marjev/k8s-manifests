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

## 🚀 Getting Started

### Requirements

Before running this project, ensure you have the following installed:

- **Docker**: Install Docker Desktop or Docker Engine
- **Minikube**: Download and install Minikube
- **kubectl**: Install the kubectl CLI tool

### Setup

1. **Start Minikube with Docker as a driver**:
   ```bash
   minikube start --driver=docker
   ```

2. **Create a namespace**:
   ```bash
   kubectl create namespace argocd
   ```

3. **Install Argo CD using manifests**:
   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

4. **Apply the manifest**:
   ```bash
   kubectl apply -f <path-to-your-manifests>
   ```

5. **Verify Argo CD installation**:
   ```bash
   kubectl get pods -n argocd
   ```
   Wait until all pods are in `Running` state.

6. **Install Argo CD CLI**:
   - **macOS**: `brew install argocd`
   - **Linux**: Download from [Argo CD releases](https://github.com/argoproj/argo-cd/releases/latest)
   - **Windows**: Download from [Argo CD releases](https://github.com/argoproj/argo-cd/releases/latest)

### Access Argo CD Server

1. **Set up port forwarding**:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```

2. **Retrieve the initial admin password**:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```

3. **Login with Argo CD CLI**:
   ```bash
   argocd login localhost:8080
   ```
   Use `admin` as the username and the password retrieved in the previous step.

## ✅ Evaluation

### Install kubeconform

Install kubeconform to validate Kubernetes manifests:

- **macOS**: `brew install kubeconform`
- **Linux**: Download from [kubeconform releases](https://github.com/yannh/kubeconform/releases/latest)
- **Windows**: Download from [kubeconform releases](https://github.com/yannh/kubeconform/releases/latest)

### Run kubeconform

Run kubeconform against the apps directory in strict mode:

```bash
kubeconform -strict apps/
```

This will validate all Kubernetes manifests in the `apps/` directory against the official Kubernetes OpenAPI schemas, ensuring they conform to Kubernetes API specifications.

### Install kube-score

Install kube-score to analyze Kubernetes manifests for best practices and security issues:

- **macOS**: `brew install kube-score`
- **Linux**: Download from [kube-score releases](https://github.com/zegl/kube-score/releases/latest)
- **Windows**: Download from [kube-score releases](https://github.com/zegl/kube-score/releases/latest)

### Run kube-score

Run kube-score against your YAML files to check for best practices, security issues, and resource optimization:

```bash
# Score all YAML files recursively using glob pattern
kube-score score apps/**/*.yaml

# Score all YAML files recursively using find
find apps/ -name "*.yaml" -o -name "*.yml" | xargs kube-score score

# Score a specific file
kube-score score apps/base/myapp/deployment.yaml

# Score multiple specific files
kube-score score apps/base/myapp/*.yaml
```

**Note:** kube-score requires file paths, not directories. Use glob patterns (`**/*.yaml`) or `find` with `xargs` to score multiple files.

kube-score will analyze your manifests and provide recommendations for:
- Security best practices (security contexts, capabilities, etc.)
- Resource management (requests, limits, probes)
- High availability (PodDisruptionBudgets, anti-affinity)
- Network policies and service configurations
