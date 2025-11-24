# HELM — DETAILED THEORY

---

## 1. What is Helm — Simple Meaning

Helm is the **package manager for Kubernetes**.
Just like we use `apt`, `yum`, or `brew` to install software in Linux or macOS, Helm is used to install applications inside Kubernetes.

Helm combines multiple Kubernetes YAML files (Deployment, Service, ConfigMap, Secret, PVC, etc.) into one reusable packaged unit called a **Helm Chart**.

In simple words:
**Helm makes installation, upgrades, and rollbacks of Kubernetes applications easy.**

Example:
Manually you may need 10 YAML files to deploy an app.
With Helm:

```bash
helm install wordpress bitnami/wordpress
```

---

## 2. Why Helm is Needed — The Problem

Deploying applications in Kubernetes is complex due to:

* Multiple YAML files
* Environment-specific configurations
* Manual mistakes
* Difficult upgrades
* Manual rollback

Helm solves these problems by:

* Packaging everything as a chart
* Reusing the chart across dev, test, prod
* One-command install/upgrade/delete
* Easy rollback


---

## 3. What is a Helm Chart — Structure and Parts

A Helm Chart is a folder containing all YAML templates and configuration for an application.

Example structure:

```
myapp/
├── Chart.yaml        → metadata of the chart
├── values.yaml       → default configuration values
└── templates/        → Kubernetes YAML templates
```

### Explanation

* **Chart.yaml** → chart information (name, version, appVersion, description)
* **values.yaml** → configuration (image, ports, replicas)
* **templates/** → actual Kubernetes YAMLs written with variables like `{{ .Values.xyz }}`

---

## 4. How Helm Works — Working Principle

Helm works like a **templating engine**.

Process:

1. User runs `helm install`
2. Helm reads `values.yaml`
3. It replaces placeholders (`{{ }}`) inside templates
4. Generates final YAML
5. Applies them to the cluster (similar to `kubectl apply`)

---

## 5. Main Components of a Helm Chart

| Component   | Description                                        |
| ----------- | -------------------------------------------------- |
| Chart.yaml  | Metadata of the chart (name, version, description) |
| values.yaml | Configurable values used inside templates          |
| templates/  | Actual Kubernetes resource templates               |
| charts/     | Dependency charts (optional)                       |
| README.md   | Documentation (optional)                           |

**Important:**
`Chart.yaml` and `values.yaml` are mandatory in every chart.

---

## 6. What is a Helm Repository

Helm repositories store charts — similar to GitHub storing code.

Popular repositories:

* Bitnami: [https://charts.bitnami.com/bitnami](https://charts.bitnami.com/bitnami)
* Jenkins: [https://charts.jenkins.io](https://charts.jenkins.io)
* Prometheus: [https://prometheus-community.github.io/helm-charts](https://prometheus-community.github.io/helm-charts)

Commands:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo nginx
```


---

## 7. What is a Helm Release

A **release** is a deployed instance of a chart inside the cluster.

Example:

```bash
helm install nginx-demo bitnami/nginx
helm install nginx-prod bitnami/nginx
```

Both use the same chart but create two different releases.

---

## 8. Helm Basic Commands — Trainer Summary

| Command        | Purpose                        |
| -------------- | ------------------------------ |
| helm version   | Check Helm version             |
| helm create    | Create a new chart             |
| helm install   | Install an application         |
| helm upgrade   | Update an existing release     |
| helm list      | List active releases           |
| helm uninstall | Delete a release               |
| helm rollback  | Roll back to previous revision |


---

## 9. Relation Between values.yaml and templates

Templates use variables like:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

These values come from `values.yaml`:

```yaml
image:
  repository: nginx
  tag: latest
```

---

## 10. Helm Features — Classroom Explanation Points

1. **Easy Installation and Upgrade** — complete setup with one command
2. **Reusability** — same chart works in dev/stage/prod
3. **Rollback** — revert to the previous version easily
4. **Dependency Management** — charts can depend on other charts
5. **Centralized Configuration** — everything in `values.yaml`
6. **Version Control** — every chart has a version
7. **Easy Cleanup** — `helm uninstall` removes everything

---

## 11. Helm Workflow — Step-by-step

Good to draw on the classroom whiteboard.

Sequence:

1. Developer creates a chart
2. Writes templates and values
3. Publishes chart to a repository
4. DevOps engineer installs the chart
5. Helm merges templates + values
6. Generates real YAML
7. Applies them to the cluster
8. Creates a release
9. Upgrades, rollbacks, uninstall all managed by Helm
**

---

## 12. Helm vs kubectl — Simple Comparison

| Task          | kubectl             | Helm                  |
| ------------- | ------------------- | --------------------- |
| Deployment    | Multiple YAML files | One install command   |
| Update        | Manual edits        | `helm upgrade`        |
| Rollback      | Manual revert       | `helm rollback`       |
| Reuse         | Hard                | Easy with values.yaml |
| Configuration | Spread out          | Centralized           |


---

## 13. Real-world Use Cases of Helm

* Deploying Jenkins, Prometheus, Grafana, WordPress, Nginx
* Automating deployments in CI/CD pipelines
* GitOps tools like ArgoCD and Flux integrate deeply with Helm
* Production clusters use Helm for versioned deployments and rollbacks

---

## 14. Helm Advantages — Summary Table

| Feature                 | Explanation                               |
| ----------------------- | ----------------------------------------- |
| Simplified Deployment   | Complete setup with one command           |
| Reusable Charts         | Same chart works in multiple environments |
| Easy Upgrade & Rollback | Versioned releases                        |
| Centralized Config      | All configuration in one file             |
| Standardized            | Fixed structure for all charts            |


---

## 15. Helm in One Line

“**Helm is like apt or yum — but for Kubernetes.**”
It simplifies installation, upgrades, and rollbacks, making Kubernetes app management fast and consistent.
