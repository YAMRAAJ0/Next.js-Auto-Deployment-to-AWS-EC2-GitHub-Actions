🚀 Next.js Auto Deployment to AWS EC2 (GitHub Actions)

This document explains how to automatically deploy a Next.js project to AWS EC2 whenever code is pushed to a specific GitHub branch.

✅ No manual SSH
✅ No username/password prompts
✅ Secure SSH + GitHub Actions
✅ Reusable for all future projects

📌 Tech Stack

Next.js

Node.js

PM2

AWS EC2 (Ubuntu)

GitHub Actions

SSH Authentication

📁 Server Folder Structure (EC2)
/home/ubuntu/
 └── PROJECT_NAME/
     ├── app/
     ├── package.json
     ├── next.config.js
     ├── .env.local        (server only)
     └── .git

1️⃣ EC2 SERVER SETUP (ONE TIME)
Login to EC2
ssh -i your-key.pem ubuntu@EC2_PUBLIC_IP

Install required tools
sudo apt update
sudo apt install -y git nodejs npm
sudo npm install -g pm2

2️⃣ CLONE PROJECT ON EC2 (ONE TIME)
cd /home/ubuntu
git clone git@github.com:YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
git checkout aws-uploaded-code

3️⃣ ENVIRONMENT VARIABLES (EC2 ONLY)

Create .env.local:

nano .env.local


Example:

MONGODB_URI=mongodb://127.0.0.1:27017/dbname
NEXTAUTH_SECRET=your_secret


⚠️ Never commit .env.local

Ensure .gitignore includes:

.env*

4️⃣ BUILD & RUN WITH PM2 (ONE TIME)
npm install
npm run build
pm2 start npm --name PROJECT_NAME -- start
pm2 save


Check status:

pm2 status

5️⃣ SSH KEY SETUP FOR AUTO DEPLOY (ONE TIME)
Generate SSH key on EC2
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C "ec2-auto-deploy"


(Press ENTER for passphrase)

Copy public key
cat ~/.ssh/id_ed25519.pub

Add key to GitHub

GitHub → Settings → SSH and GPG keys → New SSH key

6️⃣ SWITCH EC2 REPO TO SSH (IMPORTANT)
cd /home/ubuntu/YOUR_REPO
git remote set-url origin git@github.com:YOUR_USERNAME/YOUR_REPO.git
git pull origin aws-uploaded-code


✅ No username or password should be asked.

7️⃣ ADD GITHUB SECRETS (REQUIRED)

GitHub → Repo → Settings → Secrets & Variables → Actions

Add these secrets:

Name	Value
EC2_HOST	EC2 Public IP
EC2_USER	ubuntu
EC2_KEY	Private SSH key (id_ed25519)
8️⃣ GITHUB ACTIONS WORKFLOW

Create file:

.github/workflows/deploy.yml

deploy.yml
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

9️⃣ DEPLOY FLOW (HOW IT WORKS)
git push aws-uploaded-code
        ↓
GitHub Actions
        ↓
SSH into EC2
        ↓
Pull code
        ↓
Build Next.js
        ↓
Restart PM2
        ↓
LIVE 🚀

🔄 DEPLOY NEW CHANGES (EVERY TIME)

From local machine:

git push origin aws-uploaded-code


No SSH required.

🧪 VERIFY ON EC2
pm2 logs PROJECT_NAME
git log -1 --oneline

⚠️ COMMON ISSUES
Update not showing
rm -rf .next
npm run build
pm2 restart PROJECT_NAME

GitHub Action green but no update

✔ Check correct project path in deploy.yml

✅ BEST PRACTICES

❌ Never commit .env, .next, node_modules

✅ Always use SSH authentication

✅ One branch for deployment

✅ PM2 for production

✅ Build on server

🏁 FINAL RESULT

✔ Secure
✔ Automatic
✔ Production-ready
✔ Reusable for all future projects
