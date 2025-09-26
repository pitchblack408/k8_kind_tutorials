# Lesson: Helm Basics on a `kind` Cluster
**Author:** Michael Martin  
**Date:** 2025-09-26  
**Version:** 1.0


## Learning Objectives
By the end of this lesson, learners will:
- Understand what Helm is and why it’s useful.
- Install Helm locally.
- Add and use Helm repositories.
- Deploy and manage applications with Helm charts.
- Explore the relationship between Helm and Kubernetes manifests.

---

## **1. What is Helm?**
- **Concepts:**
  - Helm is the "package manager" for Kubernetes.
  - Packages are called **Charts**.
  - Charts contain YAML templates + default values.
  - Benefits:
    - Reuse templates instead of writing raw YAML.
    - Manage upgrades/rollbacks.
    - Share applications via repositories.

---

## **2. Installing Helm**
- **Hands-on:**

    ```bash
    # Linux (latest stable)
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

    # Verify
    helm version
    ```

- **Exercise:** Run `helm help` and explore subcommands.

---

## **3. Setting Up a `kind` Cluster**
If you don’t already have one running:

```bash
kind create cluster --name helm-demo
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

---

## **4. Adding a Helm Repository**

Helm repositories are like apt or yum repos.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```

Exercise: Search for mysql in the repo.

---

## **5. Installing an Application with Helm**

Example: NGINX from Bitnami repo.

```bash
helm install my-nginx bitnami/nginx
```

Check resources created:

```bash
kubectl get all
```

Access service:

```bash
kubectl get svc my-nginx
```

If using NodePort, curl via:

```bash
kubectl port-forward svc/my-nginx 8080:80
curl http://localhost:8080
```

Exercise: Uninstall it:

```bash
helm uninstall my-nginx
```

---

## **6. Inspecting and Customizing Charts**

Show manifests before applying:

```bash
helm template my-nginx bitnami/nginx
```

Override values with --set:

```bash
helm install my-nginx bitnami/nginx --set service.type=NodePort
```

Or with a custom values.yaml:

```bash
# values.yaml
service:
  type: NodePort
  nodePorts:
    http: 30080
```

```bash
helm install my-nginx bitnami/nginx -f values.yaml
```

Exercise: Change the NodePort to 30081 and upgrade:

```bash
helm upgrade my-nginx bitnami/nginx -f values.yaml
```

---

## **7. Rollbacks and History**

View release history:

```bash
helm history my-nginx
```

Roll back to the previous version:

```bash
helm rollback my-nginx 1
```

---

## **8. Creating Your Own Chart**

Scaffold a new chart:

```bash
helm create mychart
```

Explore folder structure:

```bash
values.yaml – default config.
```


templates/ – YAML manifests with Go templating.

Install your chart:

```bash
helm install demo ./mychart
kubectl get all
```

Exercise: Modify the deployment.yaml template to use httpd instead of nginx and upgrade.

---

## **9. Final Project**

Deploy a simple 2-tier app using Helm:

Backend: MySQL (from Bitnami chart).

Frontend: Your custom chart (using helm create).

Expose via NodePort.

Test by connecting frontend to backend.

---

## **Resources**
* [Helm Documentation](https://helm.sh/docs/)
* [Bitnami Helm Charts](https://bitnami.com/stacks/helm)
* [Artifact Hub](https://artifacthub.io/)
