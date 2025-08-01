# 🚀 GitHub Actions Deployment Guide

This guide explains how to set up CI/CD deployment pipelines using GitHub Actions for two types of projects:

---

## 📦 1. Basic Project Deployment via SSH + Rsync

This setup uses GitHub Actions to **sync your code to a VPS via SSH and rsync**.

###  Assumptions
- You’ve pushed your project to GitHub.

- You have a VPS (e.g., Ubuntu) with Docker & Docker Compose installed.

- Your .env file is not committed, and it's already present on the VPS.

- The VPS has a folder like /home/deploy/fbaserver where the app will live.

### ✅ GitHub Repository Secrets

| Secret Name   | Description                              |
|---------------|------------------------------------------|
| `VPS_HOST`    | IP address or hostname of your VPS       |
| `VPS_USER`    | SSH username (e.g., `deploy`)            |
| `VPS_SSH_KEY` | Private SSH key for passwordless access  |

### Suggested Directory Structure on VPS
```plaintext
/home/deploy/fbaserver/
├── .env                 # Already created manually
├── docker-compose.yml   # Pushed by GitHub Action
├── Dockerfile           # Pushed by GitHub Action
├── entrypoint.sh        # Pushed by GitHub Action
├── ... (other source files)
```

### ✅ GitHub Workflow: `.github/workflows/deploy.yml`

```yaml
name: Deploy to VPS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H ${{ secrets.VPS_HOST }} >> ~/.ssh/known_hosts

      - name: Copy Code to VPS
        run: |
          rsync -avz --progress --delete -e "ssh -i ~/.ssh/id_ed25519" . \
          ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }}:/home/deploy/tata/test-github-action
```

---

## 🐳 2. Dockerized Django Deployment with PostgreSQL, Redis & Channels

This setup deploys a **Dockerized Django project** with:
- Redis
- PostgreSQL
- Django Channels (ASGI)
- Gunicorn + UvicornWorker

### ✅ Repository Folder on VPS

The project will be deployed into:

```
/home/deploy/fbaserver/
```

> Ensure `.env` is already present on the VPS inside this directory.

### ✅ Required GitHub Secrets

| Secret Name      | Description                            |
|------------------|----------------------------------------|
| `VPS_HOST`       | IP or hostname of your VPS             |
| `VPS_USER`       | SSH user (e.g., `deploy`)              |
| `VPS_SSH_KEY`    | Private SSH key in plain text          |

### ✅ GitHub Workflow: `.github/workflows/deploy.yml`

```yaml
name: Deploy Django App to VPS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -H ${{ secrets.VPS_HOST }} >> ~/.ssh/known_hosts

      - name: Rsync project files to VPS
        run: |
          rsync -avz --delete -e "ssh -i ~/.ssh/id_ed25519" ./ \
          ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }}:/home/deploy/fbaserver/

      - name: Deploy on VPS with Docker Compose
        run: |
          ssh -i ~/.ssh/id_ed25519 ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }} << 'EOF'
            cd /home/deploy/fbaserver
            docker compose pull
            docker compose down --remove-orphans
            docker compose up --build -d
            docker image prune -f
          EOF
```

# VPS Setup (One-Time)

##  1. Install Docker & Docker Compose:
```bash
sudo apt update && sudo apt install docker.io docker-compose -y
sudo usermod -aG docker $USER
```

## 2. Create folder and add .env:
```bash
mkdir -p /home/deploy/fbaserver
nano /home/deploy/fbaserver/.env
```
Paste your .env content here.

## Add your public SSH key to VPS:
```bash
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```
Paste the public key (corresponding to the private key in SSH_KEY secret).


# ✅ What Happens When You Push to main
- GitHub checks out your code.

- It securely connects to your VPS via SSH.

- It rsyncs the project files to /home/deploy/fbaserver.

- It runs docker compose up --build -d on your server.

- Your app starts with entrypoint.sh handling migration, seeding, and ASGI with Gunicorn + UvicornWorker.