# CI/CD Pipeline for Flask Application

This guide provides step-by-step instructions for setting up CI/CD pipelines using **Jenkins** and **GitHub Actions** to automate the testing and deployment of a simple **Flask** web application.

---

## 1️⃣ Jenkins CI/CD Pipeline for Flask App

### 1.1 Prerequisites
- AWS EC2 instance (Ubuntu 22.04 LTS recommended)
- Jenkins installed and configured
- Python and Flask installed
- GitHub repository for Flask app

### 1.2 Launch and Configure EC2 Instance
1. Log in to your **AWS Management Console**.
2. Navigate to **EC2 Dashboard** and click **Launch Instance**.
3. Select **Ubuntu 22.04 LTS** AMI.
4. Choose **t2.medium** (recommended for Jenkins).
5. Configure **Security Group**:
   - **SSH (22)** - Your IP
   - **HTTP (80)** - Anywhere
   - **Custom TCP Rule (8080)** - Anywhere (for Jenkins UI)
6. Download and keep the **Key Pair** safe.
7. Click **Launch**.

### 1.3 Connect to EC2 Instance
```sh
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

### 1.4 Install Jenkins
```sh
sudo apt update && sudo apt upgrade -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
echo "deb http://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
sudo apt install -y openjdk-11-jdk jenkins
```

### 1.5 Start and Enable Jenkins
```sh
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### 1.6 Access Jenkins
1. Get the Jenkins initial admin password:
   ```sh
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
2. Open a browser and go to: `http://your-ec2-public-ip:8080`
3. Enter the admin password and follow the setup wizard.
4. Install **Suggested Plugins** and create an admin user.

### 1.7 Install Dependencies
```sh
sudo apt install -y git python3 python3-pip
```

### 1.8 Clone the Flask Repository
```sh
git clone https://github.com/your-username/flask-app.git
cd flask-app
```

### 1.9 Create Jenkinsfile
In your repository, add a **Jenkinsfile** with the following content:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Test') {
            steps {
                sh 'pytest'
            }
        }
        stage('Deploy') {
            steps {
                sh 'python app.py &'
            }
        }
    }
}
```

### 1.10 Configure Jenkins Pipeline
1. Go to **Jenkins Dashboard** → Click **New Item**.
2. Select **Pipeline** and name it **Flask-CI/CD**.
3. In **Pipeline Definition**, select **Pipeline Script from SCM**.
4. Choose **Git**, enter your repo URL, and credentials.
5. Save the pipeline.

### 1.11 Trigger Builds
Every push to the GitHub repo will trigger the Jenkins pipeline.

![image](https://github.com/user-attachments/assets/29b0f2ca-b67b-448d-8ef1-173df509bd75)


---

## 2️⃣ GitHub Actions CI/CD Pipeline for Flask App

### 2.1 Prerequisites
- GitHub repository for Flask app
- GitHub Actions enabled
- Python environment setup

### 2.2 Fork and Clone Repository
```sh
git clone https://github.com/your-username/flask-app.git
cd flask-app
```

### 2.3 Create GitHub Actions Workflow
1. In your repository, create `.github/workflows/cicd.yml`.
2. Add the following YAML content:
```yaml
name: CI/CD Pipeline for Flask App
on:
  push:
    branches:
      - main
      - staging
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v3
        with:
          python-version: '3.10'
      - name: Install Dependencies
        run: pip install -r requirements.txt
      - name: Run Tests
        run: pytest
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EC2
        run: |
          ssh -i "${{ secrets.EC2_KEY }}" ubuntu@${{ secrets.EC2_IP }} "cd flask-app && git pull && python app.py &"
```

### 2.4 Add GitHub Secrets
1. Go to **GitHub Repository** → **Settings** → **Secrets** → **Actions**.
2. Add the following secrets:
   - **EC2\_KEY** (your `.pem` private key content)
   - **EC2\_IP** (your EC2 instance IP address)

### 2.5 Push Code to Trigger Workflow
```sh
git add .
git commit -m "Setup GitHub Actions pipeline"
git push origin main
```
- This will trigger GitHub Actions and deploy your Flask app to EC2.

---

## ✅ Summary
- **Jenkins** is set up on EC2 for automated CI/CD.
- **GitHub Actions** enables CI/CD workflows for testing and deployment.
- **Secrets** securely store credentials.
- **Flask app** is deployed on EC2.

Your CI/CD pipelines are now fully automated! 🚀
