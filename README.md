# Talos Homelab - Kubernetes on Proxmox

This repository hosts the **GitOps** configuration for my personal Kubernetes cluster. The cluster is built on **Talos Linux** running on Proxmox VMs, and it uses **ArgoCD** to automatically sync the cluster state with this repository.

## 🏗 Architecture

*   **OS:** [Talos Linux](https://www.talos.dev/) (Immutable, API-driven Kubernetes OS).
*   **GitOps:** [ArgoCD](https://argo-cd.readthedocs.io/) (App-of-Apps pattern).
*   **Ingress Controller:** [ingress-nginx](https://kubernetes.github.io/ingress-nginx/).
*   **Load Balancer:** [MetalLB](https://metallbuniverse.tf/) (Layer 2 mode, advertising IPs `10.0.0.251-253`).

## 📂 Repository Structure

The repository relies on the "App of Apps" pattern to keep things organized.

*   **`bootstrap.yaml`**: The **Root** Application. This is the only file applied manually. It tells ArgoCD to watch the `gitops/` directory.
*   **`gitops/apps.yaml`**: The **Manager**. This file lists every application installed in the cluster (Infrastructure + User Apps).
*   **`infrastructure/`**: System-level configurations.
    *   `metallb/`: Load Balancer config.
    *   `ingress-nginx/`: Ingress controller setup.
    *   `argocd/`: ArgoCD self-management.
*   **`apps/`**: User-facing applications.
    *   `hello-world/`: Simple Nginx demo.
    *   `demo-app/`: Podinfo demo.
*   **`*.yaml` (Root)**: Talos machine configurations (`controlplane.yaml`, `worker.yaml`).

## 🚀 Bootstrapping the Cluster

If the cluster is wiped, follow these steps to restore the entire state:

1.  **Provision Nodes:** Use `talosctl` to apply `controlplane.yaml` and `worker.yaml` to your VMs.
2.  **Bootstrap GitOps:** Run the following command **once**:
    ```bash
    kubectl apply -f bootstrap.yaml
    ```
3.  **Wait:** ArgoCD will initialize, read `gitops/apps.yaml`, and automatically install MetalLB, Nginx, and all apps.

## 🌐 Network Access

Since this runs on bare metal (Proxmox), we use MetalLB to assign "External IPs" from the home network LAN.

*   **Ingress IP:** `10.0.0.251`
*   **Domains:** `*.local.gd` (or your custom domain).

### Local Access (DNS)
To access services, update your local computer's `/etc/hosts` file (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):

```text
10.0.0.251  argo.local.gd
10.0.0.251  hello.local.gd
```

## ➕ How to Add a New App

1.  **Create Manifests:** Create a new folder in `apps/` (e.g., `apps/plex`) and add your standard Kubernetes YAML files (Deployment, Service, Ingress).
2.  **Register App:** Edit `gitops/apps.yaml` and append a new `Application` block pointing to your new folder.
3.  **Push:** Commit and push to GitHub.
    ```bash
    git add .
    git commit -m "Add Plex"
    git push
    ```
4.  **Done:** ArgoCD will detect the change and deploy the app automatically.

## 🛠 Troubleshooting

*   **ArgoCD UI:** Access at `https://argo.local.gd`.
*   **Force Sync:** If GitHub takes too long to trigger Argo, click "Refresh" in the ArgoCD UI.
*   **Check Apps:** `kubectl get applications -n argocd`
