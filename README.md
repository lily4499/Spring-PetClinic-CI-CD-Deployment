
# 🚀 Spring PetClinic CI/CD Deployment – EC2 + EKS with Terraform


![image](https://github.com/user-attachments/assets/1de65d4e-ff11-465b-9494-a35d4e9bec6e)

---

## 🌍 Real-World Scenario

You’re a DevOps Engineer in a growing SaaS company. Your goal is to automate deployment of the **Spring PetClinic** Java app using:

- An **EC2 Linux server** (DevOps jump host)  
- An **EKS Kubernetes cluster** (for running the app)  
- Terraform, Docker, kubectl, and Maven

---

## 📌 Why Create a Linux Server?

| Reason | Description |
|--------|-------------|
| Tool Isolation | Avoid polluting local dev environments |
| Jump Host | Secure access to AWS EKS APIs |
| CI/CD Agent | Acts as a centralized automation server |
| Real-World Practice | Simulates what real DevOps teams use in staging/production |

---

## 📁 Project Structure

```bash
spring-petclinic-infra/
├── terraform/
│   ├── ec2/
│   │   └── main.tf
│   ├── eks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── provider.tf
├── scripts/
│   └── setup-devops.sh
├── docker/
│   └── Dockerfile
├── k8s/
│   └── deployment.yaml
├── README.md
```

---

## ⚙️ Step-by-Step Instructions

### 1️⃣ Provision EC2 Jump Host (Ubuntu 22.04)

```bash
cd terraform/ec2
terraform init
terraform apply -auto-approve
```

- SSH into the EC2 instance using your key pair
- Switch to your new user and run the setup script

```bash
sudo adduser devopsuser
sudo usermod -aG sudo devopsuser
su - devopsuser
chmod +x setup-devops.sh
./setup-devops.sh
```

---

### 2️⃣ Setup DevOps Tools on EC2

The `setup-devops.sh` script installs:

- Java 17  
- Maven  
- Docker + Docker Compose  
- Python venv + pip  
- kubectl  
- Terraform  
- doctl (for DigitalOcean if needed)

---

### 3️⃣ Provision EKS Cluster Using Terraform

```bash
cd terraform/eks
terraform init
terraform apply -auto-approve

aws eks --region us-east-1 update-kubeconfig --name petclinic-eks
```

---

### 4️⃣ Clone & Build Spring PetClinic App

```bash
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic
mvn clean install -DskipTests
```

---

### 5️⃣ Create Docker Image

```bash
docker build -t laly9999/spring-petclinic:v1 .
docker login
docker push laly9999/spring-petclinic:v1
```

---

### 6️⃣ Deploy to Kubernetes (EKS)

```bash
kubectl apply -f k8s/deployment.yaml
kubectl get svc
```

- Copy the LoadBalancer `EXTERNAL-IP`
- Visit `http://EXTERNAL-IP` in your browser

---

## ✅ Final Outcome

| Step | Outcome |
|------|---------|
| EC2 Provisioned | Jump host ready for CI/CD |
| DevOps Tools | Installed with shell script |
| EKS Cluster | Deployed with Terraform |
| App | Built and dockerized |
| Deployment | Running on Kubernetes with public access |


---

## 🧑‍💻 Author

**Liliane Konissi**  
DevOps Engineer | CI/CD | Cloud Native | Terraform | Kubernetes  
[GitHub Profile](https://github.com/lily4499)

```

---

Would you like a Python script to generate all the files referenced here automatically?
