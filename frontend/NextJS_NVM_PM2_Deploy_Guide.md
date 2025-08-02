# 🚀 Deploy Next.js Frontend via GitHub Actions to VPS using NVM + PM2 (Node.js v22)

This guide shows how to deploy a Next.js frontend to your VPS using GitHub Actions. It installs Node.js v22 via NVM, uses PM2 to manage the server process, and sets up GitHub Actions for automatic deployment.

---

## 🧰 VPS Setup

SSH into your VPS and run:

```bash
# 1. Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# 2. Load NVM into current session
. "$HOME/.nvm/nvm.sh"

# 3. Install and use Node.js v22
nvm install 22
nvm use 22

# 4. Install PM2 globally
npm install -g pm2

# 5. Create a directory for your frontend
mkdir -p /home/deploy/fba-frontend
```

To make NVM and PM2 available in non-login shells (used by GitHub Actions), add this to your `~/.bashrc`:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

###  ❓ Why do we need this in ~/.bashrc?
When GitHub Actions connects to your VPS via SSH, it does so using a non-login, non-interactive shell.

That means:

Your .bashrc or .bash_profile might not run fully

Environment setup (like loading NVM) may be skipped

As a result, commands like nvm, node, or pm2 might not be found during automated deployment

###✅ What does this do?
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```
| Line                          | Purpose                                                              |
| ----------------------------- | -------------------------------------------------------------------- |
| `export NVM_DIR="$HOME/.nvm"` | Tells the shell where NVM is installed (standard location)           |
| `[ -s "$NVM_DIR/nvm.sh" ]`    | Checks if the NVM script exists and is not empty                     |
| `\. "$NVM_DIR/nvm.sh"`        | Loads NVM into the shell (so you can use `nvm`, `node`, `pm2`, etc.) |


---

## 🔐 GitHub Secrets Required

Add the following in your repo:

| Name               | Description                      |
|--------------------|----------------------------------|
| `VPS_SSH_KEY`      | Your private SSH key             |
| `VPS_USER`         | VPS username (e.g. deploy)       |
| `VPS_HOST`         | VPS IP address                   |
| `VPS_FRONTEND_DIR` | Directory on VPS (e.g. `/home/deploy/fba-frontend`) |

---

## 📄 GitHub Actions Workflow

Save this file as `.github/workflows/deploy-frontend.yml`:

```yaml
name: 🚀 Deploy Next.js Frontend to VPS

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest

    steps:
    - name: 🧾 Checkout source code
      uses: actions/checkout@v4

    - name: 🧰 Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: 22
        cache: 'npm'

    - name: 📦 Install dependencies
      run: npm ci

    - name: 🏗️ Build Next.js app
      run: npm run build

    - name: 📤 Upload build to VPS via SCP
      uses: appleboy/scp-action@v0.1.7
      with:
        host: ${{ secrets.VPS_HOST }}
        username: ${{ secrets.VPS_USER }}
        key: ${{ secrets.VPS_SSH_KEY }}
        source: |
          .next
          public
          package.json
          package-lock.json
          ecosystem.config.js
        target: ${{ secrets.VPS_FRONTEND_DIR }}

    - name: 🔁 Restart frontend with PM2 on VPS (NVM aware)
      run: |
        ssh -o StrictHostKeyChecking=no ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }} << 'EOF'
          export NVM_DIR="$HOME/.nvm"
          . "$NVM_DIR/nvm.sh"
          nvm use 22
          cd ${{ secrets.VPS_FRONTEND_DIR }}
          npm install --omit=dev
          pm2 reload ecosystem.config.js || pm2 start ecosystem.config.js
        EOF
```

---

## ⚙️ PM2 Config: `ecosystem.config.js`

Place this file at the root of your frontend project:

```js
module.exports = {
  apps: [
    {
      name: "nextjs-frontend",
      script: "node_modules/next/dist/bin/next",
      args: "start -p 3000",
      env: {
        NODE_ENV: "production",
      },
    },
  ],
};
```

---

## ✅ Done!

You can now:

- Push to `main`
- GitHub Actions will build, deploy, and restart your Next.js frontend via PM2
- Access your app at `http://your-vps-ip:3000`

➡️ Optional: Use Nginx to reverse proxy to `http://localhost:3000` for domain + HTTPS.