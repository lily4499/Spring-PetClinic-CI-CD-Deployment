
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
## file-setup.py

```python
import os

# Define base directory
base_dir = "/home/lilia/VIDEOS/spring-petclinic-infra"

# File paths and contents
files = {
    "terraform/ec2/main.tf": """
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "devops" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  key_name      = "your-keypair-name"

  tags = {
    Name = "DevOps-Jump-Host"
  }

  vpc_security_group_ids = ["sg-xxxxxxxx"]
}
""",
    "terraform/eks/main.tf": """
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "petclinic-eks"
  cluster_version = "1.29"
  subnets         = module.vpc.public_subnets
  vpc_id          = module.vpc.vpc_id

  manage_aws_auth = true
  node_groups = {
    default = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1

      instance_types = ["t3.medium"]
    }
  }
}
""",
    "terraform/eks/variables.tf": """
variable "region" {
  default = "us-east-1"
}
""",
    "terraform/eks/outputs.tf": """
output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}
""",
    "terraform/eks/provider.tf": """
provider "aws" {
  region = "us-east-1"
}
""",
    "scripts/setup-devops.sh": """
#!/bin/bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt install openjdk-17-jdk openjdk-17-jre -y
java --version

sudo apt update -y
sudo apt install maven -y
mvn -version

sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=\\"$(dpkg --print-architecture)\\" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \\
  \\"$(. /etc/os-release && echo \\"$VERSION_CODENAME\\")\\" stable" | \\
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
docker --version

apt install python3.10-venv -y
apt install python3-pip -y

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
sudo chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
kubectl version --client

wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform -y

wget https://github.com/digitalocean/doctl/releases/download/v1.94.0/doctl-1.94.0-linux-amd64.tar.gz
tar xf doctl-1.94.0-linux-amd64.tar.gz
sudo mv doctl /usr/local/bin
""",
    "docker/Dockerfile": """
FROM eclipse-temurin:17-jdk-jammy

WORKDIR /app
COPY target/spring-petclinic-3.1.0-SNAPSHOT.jar /app
EXPOSE 8080
CMD ["java", "-jar", "spring-petclinic-3.1.0-SNAPSHOT.jar"]
""",
    "k8s/deployment.yaml": """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: petclinic
spec:
  replicas: 1
  selector:
    matchLabels:
      app: petclinic
  template:
    metadata:
      labels:
        app: petclinic
    spec:
      containers:
        - name: petclinic
          image: laly9999/spring-petclinic:v1
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: petclinic-service
spec:
  type: LoadBalancer
  selector:
    app: petclinic
  ports:
    - port: 80
      targetPort: 8080
"""
}

# Create files
for relative_path, content in files.items():
    file_path = os.path.join(base_dir, relative_path)
    os.makedirs(os.path.dirname(file_path), exist_ok=True)
    with open(file_path, "w") as f:
        f.write(content.strip())

import ace_tools as tools; tools.display_dataframe_to_user(name="Project File Structure", dataframe=None)
"✅ All files have been created in /home/lilia/VIDEOS/spring-petclinic-infra/"

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
