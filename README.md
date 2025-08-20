# WebApp Deployment on Kubernetes (AWS with kOps)

This project demonstrates how to set up a production-grade Kubernetes cluster using `kOps` on AWS and deploy a sample web application with persistent storage.

---

## 📌 Table of Contents

- [Prerequisites](#prerequisites)
- [1. Setup AWS and kOps](#1-setup-aws-and-kops)
- [2. Create Kubernetes Cluster using kOps](#2-create-kubernetes-cluster-using-kops)
- [3. Validate Cluster](#3-validate-cluster)
- [4. Deploy the Web Application](#4-deploy-the-web-application)
- [5. Verify the Deployment](#5-verify-the-deployment)
- [6. Clean Up](#6-clean-up)
- [Project Structure](#project-structure)

---

## 🛠 Prerequisites

- AWS CLI (configured)
- kOps
- kubectl
- A registered domain name (can use Route 53)
- Amazon Linux or Ubuntu server (tested on Amazon Linux 2)
- S3 bucket to store cluster state

---

## 1. Setup AWS and kOps

### Create and configure S3 bucket

```bash
aws s3api create-bucket --bucket <your-kops-state-store> --region <region>
aws s3api put-bucket-versioning --bucket <your-kops-state-store> --versioning-configuration Status=Enabled
export KOPS_STATE_STORE=s3://<your-kops-state-store>
```

### IAM and DNS

- Create a Route 53 hosted zone (public) with a domain like `yourdomain.com`.
- Update IAM user with admin access or appropriate `kOps` permissions.

---

## 2. Create Kubernetes Cluster using kOps

```bash
export NAME=cluster.yourdomain.com
kops create cluster \
  --name=$NAME \
  --state=$KOPS_STATE_STORE \
  --zones=eu-west-2a,eu-west-2b \
  --node-count=3 \
  --node-size=t3.medium \
  --master-size=t3.medium \
  --dns-zone=yourdomain.com \
  --cloud=aws \
  --yes
```

---

## 3. Validate Cluster

Wait a few minutes and run:

```bash
kops validate cluster --state=$KOPS_STATE_STORE
```

Ensure the cluster is healthy and all nodes are ready.

---

## 4. Deploy the Web Application

### Apply StorageClass and PersistentVolumeClaim

```bash
kubectl apply -f storage-aws.yaml
```

```yaml
# storage-aws.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: cloud-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
spec:
  storageClassName: cloud-ssd
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 7Gi
```

### Apply workloads and Services

```bash
kubectl apply -f workloads.yaml
kubectl apply -f services.yaml
```

Go to the cloud-setup folder to get files.

## 5. Verify the Workloads

```bash
kubectl get pods
kubectl get svc
```

You should see the external `LoadBalancer` address. Open it in your browser to access the web app.

---

## 6. Clean Up

To delete the Kubernetes cluster:

```bash
kops delete cluster --name=$NAME --state=$KOPS_STATE_STORE --yes
```

Also, clean up the S3 bucket, Route 53 records, and any created volumes (EBS) manually to avoid costs.

---

## 📂 Project Structure

```
├── workloads.yaml
├── services.yaml
├── storage-aws.yaml
├── mongo-stack.yaml
├── README.md
```

---

## 📌 Author

Md Ashraf Mahmud Anjan  
DevOps Enthusiast | Kubernetes Practitioner  
GitHub: [GitHub](https://github.com/Anjan854)

---

## 📜 License

This project is licensed under the MIT License.
