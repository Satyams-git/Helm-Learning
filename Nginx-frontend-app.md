# Helm + Nginx Frontend App Deployment (Kind Cluster) — Step-by-step Guide

## Objective

This hands-on exercise demonstrates how to deploy a simple Nginx-based static HTML frontend application using a Helm chart on a Kind Kubernetes cluster. After completing this guide, students will understand:

* The structure and purpose of a Helm chart
* How Deployment and Service YAML templates work
* How to access the application using port-forwarding and test it in a browser

---

## Prerequisites

1. A running Kubernetes cluster (Kind, Minikube, or kubeadm)
2. `kubectl` and `helm` installed
3. Docker image available: `satyamsri/nginx-app`
4. Working internet connection

---

## Step 1 — Create a Namespace

A dedicated namespace is created to keep all Helm resources isolated.

Command:

```bash
kubectl create namespace nginx-app-ns
```

Verify:

```bash
kubectl get ns
```

You should now see `nginx-app-ns` in the namespace list.

---

## Step 2 — Create a Helm Chart

Helm is the package manager for Kubernetes. We begin by generating a chart structure.

Command:

```bash
helm create nginx-app
```

This creates the following directory structure:

```
nginx-app/
  ├── Chart.yaml
  ├── values.yaml
  ├── templates/
  └── charts/
```

### Key Components

* **Chart.yaml** — metadata about the chart
* **values.yaml** — configuration values used by templates
* **templates/** — Kubernetes YAML templates to be rendered and deployed

---

## Step 3 — Remove Default Templates

The default Helm chart contains sample manifests (Nginx deployment). Since we are deploying our custom Nginx app, we will remove them.

Commands:

```bash
cd nginx-app/templates
rm deployment.yaml service.yaml ingress.yaml hpa.yaml tests/test-connection.yaml
```

The `templates/` directory is now ready for custom manifests.

---

## Step 4 — Create a Custom Deployment Template

This Deployment defines how our Nginx container will run in the cluster.

Create file: `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
  namespace: {{ .Release.Namespace }}
  labels:
    app: nginx-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 50m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
```

### Notes

* `replicas` determines the number of pods
* `containerPort` is set to 80 for Nginx
* Resource requests/limits help the scheduler allocate resources efficiently

---

## Step 5 — Create a Service Template

The Service exposes the application inside the cluster. We map Service port 81 to container port 80.

Create file: `templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: {{ .Release.Namespace }}
spec:
  selector:
    app: nginx-app
  ports:
    - name: http
      port: 81
      targetPort: 80
      protocol: TCP
  type: ClusterIP
```

### Notes

* `ClusterIP` means service is accessible only inside the cluster
* We will use port-forwarding for external browser access

---

## Step 6 — Customize `values.yaml`

This file defines the default values used by templates.

Replace `values.yaml` content with:

```yaml
replicaCount: 1

image:
  repository: satyamsri/nginx-app
  tag: latest
  pullPolicy: IfNotPresent
```

---

## Step 7 — Update Chart Metadata

Edit `Chart.yaml` to provide meaningful metadata.

```yaml
apiVersion: v2
name: nginx-app
description: Static Nginx Frontend App deployed using Helm
type: application
version: 0.1.0
appVersion: "1.0"
```

---

## Step 8 — Install the Helm Chart

This command renders the templates and deploys the resources in the cluster.

Command:

```bash
helm install nginx-release ./nginx-app -n nginx-app-ns
```

Verify:

```bash
kubectl get all -n nginx-app-ns
```

Expected output includes:

```
pod/nginx-release-xxxx          Running
service/nginx-svc               ClusterIP   10.x.x.x   <none>   81/TCP
deployment.apps/nginx-release   1/1         Running
```

---

## Step 9 — Access the App in Browser (Port-Forwarding)

Because Kind does not support external LoadBalancer IPs, we use port-forwarding.

Command:

```bash
sudo -E kubectl port-forward svc/nginx-svc -n nginx-app-ns 81:81 --address=0.0.0.0
```

Open in browser:

```
http://localhost:81
```

You should see the HTML page served by the Nginx container.

---

## Step 10 — Verify Logs and Pod Status

Commands:

```bash
kubectl get pods -n nginx-app-ns
kubectl logs -n nginx-app-ns <pod-name>
```

Nginx access logs will appear when you refresh the page.

---

## Step 11 — Cleanup After the Lab

Commands:

```bash
helm uninstall nginx-release -n nginx-app-ns
kubectl delete ns nginx-app-ns
```

This removes the release and deletes all associated resources.

---

## Learning Summary

* Helm charts organize Kubernetes manifests into reusable packages
* Custom Deployment and Service templates allow fine-grained control
* `values.yaml` centralizes configuration
* Port-forwarding is the simplest way to access apps in local clusters like Kind

