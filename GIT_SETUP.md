# Git Push Setup Guide

## Problem: Authentication Required

GitHub requires authentication to push. You have two options:

## Option 1: Use Personal Access Token (Recommended)

### Step 1: Create GitHub Personal Access Token

1. Go to GitHub: https://github.com/settings/tokens
2. Click **Generate new token** → **Generate new token (classic)**
3. Give it a name: `CI-Project-Push`
4. Select scopes:
   - ✅ **repo** (full control of private repositories)
5. Click **Generate token**
6. **COPY THE TOKEN** (you won't see it again!)

### Step 2: Push Using Token

When you push, use the token as password:

```bash
git push origin master
# Username: your-github-username
# Password: paste-your-token-here (NOT your GitHub password)
```

### Step 3: Save Credentials (Optional)

To avoid entering credentials every time:

```bash
# Store credentials in macOS Keychain
git config --global credential.helper osxkeychain

# Then push (enter credentials once, they'll be saved)
git push origin master
```

## Option 2: Switch to SSH (More Secure)

### Step 1: Generate SSH Key

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your-email@example.com"

# Press Enter to accept default location
# Enter passphrase (optional but recommended)

# Copy public key
cat ~/.ssh/id_ed25519.pub
```

### Step 2: Add SSH Key to GitHub

1. Copy the output from above command
2. Go to: https://github.com/settings/keys
3. Click **New SSH key**
4. Title: `Mac CI-Project`
5. Paste your public key
6. Click **Add SSH key**

### Step 3: Change Remote to SSH

```bash
# Check current remote
git remote -v

# Change to SSH
git remote set-url origin git@github.com:manjukolkar/CI-Project.git

# Verify
git remote -v

# Test connection
ssh -T git@github.com
```

### Step 4: Push

```bash
git push origin master
```

## Quick Fix: Push Right Now

If you just want to push immediately:

```bash
# Method 1: Use token in URL (one-time)
git push https://YOUR_TOKEN@github.com/manjukolkar/CI-Project.git master

# Method 2: Use GitHub CLI (if installed)
gh auth login
git push origin master
```

## Troubleshooting

### "Permission denied"
- Check your SSH key is added to GitHub
- Verify SSH connection: `ssh -T git@github.com`

### "Authentication failed"
- Use Personal Access Token, not password
- Make sure token has `repo` scope

### "Repository not found"
- Check repository name: `manjukolkar/CI-Project`
- Verify you have push access

## Recommended Setup

For this project, I recommend:

1. **Use Personal Access Token** (easiest)
2. **Save to Keychain** (so you don't enter it every time)
3. **Or switch to SSH** (more secure long-term)

