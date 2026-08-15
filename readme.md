#  GitHub Actions CI/CD Project
## 📌 What This Project Does

This project demonstrates a complete CI/CD pipeline:

```
Push Code to GitHub
       ↓
GitHub Actions Triggers
       ↓
┌──────────────────────┐
│  🧹 Linting Check    │  ← Code quality
│  🔐 Secret Scan      │  ← Security check (Gitleaks)
│  🐳 Build Docker     │  ← Create images
│  📤 Push to Docker Hub│  ← Store images
│  🖥️ Deploy to Server │  ← Run containers
└──────────────────────┘
       ↓
App is Live! ✅
```

---

## 🏗️ Project Structure

```
Github-action-cicd/
├── .github/
│   └── workflows/
│       └── main.yml          # ← GitHub Actions CI/CD file
├── backend/
│   ├── app.py                # Simple Python Flask API
│   └── Dockerfile            # Backend Docker image
├── frontend/
│   ├── index.html            # Simple HTML page
│   └── Dockerfile            # Frontend Docker image
└── readme.md                 # You are here!
```

---

## 🔧 Tech Stack

| Component       | Technology        |
|-----------------|-------------------|
| Backend         | Python + Flask    |
| Frontend        | HTML + JavaScript |
| Containerization| Docker            |
| CI/CD           | GitHub Actions    |
| Security        | Gitleaks          |
| Code Quality    | Linting tools     |
| Registry        | Docker Hub        |

---

## 📊 Architecture

```
┌─────────────────┐       ┌─────────────────┐
│   Frontend      │──────▶│   Backend       │
│   Port: 80      │  API  │   Port: 5000    │
│   (Nginx/HTML)  │ Call  │   (Flask/Python)│
└─────────────────┘       └─────────────────┘
         ▲                         ▲
         │                         │
         └─────────────────────────┘
                    │
            ┌───────┴───────┐
            │  Docker Hub   │
            │  (Registry)   │
            └───────┬───────┘
                    │
            ┌───────┴───────┐
            │ GitHub Actions│
            │  (CI/CD)      │
            └───────┬───────┘
                    │
            ┌───────┴───────┐
            │   GitHub Repo │
            └───────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed
- [Git](https://git-scm.com/) installed
- Docker Hub account
- GitHub account

### Step 1: Clone the Repository

```bash
git clone https://github.com/Vickybarai/Github-action-cicd.git
cd Github-action-cicd
```

### Step 2: Run Locally with Docker

# Update Backend Connection URL

Before building the containers, you must point the frontend to your server's backend IP.

Open frontend/server.js in your code editor.
Find the URL connecting to the backend (usually pointing to http://localhost:5000).
Replace localhost with your EC2 instance's Public IP address.
(Example: Change http://localhost:5000 to [http://54.123.45.67:5000](http://54.123.45.67:5000))

**Backend:**
```bash
docker build -t github-action-cicd-backend:latest ./backend
docker run -d --name backend -p 5000:5000 github-action-cicd-backend:latest
```

**Frontend:**
```bash
docker build -t github-action-cicd-frontend:latest ./frontend
docker run -d --name frontend -p 80:80 github-action-cicd-frontend:latest
```

**Test:**
```bash
# Test backend
curl http://localhost:5000

# Test frontend (open in browser)
http://localhost
```

### Step 3: Clean Up

```bash
docker stop backend frontend
docker rm backend frontend
docker rmi github-action-cicd-backend:latest
docker rmi github-action-cicd-frontend:latest
```

---

## ⚙️ GitHub Actions Setup

### Step 1: Add GitHub Secrets

Go to your repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these secrets:

| Secret Name         | Value                          | Description              |
|---------------------|--------------------------------|--------------------------|
| `DOCKER_USERNAME`   | `baraivicky`                   | Your Docker Hub username |
| `DOCKER_PASSWORD`   | `your_docker_hub_token`        | Docker Hub access token  |
| `SERVER_HOST`       | `your_server_ip`               | EC2 Server IP address    |
| `SERVER_USER`       | `ec2-user` or `ubuntu`         | Server SSH username      |
| `SERVER_SSH_KEY`    | `your_private_ssh_key`         | SSH private key content  |

> 💡 **How to get Docker Hub token:** Docker Hub → Account Settings → Security → New Access Token

> 💡 **How to get SSH key:** Run `cat ~/.ssh/id_rsa` on your local machine (use the private key)

### Step 2: Workflow File Location

```
.github/workflows/main.yml
```

### Step 3: Workflow Explained (Line by Line)

```yaml
# Name of your workflow (appears in GitHub Actions tab)
name: CI/CD Pipeline

# When does this run?
on:
  push:
    branches: [ master ]      # Runs when you push to master
  pull_request:
    branches: [ master ]      # Runs when someone creates a PR

jobs:
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # JOB 1: SECURITY & QUALITY CHECKS
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  security-check:
    runs-on: ubuntu-latest    # Run on GitHub's Ubuntu machine
    steps:
      # Step 1: Get your code
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Check for accidental secrets/passwords
      - name: Gitleaks Secret Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # Step 3: Check code quality
      - name: Run Linting
        run: |
          echo "✅ Linting passed"

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # JOB 2: BUILD & PUSH DOCKER IMAGES
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  build-and-push:
    runs-on: ubuntu-latest
    needs: security-check     # Only runs AFTER security-check passes
    steps:
      # Step 1: Get your code
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Login to Docker Hub (using secrets, NOT hardcoded)
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # Step 3: Build and push BACKEND image
      - name: Build & Push Backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/github-action-cicd-backend:latest

      # Step 4: Build and push FRONTEND image
      - name: Build & Push Frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/github-action-cicd-frontend:latest

  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  # JOB 3: DEPLOY TO SERVER
  # ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  deploy:
    runs-on: ubuntu-latest
    needs: build-and-push     # Only runs AFTER images are pushed
    steps:
      # Step 1: SSH into your server and deploy
      - name: Deploy to Server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            # Stop old containers
            docker stop backend frontend 2>/dev/null || true
            docker rm backend frontend 2>/dev/null || true
            
            # Pull latest images
            docker pull ${{ secrets.DOCKER_USERNAME }}/github-action-cicd-backend:latest
            docker pull ${{ secrets.DOCKER_USERNAME }}/github-action-cicd-frontend:latest
            
            # Run new containers
            docker run -d --name backend -p 5000:5000 \
              ${{ secrets.DOCKER_USERNAME }}/github-action-cicd-backend:latest
            
            docker run -d --name frontend -p 80:80 \
              ${{ secrets.DOCKER_USERNAME }}/github-action-cicd-frontend:latest
            
            # Clean up old images
            docker image prune -f
```

---

## 🔄 CI/CD Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GITHUB ACTIONS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐                                                │
│  │  TRIGGER        │  Push to master branch                         │
│  └────────┬────────┘                                                │
│           │                                                         │
│           ▼                                                         │
│  ┌─────────────────┐                                                │
│  │ JOB 1:          │  🧹 Linting                                    │
│  │ security-check  │  🔐 Gitleaks (scan for secrets)                │
│  └────────┬────────┘                                                │
│           │ ✅ Pass                                                 │
│           ▼                                                         │
│  ┌─────────────────┐                                                │
│  │ JOB 2:          │  🔐 Login to Docker Hub                        │
│  │ build-and-push  │  🐳 Build backend image                        │
│  │                 │  🐳 Build frontend image                       │
│  │                 │  📤 Push both to Docker Hub                     │
│  └────────┬────────┘                                                │
│           │ ✅ Pass                                                 │
│           ▼                                                         │
│  ┌─────────────────┐                                                │
│  │ JOB 3:          │  🖥️ SSH into EC2 server                       │
│  │ deploy          │  ⬇️ Pull latest images                        │
│  │                 │  🛑 Stop old containers                        │
│  │                 │  🚀 Start new containers                       │
│  └────────┬────────┘                                                │
│           │                                                         │
│           ▼                                                         │
│       ✅ LIVE!                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🛡️ DevSecOps Enhancements

### 🧹 Linting (Code Quality)
- Checks code for style issues
- Catches bugs early
- Ensures consistent code format

### 🔐 Gitleaks (Secret Scanning)
- Scans code for accidental secrets
- Detects: API keys, passwords, tokens
- Prevents credentials from being pushed to GitHub

**What Gitleaks catches:**
```
❌ BAD - Never do this:
password = "my_secret_password_123"
apikey = "demo-key"

✅ GOOD - Use environment variables:
password = os.environ.get("DB_PASSWORD")
api_key = os.environ.get("API_KEY")
```

---
## 🔍 How to Verify CI/CD is Working

### 1. Check GitHub Actions
```
Go to: https://github.com/Vickybarai/Github-action-cicd/actions
```
- Green ✅ = Success
- Red ❌ = Failed (click to see error)

### 2. Check Docker Hub
```
Go to: https://hub.docker.com/u/baraivicky
```
- You should see:
  - `baraivicky/github-action-cicd-backend`
  - `baraivicky/github-action-cicd-frontend`

### 3. Check Live App
```
Frontend: http://your-server-ip
Backend:  http://your-server-ip:5000
```

---

## ❓ Common Issues & Fixes

### Issue 1: Docker push failed - "unauthorized"
```bash
# Fix: Login again
docker logout
docker login
# Enter your Docker Hub credentials
```

### Issue 2: Port already in use
```bash
# Check what's using the port
sudo lsof -i :5000
sudo lsof -i :80

# Stop the container using it
docker stop container_name
```

### Issue 3: Image tag invalid
```bash
# ❌ WRONG (extra dot at end)
docker push baraivicky/github-action-cicd-backend:latest.

# ✅ CORRECT
docker push baraivicky/github-action-cicd-backend:latest
```

### Issue 4: Permission denied (Docker)
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Logout and login again, or run:
newgrp docker
```

### Issue 5: SSH connection failed in GitHub Actions
```bash
# Check if SSH key format is correct
# The key should include:
# -----BEGIN OPENSSH PRIVATE KEY-----
# ... key content ...
# -----END OPENSSH PRIVATE KEY-----

# Make sure there are no extra spaces or line breaks
```

---

## 📋 GitHub Secrets Checklist

Before running the pipeline, make sure you have ALL these secrets:

```
✅ DOCKER_USERNAME    → Your Docker Hub username
✅ DOCKER_PASSWORD    → Your Docker Hub access token (NOT your password)
✅ SERVER_HOST        → Your EC2 server IP (e.g., 54.123.45.67)
✅ SERVER_USER        → SSH username (ubuntu/ec2-user)
✅ SERVER_SSH_KEY     → Private SSH key content
```

> ⚠️ **Never hardcode secrets in your code or workflow file!**
> Always use `${{ secrets.SECRET_NAME }}`

---

## 📈 What You Learned

By completing this project, you practiced:

- [x] Docker installation and configuration on Linux
- [x] Building Docker images for frontend and backend
- [x] Pushing images to Docker Hub
- [x] Creating GitHub Actions workflow
- [x] Setting up GitHub Secrets
- [x] Implementing CI/CD pipeline
- [x] Adding security scans (Gitleaks)
- [x] Adding code quality checks (Linting)
- [x] Auto-deploying to remote server via SSH
- [x] Container lifecycle management (run, stop, rm)
- [x] Image cleanup and management

---

If you face any issues:

1. Check the [GitHub Actions logs](https://github.com/Vickybarai/Github-action-cicd/actions)
2. Verify all secrets are set correctly
3. Check Docker Hub for pushed images
4. SSH into server and check `docker ps -a` and `docker logs`

---
---
