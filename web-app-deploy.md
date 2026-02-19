# 🚀 COMPLETE GitHub Action (Production Ready)

(SSH + Docker Compose)

This guide sets up GitHub Actions to SSH into your VM and deploy your app using docker-compose whenever you push to the main branch.

1) Create GitHub Actions Workflow Folder

 📂 File Location

```bash
mkdir -p .github/workflows

2. Create deploy.yml inside .github/workflows

Create this file:

.github/workflows/deploy.yml

You are on the server here:

```
root@srv814942:~/Leaflet_Production#
```

Since you are logged in as root, your absolute path is likely:

```
/root/Leaflet_Production
```

Paste this workflow content exactly:

name: Docker Deploy
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Leaflet_Production
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }} # Ensure this secret is set to: root
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            set -e

            # Navigate to your specific folder
            cd /root/Leaflet_Production

            # Pull latest changes
            git pull origin main

            # Rebuild and restart containers
            docker-compose down
            docker-compose up -d --build

            # Cleanup
            docker image prune -f
```
3. Generate SSH Key for GitHub Actions (Deploy Key)

Run:

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f github_deploy_key
```

Add the public key to authorized keys:

```bash
cat github_deploy_key.pub >> authorized_keys
```

Show the private key:

```bash
cat github_deploy_key
```

Copy the entire output including:

```
-----BEGIN OPENSSH PRIVATE KEY-----
```

and

```
-----END OPENSSH PRIVATE KEY-----
```

It starts with:

```
-----BEGIN OPENSSH PRIVATE KEY-----
```

It ends with:

```
-----END OPENSSH PRIVATE KEY-----
```

---

4. Add Secrets to GitHub Repository

Steps:

1. Click the Settings tab at the top of your repository.
2. On the left sidebar, look for Secrets and variables.

Here is exactly where to look and click:

Look at the "Repository secrets" section (the bottom half of your screen).

On the far right of that section header, there is a green button named New repository secret.

Click that button.

You must set these secrets:

* SERVER_HOST → your VM IP/hostname
* SERVER_USER → root
* SERVER_SSH_KEY → paste the entire private key you copied from `cat github_deploy_key`

---

5. Switch Server Connection to SSH using a Deploy Key (Git Pull Auth)

We need to switch your server's connection to SSH so it can authenticate automatically using a "Deploy Key."

Generate a default SSH key on the server:

```bash
ssh-keygen -t ed25519 -C "server_pull_key"
```

Print the public key:

```bash
cat /root/.ssh/id_ed25519.pub
```

---

6. Add Deploy Key in GitHub Repo (NOT Secrets)

Steps:

1. Click Settings (top tabs).
2. On the left sidebar, click Deploy keys (Note: This is different from "Secrets"!).

Click Add deploy key and fill:

* Title: Production Server
* Key: Paste the public key you just copied.
* Check the box Allow write access

---

7. Change Git Remote URL to SSH (On Server)

Change the Link to SSH:

```bash
cd /root/Leaflet_Production
git remote set-url origin git@github.com:kumar045/Leaflet_Production.git
```

---

8. Trust GitHub Host Once (Manual First-Time SSH)

Finally we need to manually force the connection once to save GitHub as a trusted host.

```bash
ssh -T git@github.com
```

Type `Yes` and press enter.
