
# paths overview
| Option                          | Static | SSR | Complexity   | Best For              |
| ------------------------------- | ------ | --- | ------------ | --------------------- |
| Vercel                          | ✅      | ✅   | ⭐ Easiest    | 95% of projects       |
| Static → VPS + Nginx            | ✅      | ❌   | ⭐⭐ Simple    | blogs, landing pages  |
| Full Next.js Server → VPS + PM2 | ✅      | ✅   | ⭐⭐⭐ Flexible | dashboards, auth apps |


**Build the Next.js app using GitHub Actions → Upload to VPS via SSH → Serve via Nginx or Node (PM2)**

# Overview: Next.js Deployment via GitHub Actions → VPS (with PM2 or Nginx)

## 🔄 Flow:
### 1. GitHub Actions:

- Installs dependencies

- Builds the Next.js app (next build)

- Exports output or prepares .next and node_modules

- Uses scp to upload to your VPS

### VPS:

- Option 1: Use pm2 to run next start (for SSR)

- Option 2: Use Nginx to serve static files (for static builds only)

## choose your mode
| Mode              | Command                    | For SSR/API? | Tools Needed   |
| ----------------- | -------------------------- | ------------ | -------------- |
| **Static Export** | `next export`              | ❌ No         | Nginx only     |
| **Node Server**   | `next build && next start` | ✅ Yes        | PM2 or systemd |
You selected Node Server (SSR) mode.

# 🧰 What You’ll Need on the VPS
Node.js (same version as in your project)

PM2 (npm i -g pm2)

.env file (not committed to GitHub)

Directory like /home/deploy/my-next-app

Optional: Nginx as a reverse proxy

# 🔐 GitHub Secrets Needed
| Secret Name        | Description                          |
| ------------------ | ------------------------------------ |
| `VPS_SSH_KEY`      | SSH private key                      |
| `VPS_USER`         | VPS username (e.g. `deploy`)         |
| `VPS_HOST`         | VPS IP address                       |
| `VPS_FRONTEND_DIR` | Path like `/home/deploy/fba-frontend` |

# 📦 GitHub Actions Workflow (Preview)
```yml
name: Deploy Next.js Frontend

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: 22

    - name: Install dependencies
      run: npm ci

    - name: Build Next.js app
      run: npm run build

    - name: Upload build to VPS
      uses: appleboy/scp-action@v0.1.7
      with:
        host: ${{ secrets.VPS_HOST }}
        username: ${{ secrets.VPS_USER }}
        key: ${{ secrets.VPS_SSH_KEY }}
        source: ".next/,public/,package.json,ecosystem.config.js,*.js,*.json"
        target: ${{ secrets.VPS_FRONTEND_DIR }}

    - name: Restart PM2 on VPS
      run: |
        ssh -o StrictHostKeyChecking=no ${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }} '
          cd ${{ secrets.VPS_FRONTEND_DIR }} &&
          npm install --omit=dev &&
          pm2 reload ecosystem.config.js || pm2 start ecosystem.config.js
        '
```

# 📁 Required VPS Files
## ecosystem.config.js (in your project)
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


