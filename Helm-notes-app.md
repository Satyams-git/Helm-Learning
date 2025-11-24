# Helm + Notes App Deployment (Kind Cluster) — Step-by-step Guide

## Objective

Students will learn:

1. How to deploy a custom PHP + Apache application using Helm.
2. Helm chart structure and how templates and values work.
3. How to expose the service in a Kind cluster and test the app from a browser.

---

## Prerequisites

* A Kubernetes cluster (Kind, Minikube, or kubeadm). This guide assumes you are using Kind.
* `kubectl` installed and configured for your cluster.
* `helm` (Helm 3) installed.
* Internet access to pull the Docker image.
* Docker image available: `satyamsri/notes-app`.

---

## Step 1 — Create a namespace

**Why:** Deploying into a dedicated namespace keeps application resources isolated from other workloads.

Command:

```bash
kubectl create namespace notes-ns
```

Verify:

```bash
kubectl get ns
```

You should see `notes-ns` in the list.

---

## Step 2 — Create a Helm chart scaffold

**Why:** A Helm chart is a package that contains Kubernetes manifests (templates) and default configuration (`values.yaml`).

Command:

```bash
helm create notes-app
```

This creates the following structure:

```
notes-app/
  ├── Chart.yaml
  ├── values.yaml
  ├── templates/
  └── charts/
```

---

## Step 3 — Remove default sample templates

**Why:** The Helm scaffold includes example manifests (for nginx) which we will replace with our custom manifests for the PHP + Apache app.

Commands:

```bash
cd notes-app/templates
rm deployment.yaml service.yaml ingress.yaml hpa.yaml tests/test-connection.yaml
```

This leaves the `templates/` directory empty so you can add your custom YAML files.

---

## Step 4 — Create a custom Deployment template

**Why:** This Deployment runs the PHP + Apache container and mounts a directory `/data` for note storage (emptyDir in this example).

Create file: `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
  labels:
    app: notes-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
        volumeMounts:
        - name: notes-data
          mountPath: /data
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 250m
            memory: 256Mi
      volumes:
      - name: notes-data
        emptyDir: {}
```

**Notes:**

* `{{ .Release.Name }}` and `{{ .Release.Namespace }}` are dynamic values injected by Helm at install/upgrade time.
* `replicaCount` and `image.*` come from `values.yaml`.
* `emptyDir` provides ephemeral storage inside the pod. For persistent storage across pod restarts, replace with a `PersistentVolumeClaim`.

---

## Step 5 — Create a Service template

**Why:** The Service exposes the app inside the cluster. We will expose port `82` on the Service which forwards to container port `80`.

Create file: `templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: notes-svc
  namespace: {{ .Release.Namespace }}
spec:
  selector:
    app: notes-app
  ports:
    - name: http
      port: 82
      targetPort: 80
      protocol: TCP
  type: ClusterIP
```

**Notes:**

* Type `ClusterIP` is sufficient for cluster-internal access. In Kind, `LoadBalancer` will not create a cloud LB; use port-forwarding for local access.

---

## Step 6 — Customize `values.yaml`

**Why:** `values.yaml` contains the default configuration for the chart: image details, replica count, resource settings, etc. Users can override these at install time or edit this file.

Open `values.yaml` and replace its content with:

```yaml
replicaCount: 1

image:
  repository: satyamsri/notes-app
  tag: latest
  pullPolicy: IfNotPresent
```

You can later override `image.tag` or `replicaCount` with `--set` flags during `helm install` or by providing a custom values file.

---

## Step 7 — Update `Chart.yaml` (optional)

**Why:** Update chart metadata so it accurately describes the application.

Edit `Chart.yaml` to:

```yaml
apiVersion: v2
name: notes-app
description: A PHP Notes App deployed using Helm
type: application
version: 0.1.0
appVersion: "1.0"
```

---

## Step 8 — Install the chart

**Why:** Deploy the application into the `notes-ns` namespace using Helm.

Command:

```bash
helm install notes-release ./notes-app -n notes-ns
```

Verify deployment and service:

```bash
kubectl get all -n notes-ns
```

Expected output (example):

```
pod/notes-release-xxxx   Running
service/notes-svc       ClusterIP   10.x.x.x   <none>   82/TCP
deployment.apps/notes-release   1/1   Running
```

If the pods are `ImagePullBackOff`, check image name and network access.

---

## Step 9 — Port-forward the Service (browser access)

**Why:** Kind clusters typically do not provide an external LoadBalancer. Use `kubectl port-forward` to access the app from your workstation or other hosts.

Command (to bind to all interfaces):

```bash
sudo -E kubectl port-forward svc/notes-svc -n notes-ns 82:82 --address=0.0.0.0
```

Then open in the browser:

```
http://<your-node-ip>:82
```

**Notes:**

* If you run `kubectl` on your local machine and it is the same machine you use for browsing, you can omit `--address=0.0.0.0` and `sudo` and use `localhost` instead (e.g., `http://localhost:82`).
* Using `--address=0.0.0.0` makes port-forward listen on all interfaces; use with care for security reasons.

---

## Step 10 — Verify logs and that notes are saved

Commands:

```bash
kubectl get pods -n notes-ns
kubectl logs -n notes-ns <pod-name>
```

**What to look for:**

* Pod status `Running`.
* Logs showing PHP/Apache startup and any write activity when a user saves a note.
* If you used `emptyDir`, data is ephemeral. Notes will disappear if the pod is deleted or rescheduled. For persistent storage, use a PVC backed by a hostPath (for local testing) or a proper StorageClass.

---

## Step 11 — Cleanup (optional after lab)

Commands to remove the release and namespace:

```bash
helm uninstall notes-release -n notes-ns
kubectl delete ns notes-ns
```

---

## Classroom Explanation Points

1. **Helm chart concept:** A Helm chart packages templated Kubernetes manifests and default configuration so the same chart can be reused across environments.
2. **Configuration via `values.yaml`:** Changing `values.yaml` or supplying overrides at install time allows different environments (dev/stage/prod) to use the same templates.
3. **Deployment role:** The Deployment runs the PHP + Apache container and mounts `/data` to store note files.
4. **Service role:** The Service exposes application network endpoints inside the cluster and maps Service port `82` to the container port `80`.
5. **Access in local clusters:** Kind and Minikube normally do not provide cloud LoadBalancers, so using `kubectl port-forward` is the common and simple method to reach cluster services from your workstation.

---

## Troubleshooting tips

* **Pod in `CrashLoopBackOff`:** Check `kubectl logs <pod>` for errors. Verify the container command, entrypoint, environment variables, and whether required files exist.
* **`ImagePullBackOff`:** Verify `image.repository` and `image.tag`. Ensure your environment can reach Docker Hub (or the registry used) and that the image is public or you configured imagePullSecrets.
* **Port-forward not reachable from other machines:** By default port-forward binds to `localhost`. Use `--address=0.0.0.0` and run as sudo if you need to expose to other hosts — but be mindful of security.
* **Data not persistent:** `emptyDir` is ephemeral. For persistence, add a `PersistentVolumeClaim` and mount that instead of `emptyDir`.

---

## Quick Command Recap

```bash
kubectl create namespace notes-ns
helm create notes-app
# remove default templates and add custom deployment + service YAML
helm install notes-release ./notes-app -n notes-ns
kubectl get all -n notes-ns
sudo -E kubectl port-forward svc/notes-svc -n notes-ns 82:82 --address=0.0.0.0
helm uninstall notes-release -n notes-ns
kubectl delete ns notes-ns
```
---
