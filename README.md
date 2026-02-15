
# 🌍 3-Tier Application Deployment in AWS

---

## 🧩 What Are We Creating?

We are building a **3-Layer Cloud Application** like your production app:

```
User → Website → Server → Database
```

| Layer    | Work              | Example       |
| -------- | ----------------- | ------------- |
| Frontend | Website UI        | React App     |
| Backend  | Application Logic | Node.js API   |
| Database | Store Data        | MongoDB / RDS |

---

# 🏗️ FINAL ARCHITECTURE

![Image](https://miro.medium.com/1%2APDHya_zt_n657nYUAm1OeA.jpeg)

![Image](https://docs.aws.amazon.com/images/prescriptive-guidance/latest/patterns/images/pattern-img/ce977924-012c-4fb6-8e51-94d6e5c829a6/images/378456a3-f4d1-4a57-bb36-879c240cabfb.png)

![Image](https://d2908q01vomqb2.cloudfront.net/ca3512f4dfa95a03169c5a670a4c91a19b3077b4/2018/11/20/image1-1.png)

![Image](https://www.stacksimplify.com/course-images/aws-eks-alb-ingress-basics.png)

---

# 🖥️ STEP 1 — Create Cloud Computer (EC2)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2A4upHIZmoqdyTguqGFNiflg.png)

![Image](https://media2.dev.to/cdn-cgi/image/width%3D800%2Cheight%3D%2Cfit%3Dscale-down%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fbwnj7azufwh0n9ifusng.jpg)

![Image](https://i.sstatic.net/e1r3o.png)

![Image](https://i.sstatic.net/wCuub.png)

Go to:

👉 AWS Console → EC2 → Launch Instance

Fill Details:

| Field         | Value        |
| ------------- | ------------ |
| Instance Name | 3-tier-hq    |
| OS            | Ubuntu 22.04 |
| Instance Type | t2.medium    |
| Storage       | 20 GB        |

Create Key Pair:

```
baniprasad-key.pem
```

Download this file.

---

# 🔐 STEP 2 — Connect to EC2 Server

Open Terminal:

```bash
ssh -i "baniprasad-key.pem" ubuntu@<EC2-PUBLIC-IP>
```

Now you are inside your AWS Cloud Server ☁️

---

# 📂 STEP 3 — Download Application Code

```bash
git clone https://github.com/BaniprasadMangaraj/aws-eks-3tier-deployment.git
cd aws-eks-3tier-deployment/Application-Code
```

You will see:

```
frontend
backend
```

---

# 🐳 STEP 4 — Install Docker

```bash
sudo apt-get update
sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
```

Check:

```bash
docker ps
```

---

# 🌐 STEP 5 — Run Website (Frontend Test)

Go inside frontend:

```bash
cd frontend
nano Dockerfile
```

Paste:

```dockerfile
FROM node:14
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
CMD [ "npm", "start" ]
```

Build Website:

```bash
docker build -t threetier-frontend .
docker run -d -p 3000:3000 threetier-frontend
```

---

## 🔓 Allow Website Port

Go to:

👉 EC2 → Security Group → Inbound Rules

Add:

```
Type: Custom TCP
Port: 3000
Source: Anywhere
```

Open Browser:

```
http://<EC2-PUBLIC-IP>:3000
```

🎉 Website Running Successfully!

Stop container:

```bash
docker kill <container-id>
```

---

# ☁️ STEP 6 — Connect AWS to Server

Install AWS CLI:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

Enter:

```
Access Key
Secret Key
Region: ap-south-1
Output: json
```

⚠️ Use **ap-south-1 (Mumbai Region)**
Because your **Production RDS is already running here**.

---

# ☸️ STEP 7 — Create Kubernetes Cluster (EKS)

Install Tools:

```bash
curl -o kubectl https://amazon-eks.s3.ap-south-1.amazonaws.com/1.27.0/2023-09-15/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin
```

```bash
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```

Create Cluster:

```bash
eksctl create cluster \
--name three-tier-cluster \
--region ap-south-1 \
--node-type t3.medium \
--nodes-min 2 \
--nodes-max 2
```

⏳ Wait 15–20 minutes

Connect Cluster:

```bash
aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster
kubectl get nodes
```

---

# 💾 STEP 8 — Setup Database

![Image](https://leading-bell-3e1c02e64d.media.strapiapp.com/npa_WB_0_8ab1556808.png)

![Image](https://miro.medium.com/1%2AZBUzOwaMkbfPD3iZXxVH1g.gif)

![Image](https://www.mongodb.com/docs/kubernetes-operator/v1.33/images/multi-cluster-arch-with-service-mesh.svg)

![Image](https://www.mongodb.com/docs/kubernetes/current/images/mdb-resources-arch.svg)

Run:

```bash
cd aws-eks-3tier-deployment/Kubernetes-Manifests-file/Database
kubectl create namespace three-tier
kubectl apply -f .
```

MongoDB Ready ✅

---

# ⚙️ STEP 9 — Start Backend

```bash
cd ../Backend
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

# 🖼️ STEP 10 — Start Frontend

```bash
cd ../Frontend
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

---

# 🌍 STEP 11 — Make App Public (Load Balancer)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AIB56TluS4psX_B5UK4fPbw.png)

![Image](https://miro.medium.com/1%2Az5fKfkqatiV9qVCdUTgdpA.png)

![Image](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2019/10/06/illustration-2.png)

![Image](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2024/06/26/33.png)

Install Helm:

```bash
sudo snap install helm --classic
```

Apply Ingress:

```bash
kubectl apply -f ingress.yaml
kubectl get ing -n three-tier
```

AWS Automatically Creates:

✔ Application Load Balancer
✔ Public URL
✔ Security Group

---

# 🎉 SUCCESS!

Now your:

✅ Website
✅ Backend
✅ Database

All are running in AWS Cloud ☁️

---

# 📊 Final Flow

```
User
 ↓
AWS Load Balancer
 ↓
Frontend (React)
 ↓
Backend (Node.js)
 ↓
MongoDB
```

---

# 🧹 CLEANUP (TO STOP BILLING)

```bash
eksctl delete cluster --name three-tier-cluster --region ap-south-1
```

Also Delete:

✔ EC2 Instance
✔ Load Balancer
✔ Security Groups
✔ EBS Volume

---

### 🔹 GitHub Repository Reference

Application Source Code:

```
https://github.com/LondheShubham153/TWSThreeTierAppChallenge
```

Kubernetes Manifests and Deployment Files:

```
https://github.com/BaniprasadMangaraj/aws-eks-3tier-deployment
```

---

### 🔹 Video Tutorial Reference

Step-by-Step Deployment Walkthrough:

```
https://youtu.be/YlUa3t9Aaic?si=4IHTu-LZBNwuRnwa
```

---



Tell me 👍
