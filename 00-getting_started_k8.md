# Lesson Plan: Kubernetes Basics with `kind`

Author: Michael Martin<br>
Date: 2025-09-26<br>
Version: 1.0

## Learning Objectives
By the end, learners will:
- Install and use `kind` to create Kubernetes clusters.
- Practice `kubectl` commands against a local cluster.
- Deploy and manage workloads using YAML manifests.
- Understand pods, deployments, services, and configuration resources.

---

## **Lesson 1: What is Kubernetes?**
- **Concepts:**
  - Why Kubernetes exists (container orchestration, scaling, resilience).
  - Control plane vs worker nodes.
  - Kubernetes objects (pods, services, deployments).
- **Hands-on (prep):**
  - Install:

    ```bash
    # Install kubectl (Linux example)
    curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl && sudo mv kubectl /usr/local/bin/
    
    # Install kind
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
    chmod +x ./kind && sudo mv ./kind /usr/local/bin/
    ```

- **Exercise:** Draw Kubernetes architecture and label components.

---

## **Lesson 2: Creating Your First Cluster with `kind`**
- **Concepts:**
  - `kind` runs Kubernetes inside Docker containers.
  - Cluster creation and management.
- **Hands-on:**

    ```bash
    kind create cluster --name k8s-basics
    kubectl cluster-info
    kubectl get nodes
    ```

  - Destroy cluster:

    ```bash
    kind delete cluster --name k8s-basics
    ```

- **Exercise:** Create and delete a cluster twice to practice.

---

## **Lesson 3: Pods – The Smallest Unit**
- **Concepts:**
  - What is a Pod? (one or more containers, shared network/storage).
- **Hands-on:**

    ```yaml
    # nginx-pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
    ```

    ```bash
    kubectl apply -f nginx-pod.yaml
    kubectl get pods
    kubectl describe pod nginx-pod
    kubectl delete pod nginx-pod
    ```

- **Exercise:** Create a pod with `httpd` instead of `nginx`.

---

## **Lesson 4: Deployments and Scaling**
- **Concepts:**
  - Deployments manage ReplicaSets → ReplicaSets manage Pods.
  - Rolling updates and rollbacks.
- **Hands-on:**

    ```yaml
    # nginx-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.25
            ports:
            - containerPort: 80
    ```

    ```bash
    kubectl apply -f nginx-deployment.yaml
    kubectl get deployments
    kubectl scale deployment nginx-deployment --replicas=4
    kubectl rollout history deployment nginx-deployment
    ```

- **Exercise:** Roll back to a previous version after updating the image.

---

## **Lesson 5: Services and Networking**
- **Concepts:**
  - Service types: ClusterIP (default), NodePort, LoadBalancer.
- **Hands-on:**

    ```yaml
    # nginx-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
        app: nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: NodePort
    ```

    ```bash
    kubectl apply -f nginx-service.yaml
    kubectl get svc
    ```

  - Get port:

    ```bash
    kubectl get svc nginx-service -o wide
    curl localhost:<NodePort>
    ```

- **Exercise:** Create another service for `httpd`.

---

## **Lesson 6: ConfigMaps and Secrets**
- **Concepts:**
  - Storing non-sensitive vs sensitive data.
- **Hands-on:**

    ```bash
    kubectl create configmap app-config --from-literal=APP_MODE=demo
    kubectl create secret generic app-secret --from-literal=DB_PASS=supersecret
    ```

  - Pod consuming config:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: config-demo
    spec:
      containers:
      - name: alpine
        image: alpine
        command: [ "sh", "-c", "env && sleep 3600" ]
        env:
        - name: APP_MODE
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_MODE
        - name: DB_PASS
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASS
    ```

- **Exercise:** Exec into the pod and run `env | grep APP_MODE`.

---

## **Lesson 7: Persistence with Volumes**
- **Concepts:**
  - Ephemeral vs Persistent storage.
  - PersistentVolumeClaims in kind (uses hostPath).
- **Hands-on:**

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: demo-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
    ```

  - Use in a pod:

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: pvc-demo
    spec:
      containers:
      - name: busybox
        image: busybox
        command: [ "sh", "-c", "echo Hello > /data/hello.txt && sleep 3600" ]
        volumeMounts:
        - mountPath: /data
          name: storage
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: demo-pvc
    ```

- **Exercise:** Exec into the pod and verify `/data/hello.txt`.

---

## **Lesson 8: Debugging and Observability**
- **Concepts:**
  - Common troubleshooting steps (`kubectl logs`, `describe`, `exec`).
  - Health probes.
- **Hands-on:**

    ```yaml
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    ```

  - Add probe to nginx deployment, break it (wrong port), and observe restarts.

- **Exercise:** Fix the probe and watch it recover.

---

## **Final Project**
- Deploy a **2-tier app** with:
  - Backend (Deployment + PVC + ConfigMap for settings).
  - Frontend (Deployment + Service).
  - Secret for credentials.
  - Access via NodePort.
- **Stretch:** Add an Ingress with `kind`’s ingress controller.

---

## **Resources**
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [kind Docs](https://kind.sigs.k8s.io/)
- Katacoda Kubernetes Scenarios (hands-on labs)
- Book: *Kubernetes Up & Running* by Brendan Burns et al.
- CNCF YouTube channel
