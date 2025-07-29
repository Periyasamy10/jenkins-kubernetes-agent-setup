# Jenkins Kubernetes Dynamic Agent Setup

This project helps you set up Jenkins on Kubernetes with dynamic slave agents.

---

## 1. Prerequisite

* An AWS account with an EKS cluster created and `kubectl` access configured
* AWS CLI
* `kubectl`

---

## 2. Create VPC and EKS Cluster

Use this repo to set up your VPC and EKS cluster:

👉 [terraform-aws-eks](https://github.com/Periyasamy10/terrafrom-aws-eks)

Make sure you’ve run the following after provisioning:

```bash
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster
```

---

## 3. Clone This Repo

```bash
git clone https://github.com/Periyasamy10/jenkins-kubernetes-agent-setup.git
cd jenkins-kubernetes-agent-setup/kubernetes-jenkins
```

---

## 4. Apply Jenkins Configuration Manifests

### Namespace

```bash
kubectl apply -f namespace.yaml
```

### Service Account

```bash
kubectl apply -f serviceAccount.yaml
```

### Persistent Volume (Edit Required)

Edit the `volume.yaml` file and update the node name:

```yaml
nodeAffinity:
  required:
    nodeSelectorTerms:
    - matchExpressions:
      - key: kubernetes.io/hostname
        operator: In
        values:
        - <YOUR_NODE_NAME>
```

Get the node name with:

```bash
kubectl get nodes
```

Then apply:

```bash
kubectl apply -f volume.yaml
```

### Jenkins Service

```bash
kubectl apply -f service.yaml
```

### Jenkins Deployment

```bash
kubectl apply -f deployment.yaml
```

This will bring up the Jenkins pod in the `devops-tools` namespace.

---

## 5. Access Jenkins

1. Get the LoadBalancer URL:

```bash
kubectl get svc -n devops-tools
```

2. Open Jenkins in browser:

```
http://<load-balancer-ip>:8080
```

3. Get the admin password:

```bash
kubectl logs -n devops-tools -l app=jenkins -c jenkins
```

4. Complete the setup wizard

---

## 6. Download Kubernetes Plugin

Before configuring Jenkins for dynamic agent provisioning, ensure the **Kubernetes plugin** is installed:

1. Navigate to **Manage Jenkins → Plugins**
2. Go to the **Available** tab
3. Search for **Kubernetes**
4. Install the plugin and restart Jenkins if prompted

📚 **Reference**: [Jenkins Kubernetes Plugin Documentation](https://www.jenkins.io/doc/book/installing/kubernetes/)

---

## 7. Configure Jenkins Kubernetes Plugin

Go to:

* **Manage Jenkins → Manage Nodes and Clouds → Configure Clouds → Add New Cloud → Kubernetes**

Fill in:

* **Name**: Kubernetes
* **Kubernetes URL**: `https://kubernetes.default`
* **Kubernetes Namespace**: `devops-tools`
* ✅ Check `WebSocket`
* **Jenkins URL**: `http://jenkins-service.devops-tools.svc.cluster.local:8080`

Save the configuration.

---

## 8. Create and Run Pipeline

1. Go to **Jenkins Dashboard → New Item**
2. Choose **Pipeline**, provide a name (e.g., `k8s-dynamic-agent`)
3. Select **Pipeline script from SCM**
4. Choose **Git** and use this repository URL:

   ```
   https://github.com/Periyasamy10/jenkins-kubernetes-agent-setup.git
   ```
5. Set **Branch** as `main` (or the correct branch)
6. Set **Script Path** as `Jenkinsfile`
7. Click **Save**

### Run the Pipeline

Click **Build Now** — Jenkins will create a dynamic agent pod using the configuration defined in the `Jenkinsfile`, execute the job, and automatically clean up the pod after 10 minutes of inactivity.

---

## ✅ Output

* Jenkins Master and Agent(s) on EKS
* Agent pods created and destroyed dynamically
* Gradle-based test job running in Kubernetes slave

