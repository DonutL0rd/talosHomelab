# Talos Homelab - GitOps Kubernetes Cluster

This repository hosts the **GitOps** configuration for a Kubernetes cluster managed by **Talos Linux**. It utilizes **ArgoCD** to automatically synchronize the cluster state with this repository, ensuring infrastructure and applications are defined as code.

## 🏗 Architecture

*   **OS:** [Talos Linux](https://www.talos.dev/) (Immutable, API-driven Kubernetes OS).
*   **GitOps:** [ArgoCD](https://argo-cd.readthedocs.io/) (App-of-Apps pattern).
*   **Ingress Controller:** [ingress-nginx](https://kubernetes.github.io/ingress-nginx/).
*   **Load Balancer:** [MetalLB](https://metallbuniverse.tf/) (Layer 2 mode).

## 📂 Repository Structure

The repository structure follows the "App of Apps" pattern for modular and scalable management.

```text
├── bootstrap.yaml       # The Root Application. Applied manually to bootstrap ArgoCD.
├── controlplane.yaml    # Talos Control Plane machine configuration.
├── worker.yaml          # Talos Worker machine configuration.
├── talosconfig          # Talos client configuration.
├── kubeconfig           # Kubernetes client configuration.
├── apps/                # User-facing applications.
│   └── demo-app/        # Example application manifest.
├── gitops/
│   └── apps.yaml        # The Manager. Lists all apps (infra + user) for ArgoCD to manage.
└── infrastructure/      # System-level configurations.
    ├── argocd/          # ArgoCD self-management and configuration.
    ├── ingress-nginx/   # Ingress controller setup.
    └── metallb/         # MetalLB Load Balancer configuration.
```

## 🚀 Bootstrapping the Cluster

To provision a new cluster from this repository:

1.  **Provision Nodes:** Use `talosctl` to apply `controlplane.yaml` and `worker.yaml` to your nodes (VMs or Bare Metal).
    ```bash
    talosctl apply-config --insecure --nodes <node-ip> --file <role>.yaml
    ```
2.  **Bootstrap GitOps:** Apply the bootstrap manifest to install the root ArgoCD application. This is the only manual `kubectl` command required.
    ```bash
    kubectl apply -f bootstrap.yaml
    ```
3.  **Automatic Provisioning:** ArgoCD will detect the `bootstrap.yaml` application, which points to `gitops/apps.yaml`. It will then automatically install all infrastructure components (MetalLB, Ingress Nginx) and defined applications.

## 🌐 Network Access

This cluster uses MetalLB to announce LoadBalancer service IPs on the local network.

*   **Ingress Controller:** Services are exposed via the Ingress Controller, which receives an IP from the MetalLB pool.
*   **DNS:** Configure your local DNS or `/etc/hosts` to point your desired domains to the Ingress LoadBalancer IP.

### Example `/etc/hosts` configuration:
```text
<load-balancer-ip>  argocd.local
<load-balancer-ip>  myapp.local
```

## ➕ How to Add a New App

1.  **Create Manifests:** Create a new directory in `apps/` containing your Kubernetes manifests (Deployment, Service, Ingress, etc.).
2.  **Register App:** Update `gitops/apps.yaml` to include a new `Application` block pointing to your new directory.
3.  **Push Changes:** Commit and push your changes to the repository. ArgoCD will automatically detect the change and deploy the application.

## 🛠 Troubleshooting

*   **ArgoCD UI:** Access the ArgoCD dashboard via the configured Ingress URL to monitor application status.
*   **Sync Status:** If automatic sync is delayed, you can manually trigger a refresh via the ArgoCD UI or CLI.