# 📂 File Location

Create this file:

```
.github/workflows/eas-update.yml
```

---

# 🚀 COMPLETE GitHub Action (Production Ready)

```yaml
name: 🚀 EAS Update + QR (Expo Go via VPS)

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

concurrency:
  group: eas-update-${{ github.ref }}
  cancel-in-progress: true

jobs:
  publish_update:
    name: Publish OTA Update on VPS
    runs-on: ubuntu-latest

    steps:
      - name: 🔐 SSH into VPS and Publish Update
        uses: appleboy/ssh-action@v1.0.3

        env:
          EXPO_TOKEN: ${{ secrets.EXPO_TOKEN }}
          EXPO_PROJECT_ID: ${{ secrets.EXPO_PROJECT_ID }}
          EXPO_RUNTIME_VERSION: ${{ secrets.EXPO_RUNTIME_VERSION }}
          EXPO_CHANNEL: ${{ secrets.EXPO_CHANNEL }}
          GITHUB_SHA: ${{ github.sha }}
          GITHUB_REF_NAME: ${{ github.ref_name }}

        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          port: 22
          script_stop: true
          envs: EXPO_TOKEN,EXPO_PROJECT_ID,EXPO_RUNTIME_VERSION,EXPO_CHANNEL,GITHUB_SHA,GITHUB_REF_NAME

          script: |
            set -e

            echo "📦 Moving to project directory..."
            cd /root/Leaflet_Production

            echo "🔄 Pulling latest code..."
            git fetch --all
            git reset --hard origin/main

            echo "📥 Installing dependencies..."
            npm ci

            echo "🧰 Checking EAS CLI..."
            npx --yes eas-cli --version

            echo "🚀 Publishing EAS Update..."
            npx --yes eas-cli update \
              --branch "$EXPO_CHANNEL" \
              --message "CI $GITHUB_REF_NAME $GITHUB_SHA" \
              --non-interactive

            echo ""
            echo "========================================"
            echo "✅ EAS Update Published Successfully!"
            echo "========================================"
            echo ""
            echo "📱 Expo Go QR Code (SVG):"
            echo "https://qr.expo.dev/eas-update?projectId=$EXPO_PROJECT_ID&runtimeVersion=$EXPO_RUNTIME_VERSION&channel=$EXPO_CHANNEL"
            echo ""
            echo "🔗 Expo Go Deep Link:"
            echo "https://qr.expo.dev/eas-update?projectId=$EXPO_PROJECT_ID&runtimeVersion=$EXPO_RUNTIME_VERSION&channel=$EXPO_CHANNEL&format=url"
            echo ""
            echo "Open the SVG link above on any screen and scan with Expo Go."
```

---

# 🔐 Required GitHub Secrets

Go to:

```
Repo → Settings → Secrets and variables → Actions
```

Add these:

| Secret               | Value                    |
| -------------------- | ------------------------ |
| SERVER_HOST          | Your VPS IP              |
| SERVER_USER          | root (or deploy user)    |
| SERVER_SSH_KEY       | Private SSH key          |
| EXPO_TOKEN           | From `expo token:create` |
| EXPO_PROJECT_ID      | From app.json            |
| EXPO_RUNTIME_VERSION | From app.json            |
| EXPO_CHANNEL         | main                     |

---

# 🏗 What This Workflow Does

When you push to `main`:

1. GitHub Action starts
2. SSH connects to VPS
3. Hard resets to latest main
4. Installs dependencies cleanly
5. Runs `eas update`
6. Prints QR link in logs
7. Expo Go loads update instantly

No rebuild required for JS changes.

---

# 🛡 Why This Version Is Better

* Uses `git reset --hard` (no merge conflicts)
* Uses concurrency control
* Stops on failure
* Clean logs
* No password authentication
* Fully automated

---

# 📱 How to Use After Push

1. Push to `main`
2. Go to GitHub → Actions tab
3. Open workflow logs
4. Copy QR link
5. Scan with Expo Go
6. App updates instantly
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

Type `Yes` and press enter

# 🚀 EAS Update + QR (Expo Go) – CI/CD Setup Guide

This guide explains how to configure GitHub Actions to:

- Publish an **EAS Update**
- Automatically generate a **QR code**
- Scan it with **Expo Go**
- Deploy directly from your VPS

---

# 🔐 Required GitHub Secrets

Go to:

```

Repo → Settings → Secrets and variables → Actions → New repository secret

````

You only need **4 Expo secrets**.

---

## 1️⃣ EXPO_TOKEN (REQUIRED)

Used by CI to authenticate with your Expo account.

### Generate Token

Run once locally:

```bash
npx expo login
npx expo whoami
npx expo token:create
````

You’ll get something like:

```
expo_xxxxxxxxxxxxxxxxxxxxxxxxx
```

### Add to GitHub

| Name         | Value                          |
| ------------ | ------------------------------ |
| `EXPO_TOKEN` | expo_xxxxxxxxxxxxxxxxxxxxxxxxx |

---

## 2️⃣ EXPO_PROJECT_ID (REQUIRED)

Unique identifier for your Expo project.

### Find It

Open `app.json` or `app.config.js`:

```json
{
  "expo": {
    "extra": {
      "eas": {
        "projectId": "a1b2c3d4-1234-5678-9012-abcdef123456"
      }
    }
  }
}
```

That UUID is your project ID.

If missing:

```bash
npx eas project:init
```

Or:

```bash
npx expo config --json | grep projectId
```

### Add to GitHub

| Name              | Value     |
| ----------------- | --------- |
| `EXPO_PROJECT_ID` | your UUID |

---

## 3️⃣ EXPO_RUNTIME_VERSION (REQUIRED)

Must match your app's runtime version.

### Check `app.json`

Example 1:

```json
{
  "expo": {
    "runtimeVersion": "1.0.0"
  }
}
```

Example 2 (SDK policy):

```json
{
  "expo": {
    "runtimeVersion": {
      "policy": "sdkVersion"
    }
  }
}
```

If using `sdkVersion` policy → use your Expo SDK version:

Example:

```
50.0.0
```

Check with:

```bash
npx expo config | grep runtimeVersion
```

### Add to GitHub

| Name                   | Value           |
| ---------------------- | --------------- |
| `EXPO_RUNTIME_VERSION` | 1.0.0 or 50.0.0 |

---

## 4️⃣ EXPO_CHANNEL (REQUIRED)

Defines your update branch/channel.

Recommended:

```
main
```

### Add to GitHub

| Name           | Value |
| -------------- | ----- |
| `EXPO_CHANNEL` | main  |

---

# ⚙️ Automatically Provided (No Secret Needed)

These are automatically set by GitHub:

| Variable          | Description |
| ----------------- | ----------- |
| `GITHUB_SHA`      | Commit hash |
| `GITHUB_REF_NAME` | Branch name |

You do **NOT** need to create them manually.

---

# 🌐 Required Server Secrets

You should already have:

| Secret           | Description     |
| ---------------- | --------------- |
| `SERVER_HOST`    | VPS IP / domain |
| `SERVER_USER`    | SSH username    |
| `SERVER_SSH_KEY` | Private SSH key |

---

# 🧪 Final Checklist

Before using CI:

✅ Your app runs locally:

```bash
npx expo start
```

✅ Expo Go can open it
✅ `eas update` works locally

If that works → CI + QR will work.

---

# 📱 What Happens After Push

When you push to `main`:

1. GitHub connects to VPS
2. Pulls latest code
3. Runs `npm ci`
4. Runs `eas update`
5. Prints a QR link in GitHub Actions logs

Example QR link:

```
https://qr.expo.dev/eas-update?projectId=YOUR_ID&runtimeVersion=YOUR_VERSION&channel=main
```

Open that link on any screen and scan it with **Expo Go**.

---

# 🔥 Production-Ready Setup

This is the same structure used by real Expo production teams:

* Secure token-based auth
* Server-based publishing
* Channel-based updates
* Zero manual QR generation
* Full CI/CD automation
