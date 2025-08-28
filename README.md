[![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=Jenkins&logoColor=white)](https://www.jenkins.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![SonarQube](https://img.shields.io/badge/SonarQube-4E9BCD?style=for-the-badge&logo=sonarqube&logoColor=white)](https://www.sonarqube.org/)
[![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)](https://react.dev/)

# 🚀 Zomato Clone DevOps Project

Deploy a **ReactJS Zomato Clone** app on a Docker container via Jenkins, with code quality checks using SonarQube and vulnerability scanning using Trivy. 

### [Click here for the Zomato Clone App Github Repository](https://github.com/your-username/zomato-clone-devops)
---

## 🖥️ 1. Launch EC2 Instance

> **Tip:**  
> To save costs while learning, use a **Spot Instance**. For production, consider an On-Demand instance.

### Steps

1. Go to **EC2 Dashboard**.
2. Select **Spot Requests** from the left menu.
3. Click **Create Spot Fleet Request**.
4. Choose **Amazon Linux 2023 kernel-6.1 AMI**.
5. Select your VPC, availability zone, security group, and keypair.
6. Set **Target Capacity** to `1`.
7. Set **Max cost** for spot instance (e.g., `0.03` for `t3.large`).
8. In **Instance Type requirements**, select **Manually Select Instance types**.
9. Remove all auto-generated instances.
10. Add `t3.large` as the instance type.
11. Set **Allocation Strategy** to **Capacity Optimized**.
12. Click **Launch**.

---

### ⚡️ Alternative: Use Spot Instance Config File

- Download `spot_instance_request.json`.
- Fill in placeholders: Account ID, KeyName, SubnetID, Security Group ID.
- Open **AWS CloudShell** (`>_` icon).
- Upload the JSON config file.
- Run:
  ```bash
  aws ec2 request-spot-fleet --spot-fleet-request-config file://spot_instance_request.json
  ```

---

## 📦 2. Installing Packages

- Copy contents of `packages-installation.sh`.
- In terminal:
  ```bash
  nano packages-installation.sh
  ```
- Paste, save (`Ctrl+O`), and exit (`Ctrl+X`).
- SSH into EC2 again after script execution.

---

## 🛠️ 3. Setup Jenkins

1. Ensure **port 8080** is open in your security group.
2. Open `http://<instance-ip>:8080` in your browser.
3. Retrieve admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
4. Paste password in Jenkins setup.
5. Install **Suggested Plugins**.
6. Create admin user.

### 🔌 Install Required Plugins

- Eclipse Temurin Installer
- SonarQube Scanner
- NodeJS Plugin
- Docker
- Docker Commons
- Docker Pipeline
- Docker API
- docker-build-step

### 🧰 Configure Tools

- **JDK Installation**
  - Name: `jdk24`
  - Install from adoptium.net (`jdk-24.0.2+12`)
- **SonarQube Scanner**
  - Name: `sonar-scanner`
  - Install from Maven Central
- **NodeJS Installation**
  - Name: `NodeJs16`
  - Install from nodejs.org (`NodeJs 16.2.0`)

---

## 🔍 4. Run SonarQube & Configure Jenkins Credentials

1. Start SonarQube:
   ```bash
   docker run -d --name sonar -p 9000:9000 sonarqube:lts
   ```
2. Open `http://<instance-ip>:9000`.
3. Login: `admin` / `admin`.
4. Change password.
5. Generate token in **Administration > Security > Users**.
6. Add token to Jenkins:
   - **Manage Jenkins > Credentials > System > Global credentials**
   - Add **Secret Text** with ID: `Sonar-Token`.

---

## ✅ 5. Add Quality Gate in SonarQube

- Go to **Administration > Configuration > Webhooks**
- Create webhook:
  - URL: `http://<instance-ip>:8080/sonarqube-webhook/`

---

## 🐳 6. Configure ECR for Docker Images

1. Go to **AWS Console > ECR**.
2. Create repository: `project/zomato`.

---

## 📝 7. Setup Jenkins Pipeline

- Open the `Jenkinsfile` from the repo.
- Edit these variables:
  - `AWS_REGION`
  - `ECR_REGISTRY`
  - `ECR_REPOSITORY`

---

## 🔐 8. Create IAM Role for EC2

1. Go to **AWS Console > IAM > Roles**.
2. Create Role:
   - Entity Type: AWS service
   - Use Case: EC2
   - Policy: `AmazonEC2ContainerRegistryFullAccess`
3. Attach role to EC2 instance.

---

## ⚙️ 9. Create Jenkins Job

1. Go to **Jenkins Dashboard**.
2. Click **New Item**.
3. Enter job name, select **Pipeline**.
4. Paste pipeline script.
5. Save.
6. Click **Build with Parameters**.
7. Select deployment environment.
8. Click **Build**.

---

## 🌐 10. Access the Website

- Open: `http://<instance-ip>:3000` in your browser.

---

## 📚 Summary

This project demonstrates a complete DevOps workflow:
- **Infrastructure provisioning** (EC2 Spot Instance)
- **Automated package installation**
- **CI/CD pipeline with Jenkins**
- **Code quality checks (SonarQube)**
- **Security scanning (Trivy)**
- **Containerization (Docker)**
- **Artifact storage (ECR)**
- **Automated deployment**

---

