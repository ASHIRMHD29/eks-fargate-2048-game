# eks-fargate-2048-game
Kubernetes project deploying the 2048 game on Amazon EKS Fargate with ALB Ingress, TLS via ACM, and Route 53 DNS using custom domain ashirpkp.shop
======================================================================================================================================================

# 🎮 2048 Game Deployment on Amazon EKS with Fargate, ALB Ingress & TLS

This project demonstrates how to deploy the popular [2048 game](https://github.com/gabrielecirulli/2048) using **Amazon EKS with Fargate**, **Ingress ALB Controller**, **Route 53**, and **ACM** for HTTPS access via a custom domain.

---

## 🌐 Live URL

🔗 [https://ashirpkp.shop](https://ashirpkp.shop)

---

## 📌 Project Goals

- Deploy containerized web app to **AWS EKS with Fargate** (serverless compute)
- Configure **Kubernetes Ingress** with **AWS ALB Ingress Controller**
- Secure the app using **TLS certificate from AWS ACM**
- Route traffic from a **GoDaddy-registered domain via Route 53**
- Perform **DNS validation** for ACM using Route 53

---

## 🛠️ Tech Stack

| Component     | Technology                          |
|---------------|--------------------------------------|
| Container     | Docker (public ECR image)           |
| Platform      | Amazon EKS with Fargate             |
| Ingress       | AWS ALB Ingress Controller          |
| DNS           | Amazon Route 53 + GoDaddy           |
| TLS           | AWS Certificate Manager (ACM)       |
| Domain        | [ashirpkp.shop](https://ashirpkp.shop) |
| CI/CD         | Manual deployment using kubectl     |

---

## 🧾 Kubernetes Resources

All YAML files are in the `k8s/` folder:

- `deployment.yaml` – Deploys the 2048 app
- `service.yaml` – Exposes the app internally
- `ingress.yaml` – Defines external access with TLS

---

## 📸 Screenshots

| Component                 | Screenshot |
|--------------------------|------------|
| ✅ EKS pod (Fargate) running | ![Pod](screenshots/pod.png) |
| ✅ ACM certificate issued     | ![ACM](screenshots/acm.png) |
| ✅ ALB with HTTPS listener    | ![ALB](screenshots/alb.png) |
| ✅ Route 53 alias record      | ![Route53](screenshots/route53.png) |
| ✅ GoDaddy nameserver update | ![GoDaddy](screenshots/godaddy.png) |
| ✅ Game running in browser   | ![Game](screenshots/game.png) |

---

## 🏗️ EKS Setup on Your Local Machine (Fargate + ALB Ingress)

### 🔧 Prerequisites

Install the following tools:

```bash
# macOS
brew install awscli kubectl eksctl helm

# Ubuntu/Debian
sudo apt install awscli
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
curl -s --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin/
```

### 1️⃣ Configure AWS CLI

```bash
aws configure
```

### 2️⃣ Create EKS Cluster with Fargate Profile

```bash
eksctl create cluster   --name game-cluster   --region ap-south-1   --fargate   --version 1.29   --without-nodegroup
```

### 3️⃣ Confirm Cluster Access

```bash
kubectl get svc
kubectl get nodes
```

### 4️⃣ Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider   --region ap-south-1   --cluster game-cluster   --approve
```

### 5️⃣ Install AWS Load Balancer Controller

```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy   --policy-name AWSLoadBalancerControllerIAMPolicy   --policy-document file://iam-policy.json

eksctl create iamserviceaccount   --cluster game-cluster   --namespace kube-system   --name aws-load-balancer-controller   --attach-policy-arn arn:aws:iam::<YOUR_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy   --approve   --region ap-south-1

helm install aws-load-balancer-controller eks/aws-load-balancer-controller   -n kube-system   --set clusterName=game-cluster   --set serviceAccount.create=false   --set region=ap-south-1   --set vpcId=<YOUR_VPC_ID>   --set serviceAccount.name=aws-load-balancer-controller
```

### 6️⃣ Deploy App & Ingress

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
```

### 7️⃣ Create TLS Cert in ACM & Validate via Route 53

Request cert in ACM, validate using CNAME in Route 53.

---

## 📂 Folder Structure

```
.
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── screenshots/
│   ├── pod.png
│   ├── acm.png
│   ├── alb.png
│   ├── route53.png
│   ├── godaddy.png
│   └── game.png
├── README.md
```

---

## 🙋‍♂️ Author

**Ashir PKP**  
DevOps Enthusiast | AWS | Linux | Kubernetes

---

## 📬 Feedback

Open an issue for bugs or improvements. Contributions welcome!
