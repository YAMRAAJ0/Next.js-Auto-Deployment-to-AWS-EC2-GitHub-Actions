# 🚀 Next.js Auto Deployment to AWS EC2 (GitHub Actions)

This guide explains how to **automatically deploy a Next.js application to AWS EC2**
whenever code is pushed to a specific GitHub branch.

### ✅ Features
- No manual SSH after setup
- No username/password prompts
- Secure SSH-based deployment
- GitHub Actions CI/CD
- Reusable for all future projects

---

## 📌 Tech Stack

- **Next.js**
- **Node.js**
- **PM2**
- **AWS EC2 (Ubuntu)**
- **GitHub Actions**
- **SSH Authentication**

---

## 📁 Server Folder Structure (EC2)

/home/ubuntu/
└── PROJECT_NAME/
├── app/
├── package.json
├── next.config.js
├── .env.local # server only (not committed)
└── .git

---

## 1️⃣ EC2 Server Setup (One Time)

### Login to EC2
```bash
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP
Install required tools
```
---
```bash
sudo apt update
sudo apt install -y git nodejs npm
sudo npm install -g pm2
```
---
2️⃣ Clone Project on EC2 (One Time)
```bash
cd /home/ubuntu
git clone git@github.com:YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
git checkout aws-uploaded-code
```
---
3️⃣ Environment Variables (EC2 Only)
Create .env.local:

```bash
nano .env.local
```
---
Example:

env
MONGODB_URI=mongodb://127.0.0.1:27017/dbname
NEXTAUTH_SECRET=your_secret
⚠️ Never commit .env.local

Ensure .gitignore includes:
---

.env*
---
4️⃣ Build & Run with PM2 (One Time)
```bash

npm install
npm run build
pm2 start npm --name PROJECT_NAME -- start
pm2 save
```
---
Check status:
```bash
pm2 status
```
---
5️⃣ SSH Key Setup for Auto Deploy (One Time)
Generate SSH key on EC2
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C "ec2-auto-deploy"
```
---
➡️ Press ENTER for passphrase

Copy public key
bash
cat ~/.ssh/id_ed25519.pub
Add key to GitHub
vbnet
GitHub → Settings → SSH and GPG keys → New SSH key
6️⃣ Switch EC2 Repo to SSH (Important)
```bash
cd /home/ubuntu/YOUR_REPO
git remote set-url origin git@github.com:YOUR_USERNAME/YOUR_REPO.git
git pull origin aws-uploaded-code
```
---
✅ No username or password should be asked.

7️⃣ Add GitHub Secrets (Required)
GitHub → Repository → Settings → Secrets & Variables → Actions

Add the following secrets:

Name	Value
EC2_HOST	EC2 Public IP
EC2_USER	ubuntu
EC2_KEY	Private SSH key (id_ed25519)

8️⃣ GitHub Actions Workflow
Create the file:

```bash
.github/workflows/deploy.yml
```
---
yaml
```
name: Deploy to AWS EC2

on:
  push:
    branches:
      - aws-uploaded-code

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_KEY }}
          script: |
            cd /home/ubuntu/YOUR_REPO
            git pull origin aws-uploaded-code
            npm install
            npm run build
            pm2 restart PROJECT_NAME --update-env || pm2 start npm --name PROJECT_NAME -- start
```
---
9️⃣ Deployment Flow (How It Works)
armasm
Copy code
git push aws-uploaded-code
        ↓
GitHub Actions
        ↓
SSH into EC2
        ↓
Pull latest code
        ↓
Build Next.js
        ↓
Restart PM2
        ↓
LIVE 🚀
🔄 Deploy New Changes (Every Time)
From local machine:

bash
git push origin aws-uploaded-code
✅ No manual SSH required.

🧪 Verify on EC2
bash
```
pm2 logs PROJECT_NAME
git log -1 --oneline
```
⚠️ Common Issues & Fixes
❌ Changes not showing
bash
```
rm -rf .next
npm run build
```
pm2 restart PROJECT_NAME
❌ GitHub Action is green but no update
✔ Check correct project path in deploy.yml

✅ Best Practices
❌ Never commit .env, .next, node_modules

✅ Always use SSH authentication

✅ Use a dedicated deployment branch

✅ Use PM2 in production

✅ Build on the server, not locally

🏁 Final Result
✔ Secure
✔ Fully Automatic
✔ Production-Ready
✔ Reusable for Future Projects

Happy Deploying 🚀

markdown

---

### ✅ What improved
- Clean headings & spacing
- Proper code blocks
- Tables for secrets
- Clear flow & readability
- Professional GitHub appearance

If you want next, I can:
- Add **deployment architecture diagram**
- Convert this into a **template repository**
- Create **Docker-based README**
- Add **Nginx + SSL steps**

Just tell me 👍
