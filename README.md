# MEAN Stack CRUD App Setup & Deployment Guide

This document provides step-by-step instructions for running the application locally and deploying it to an AWS EC2 instance using Docker and GitHub Actions.

## 🛠 Local Setup (Using Docker)

The easiest way to run the application locally is using Docker Compose, which spins up the MongoDB database, Node.js backend, Angular frontend, and Nginx reverse proxy simultaneously.

### Prerequisites
- Docker & Docker Compose installed
- Git installed

### Steps:
1. **Clone the repository:**
   ```bash
   git clone https://github.com/StackSamuraiK/mean-assignment.git
   cd mean-assignment
   ```

2. **Start the application:**
   ```bash
   docker compose up -d --build
   ```

3. **Access the Application:**
   - **Frontend:** `http://localhost:4200`
   - **Backend API:** `http://localhost:8080/api/tutorials`
   - **Nginx Proxy:** `http://localhost` (Routes `/` to frontend and `/api` to backend)

4. **Stop the application:**
   ```bash
   docker compose down
   ```

---

## 🚀 Production Deployment (AWS EC2 + GitHub Actions)

This project includes a fully automated CI/CD pipeline. Whenever you push code to the `master` or `main` branch, GitHub Actions builds new Docker images, pushes them to Docker Hub, and automatically updates the running containers on your AWS EC2 instance.

### Phase 1: AWS EC2 Virtual Machine Setup
1. Log into your AWS Console and launch an **Ubuntu-based EC2 instance** (e.g., `t2.micro`).
2. Download the `.pem` key pair file and keep it secure.
3. Configure your EC2 **Security Group** to allow inbound traffic on:
   - Port `80` (HTTP)
   - Port `443` (HTTPS)
   - Port `22` (SSH)

4. Connect to your EC2 instance from your terminal:
   ```bash
   chmod 400 your-key.pem
   ssh -i "your-key.pem" ubuntu@<YOUR_EC2_PUBLIC_IP>
   ```

5. Install Docker and Docker Compose (V2) on the instance:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install docker.io docker-compose-v2 -y
   
   # Start and enable Docker
   sudo systemctl start docker
   sudo systemctl enable docker
   
   # Allow ubuntu user to run docker without sudo
   sudo usermod -aG docker $USER
   ```
   *(After running `usermod`, type `exit` and reconnect via SSH to apply permissions).*

6. Clone your repository into the VM:
   ```bash
   git clone https://github.com/StackSamuraiK/mean-assignment.git ~/mean-assignment
   ```

### Phase 2: GitHub Actions Configuration (Secrets)
For the GitHub pipeline to work, you must add specific secrets to your repository. 

1. Go to your GitHub Repository -> **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**.
2. Add the following 5 secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username.
   - `DOCKER_PASSWORD`: Your Docker Hub Access Token (generate this in Docker Hub -> Security).
   - `VM_HOST`: The Public IPv4 address of your AWS EC2 instance.
   - `VM_USERNAME`: `ubuntu` (the default user for AWS Ubuntu AMIs).
   - `VM_SSH_KEY`: The entire contents of your extracted `.pem` file (including the `BEGIN` and `END` blocks).

### Phase 3: Automated CI/CD Execution
1. Make a code change locally.
2. Commit and push your changes to formatting:
   ```bash
   git add .
   git commit -m "Testing deployment"
   git push origin master
   ```
3. Open your GitHub Repository -> **Actions** tab.
4. Watch the `CI/CD Pipeline` run. It will:
   - Build frontend and backend images.
   - Push images to your Docker Hub.
   - SSH into your EC2 instance.
   - Run `docker compose pull` and `docker compose up -d` to restart with the fresh code.

5. Open your browser and navigate to `http://<YOUR_EC2_PUBLIC_IP>` to see the updated application live!

---

## 📸 Proofs & Screenshots

For visual confirmation of the deployment process, working CI/CD pipelines, and the live application, please refer to the attached document containing all screenshots and proofs:

[**View Deployment Proofs Document Here**](https://docs.google.com/document/d/1AC79PO5ml67d-KX6SYhm6FDO0Pt0h_M-WtxzqiTdH6E/edit?usp=sharing)
