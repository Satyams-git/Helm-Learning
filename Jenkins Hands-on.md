# Helm + Jenkins Hands-on Lab (Kind Cluster) — Step-by-step Guide

## Objective

This lab teaches students how to:

1. Install Helm
2. Deploy Jenkins using the official Jenkins Helm chart
3. Access Jenkins through port-forwarding
4. Retrieve the initial Jenkins admin password

---

## Step 1 — Create a Working Directory for Helm Installation

It is good practice to keep installer scripts organized in a dedicated folder.

Commands:

```bash
mkdir ~/helm
cd ~/helm
```

---

## Step 2 — Download the Helm Install Script

We download the official Helm installation script from GitHub.

Commands:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
ls -l
```

This script will be used to install Helm.

---

## Step 3 — Make the Script Executable and Install Helm

Grant execute permission and run the installer. It will install the Helm binary in `/usr/local/bin/helm`.

Commands:

```bash
chmod 700 get_helm.sh
./get_helm.sh
```

Verify installation:

```bash
helm version
```

A valid version output confirms that Helm is installed successfully.

---

## Step 4 — Add and Update the Jenkins Helm Repository

Helm charts are stored in repositories. We add the Jenkins chart repository.

Commands:

```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
helm repo list
```

You should now see the Jenkins Helm repository in the list.

---

## Step 5 — Deploy Jenkins Using Helm

The following command installs Jenkins into a dedicated namespace. The `--create-namespace` flag automatically creates the namespace.

Command:

```bash
helm install jenkins-demo jenkins/jenkins --namespace jenkins-ns --create-namespace
```

This creates:

* Deployment
* StatefulSet
* Service
* PVC (Persistent Volume Claim)
* Secrets
* Additional Jenkins resources

---

## Step 6 — Check Deployment Status

It may take a few minutes for Jenkins to start because of initialization procedures and PVC binding.

Commands:

```bash
kubectl get ns
kubectl get pods -n jenkins-ns
kubectl get svc -n jenkins-ns
kubectl get pvc -n jenkins-ns
```

If the pod is stuck in `CrashLoopBackOff` or `Init` state:

```bash
kubectl logs -n jenkins-ns deploy/jenkins
kubectl logs -n jenkins-ns
```

These logs help identify issues during startup.

---

## Step 7 — Access Jenkins in a Browser (Port-forwarding)

Local clusters like Kind do not provide an external LoadBalancer. Port-forwarding is the simplest method to expose Jenkins.

Command:

```bash
kubectl port-forward svc/jenkins-demo 8080:8080 -n jenkins-ns --address=0.0.0.0
```

Open the browser:

```
http://localhost:8080
```

Stop port-forwarding:

```bash
pkill -f "kubectl port-forward"
```

---

## Step 8 — Retrieve the Jenkins Initial Admin Password

The Jenkins Helm chart creates a Kubernetes Secret containing the first-time admin password.

List secrets:

```bash
kubectl get secret -n jenkins-ns
```

Retrieve and decode the admin password:

```bash
kubectl get secret -n jenkins-ns -o jsonpath="{.items[0].data.jenkins-admin-password}" | base64 --decode ; echo
```

If the secret name is known (example: `jenkins`):

```bash
kubectl get secret jenkins -n jenkins-ns -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode ; echo
```

**Username:** `admin`

Use this username and password to log in.

---

## Step 9 — Jenkins First-time Setup

After logging in:

* Jenkins will show the setup wizard
* Install the recommended plugins
* Create admin settings as required

Once setup completes, Jenkins is ready for use.

---

## Step 10 — Cleanup After the Lab

To remove all Jenkins resources:

Commands:

```bash
helm uninstall jenkins-demo -n jenkins-ns
kubectl delete ns jenkins-ns
```

This deletes the release and the entire namespace.

---

## Summary

* Helm simplifies the installation and lifecycle management of applications like Jenkins.
* Jenkins deployment includes several Kubernetes components such as Deployments, Services, PVCs, and Secrets.
* Port-forwarding allows browser access in local clusters.
* The initial admin password is stored securely in Kubernetes Secrets.
