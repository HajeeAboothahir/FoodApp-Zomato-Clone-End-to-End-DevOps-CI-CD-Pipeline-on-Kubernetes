# 🚀 FoodApp Zomato Clone – End-to-End DevOps CI/CD Pipeline on Kubernetes

<div align="center">

![DevOps](https://img.shields.io/badge/DevOps-Kubernetes-blue)
![Jenkins](https://img.shields.io/badge/Jenkins-CI/CD-red)
![Docker](https://img.shields.io/badge/Docker-Containerization-blue)
![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange)
![Grafana](https://img.shields.io/badge/Grafana-Visualization-yellow)

### ⚡ Production-Style DevOps Project

### CI/CD | Docker | Kubernetes | Monitoring

</div>

---

# 📌 Project Overview

This project demonstrates the deployment of a **React-based Zomato Clone Application** using a complete DevOps workflow on AWS.

The application is automatically built, containerized, pushed to Docker Hub, and deployed to a Kubernetes cluster through a Jenkins CI/CD pipeline.

Additionally, cluster monitoring is implemented using Prometheus and Grafana.

---

# 🏗 Architecture

```text
Developer
    │
    ▼
GitHub Repository
    │
    ▼
Jenkins Pipeline
    │
 ┌──┴─────────────┐
 │ Build Docker  │
 │ Push DockerHub│
 └──────┬────────┘
        ▼
Docker Hub
        │
        ▼
Kubernetes Cluster
(Control Plane + 2 Worker Nodes)
        │
        ▼
NodePort Service
        │
        ▼
Browser Access

Monitoring Stack
─────────────────────────
Prometheus
     │
     ▼
Grafana
```

---

# ☁ AWS Infrastructure

| Component        | Details                                     |
| ---------------- | ------------------------------------------- |
| Cloud Provider   | AWS                                         |
| EC2 Instances    | 3                                           |
| OS               | Ubuntu 24.04                                |
| Kubernetes Setup | kubeadm                                     |
| Master Node      | Jenkins + Docker + Kubernetes Control Plane |
| Worker Node 1    | Kubernetes Worker                           |
| Worker Node 2    | Kubernetes Worker                           |
| Monitoring       | Prometheus + Grafana                        |

---

# 🛠 Tools & Technologies

### Version Control

* Git
* GitHub

### CI/CD

* Jenkins

### Containerization

* Docker

### Orchestration

* Kubernetes (kubeadm)

### Monitoring

* Prometheus
* Grafana

### Cloud

* AWS EC2

### Application

* ReactJS (Zomato Clone)

---

# 📂 Project Structure

```text
FoodAppZomato/
│
├── deployment.yml
├── service.yml
├── Dockerfile
├── Jenkinsfile
├── package.json
├── public/
├── src/
└── README.md
```

---

# 🐳 Docker Build

### Build Image

```bash
docker build -t myzomatoimage .
```

### Run Container

```bash
docker run -d -p 3000:3000 myzomatoimage
```

### Push to DockerHub

```bash
docker tag myzomatoimage hajee1994/myzomatoimage:latest

docker push hajee1994/myzomatoimage:latest
```

---

# ☸ Kubernetes Deployment

## deployment.yml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: zomato-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: zomato-app

  template:
    metadata:
      labels:
        app: zomato-app

    spec:
      containers:
      - name: zomato-app
        image: hajee1994/myzomatoimage:latest

        ports:
        - containerPort: 3000
```

---

## service.yml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: zomato-service

spec:
  selector:
    app: zomato-app

  type: NodePort

  ports:
  - port: 80
    targetPort: 3000
    nodePort: 30008
```

---

# 🚀 Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/HajeeAboothahir/FoodAppZomato.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myzomatoimage .'
            }
        }

        stage('Push Docker Image') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'Docker-credential',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker tag myzomatoimage hajee1994/myzomatoimage:latest

                    docker push hajee1994/myzomatoimage:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {

                sh 'kubectl apply -f deployment.yml'

                sh 'kubectl apply -f service.yml'

                sh 'kubectl rollout restart deployment zomato-deployment'
            }
        }

        stage('Verify Deployment') {
            steps {

                sh 'kubectl get pods'

                sh 'kubectl get svc'
            }
        }
    }
}
```

---

# 📊 Monitoring Setup

### Install Prometheus & Grafana

```bash
helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts

helm repo update

helm install monitoring \
prometheus-community/kube-prometheus-stack \
-n monitoring \
--create-namespace
```

---

### Access Grafana

```bash
kubectl edit svc monitoring-grafana -n monitoring
```

Change:

```yaml
type: ClusterIP
```

To:

```yaml
type: NodePort
```

Check Port:

```bash
kubectl get svc -n monitoring
```

Example:

```text
monitoring-grafana
80:30124/TCP
```

Access:

```text
http://WorkerNodePublicIP:30124
```

---

# 🌐 Application Access

```text
http://WorkerNodePublicIP:30008
```

Example:

```text
http://13.xx.xx.xx:30008
```

---

# 🔥 Major Challenges Faced

## 1. kubeadm init Failed

### Error

```text
containerd.sock not found
```

### Fix

```bash
sudo systemctl start containerd

sudo systemctl enable containerd
```

---

## 2. Worker Node Join Failed

### Error

```text
user is not running as root
```

### Fix

```bash
sudo kubeadm join ...
```

---

## 3. IP Forwarding Disabled

### Error

```text
/proc/sys/net/ipv4/ip_forward
```

### Fix

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf

sudo sysctl -p
```

---

## 4. OOMKilled Pods

### Error

```text
OOMKilled
```

### Root Cause

Node memory exhaustion.

### Verification

```bash
kubectl get pods

kubectl describe pod POD_NAME
```

---

## 5. Service Not Reachable

### Error

```text
Endpoints: <none>
```

### Root Cause

Label mismatch between Deployment and Service.

### Verification

```bash
kubectl get pods --show-labels

kubectl get endpoints
```

### Fix

```yaml
app: zomato-app
```

Used consistently.

---

## 6. Immutable Selector Error

### Error

```text
field is immutable
```

### Fix

```bash
kubectl delete deployment zomato-deployment

kubectl apply -f deployment.yml
```

---

## 7. Jenkins Kubernetes Authentication Failed

### Error

```text
Authentication required
```

### Fix

```bash
mkdir -p /var/lib/jenkins/.kube

cp /etc/kubernetes/admin.conf \
/var/lib/jenkins/.kube/config

chown -R jenkins:jenkins \
/var/lib/jenkins/.kube
```

---

## 8. NodePort Already Allocated

### Error

```text
provided port is already allocated
```

### Fix

Change:

```yaml
30007
```

to

```yaml
30008
```

---

# 🎯 Key DevOps Skills Demonstrated

✅ AWS Infrastructure Provisioning

✅ Docker Containerization

✅ DockerHub Registry Management

✅ Jenkins CI/CD Pipeline

✅ Kubernetes Cluster Administration

✅ Deployment Management

✅ Service Exposure

✅ Troubleshooting Production Issues

✅ Prometheus Monitoring

✅ Grafana Dashboarding

✅ Linux Administration

---

# 📸 Project Outcome

Successfully built a complete production-style DevOps pipeline that:

* Pulls source code from GitHub
* Builds Docker image automatically
* Pushes image to Docker Hub
* Deploys application into Kubernetes
* Exposes application via NodePort
* Monitors cluster using Prometheus
* Visualizes metrics through Grafana

---

# 👨‍💻 Author

**Hajee Syed Aboothahir**

DevOps Engineer | AWS | Docker | Kubernetes | Jenkins | Linux | Prometheus | Grafana

GitHub: `https://github.com/HajeeAboothahir`

---

### ⭐ If you found this project useful, don't forget to Star the repository! ⭐

**Alhamdulillah — End-to-End Kubernetes DevOps Project Successfully Completed 🚀**
