# 🔐 Secrets & Credentials Setup Guide

This file contains instructions for configuring sensitive credentials that are **NOT included** in this repository for security reasons.

**⚠️ NEVER commit this file with real secrets filled in!**

---

## 📋 Quick Checklist

After completing the dotfiles installation, configure these secrets:

- [ ] SSH Keys (GitHub, GitLab, servers)
- [ ] Git User Identity
- [ ] GitHub Personal Access Token
- [ ] npm Authentication (if publishing packages)
- [ ] Claude Code MCP Servers (optional)
- [ ] API Keys (Brave, Perplexity, etc.) - optional

---

## 1. SSH Keys

### Generate new SSH key

```bash
# Generate ED25519 key (recommended)
ssh-keygen -t ed25519 -C "your.email@example.com"

# Or RSA if ED25519 not supported
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"

# Default location: ~/.ssh/id_ed25519 (press Enter)
# Set a strong passphrase when prompted
```

### Add to ssh-agent

```bash
# Start ssh-agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519
```

### Add to GitHub

```bash
# Copy public key to clipboard
pbcopy < ~/.ssh/id_ed25519.pub

# Then go to: https://github.com/settings/keys
# Click "New SSH key"
# Paste and save
```

### Test connection

```bash
ssh -T git@github.com
# Should see: "Hi username! You've successfully authenticated..."
```

---

## 2. Git User Identity

```bash
# Configure your name and email
git config --global user.name "Your Full Name"
git config --global user.email "your.email@example.com"

# Verify
git config --global --list | grep user
```

---

## 3. GitHub Personal Access Token

### When to create

You need this for:
- `gh` CLI tool
- Private repository access
- GitHub API usage
- npm packages from GitHub registry

### Create token

1. Go to: https://github.com/settings/tokens
2. Click "Generate new token (classic)"
3. Scopes needed:
   - `repo` (full repository access)
   - `workflow` (if using GitHub Actions)
   - `read:packages` (if using GitHub packages)
4. Click "Generate token"
5. **Copy the token immediately** (you won't see it again!)

### Configure `gh` CLI

```bash
# Login with gh
gh auth login

# Follow prompts:
# - GitHub.com
# - HTTPS
# - Yes (authenticate)
# - Paste your token
```

### Store for other uses (optional)

```bash
# Add to ~/.zshrc if needed by applications
echo 'export GITHUB_TOKEN="your_github_token_here"' >> ~/.zshrc
source ~/.zshrc
```

**⚠️ Security:** Consider using environment variables in a `.env` file (gitignored) instead of `.zshrc`.

---

## 4. npm Authentication

### For public npm packages

```bash
# Login to npm
npm login

# Follow prompts:
# - Username
# - Password
# - Email
# - One-time password (if 2FA enabled)
```

### For private registries (optional)

```bash
# Configure registry
npm config set registry https://your-registry.com

# Or create ~/.npmrc with:
# //your-registry.com/:_authToken=YOUR_TOKEN
```

---

## 5. Claude Code MCP Servers (Optional)

The dotfiles include generic MCP servers (browsermcp, playwright).

### If you need company-specific MCP servers:

Edit `~/.claude/settings.json` and add:

```json
{
  "mcpServers": {
    "your-company-gitlab": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "GITLAB_PERSONAL_ACCESS_TOKEN", "mcp/gitlab"],
      "env": {
        "GITLAB_PERSONAL_ACCESS_TOKEN": "${GITLAB_PERSONAL_ACCESS_TOKEN}",
        "GITLAB_API_URL": "https://gitlab.company.com/api/v4"
      }
    }
  }
}
```

Then set environment variables in `~/.zshrc`:

```bash
export GITLAB_PERSONAL_ACCESS_TOKEN="your_token"
```

---

## 6. Optional API Keys

### Brave Search API (for web search)

1. Sign up at: https://brave.com/search/api/
2. Get API key
3. Add to ~/.zshrc:
   ```bash
   export BRAVE_API_KEY="your_brave_api_key"
   ```

### Perplexity API (for AI search)

1. Sign up at: https://www.perplexity.ai/
2. Get API key from dashboard
3. Add to ~/.zshrc:
   ```bash
   export PERPLEXITY_API_KEY="your_perplexity_key"
   ```

---

## 7. Company-Specific Credentials

### For VPTech/Veepee (if still needed)

**Note:** These are company-specific and were removed from dotfiles.

```bash
# GitLab VPTech
export GITLAB_TOKEN="your_vptech_gitlab_token"
export GITLAB_URL="https://gitlab.vptech.eu"

# Jira VPTech
export JIRA_TOKEN="your_jira_token"
export JIRA_URL="https://jira.vptech.eu"
export JIRA_EMAIL="your.email@veepee.com"
```

### For your new company

Replace with your new company's credentials following their documentation.

---

## 🔒 Security Best Practices

### 1. Use a password manager

- Store all tokens/passwords in a password manager (1Password, Bitwarden, etc.)
- Never store secrets in plain text files

### 2. Enable 2FA everywhere

- GitHub
- npm
- GitLab
- Company accounts

### 3. Rotate tokens regularly

- GitHub tokens: Rotate every 6-12 months
- API keys: Check for unused keys and revoke them

### 4. Use environment-specific configs

```bash
# Create a .env file (git ignored)
touch ~/.env

# Add secrets there
echo 'export SECRET_KEY="value"' >> ~/.env

# Load in .zshrc
echo 'source ~/.env 2>/dev/null' >> ~/.zshrc
```

### 5. Never commit secrets

Double-check before committing:

```bash
# Check what's being committed
git diff --cached

# If you accidentally committed secrets:
git reset HEAD~1  # Undo last commit (if not pushed)
# Then remove secrets and recommit
```

---

## ✅ Verification Checklist

After configuration, verify everything works:

```bash
# SSH
ssh -T git@github.com

# Git
git config --global user.name
git config --global user.email

# GitHub CLI
gh auth status

# npm
npm whoami

# Claude Code
# Open Claude Code and check MCP servers are connected
```

---

## 🆘 Troubleshooting

### SSH key not working

```bash
# Check ssh-agent is running
ssh-add -l

# Re-add key
ssh-add ~/.ssh/id_ed25519

# Test with verbose output
ssh -vT git@github.com
```

### GitHub token expired

1. Go to https://github.com/settings/tokens
2. Click on the token
3. Click "Regenerate token"
4. Update everywhere you used the old token

### npm authentication failed

```bash
# Clear npm cache
npm cache clean --force

# Re-login
npm logout
npm login
```

---

## 📚 Additional Resources

- [GitHub SSH Keys Documentation](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [GitHub Tokens Documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [npm Documentation](https://docs.npmjs.com/cli/v8/commands/npm-login)
- [Claude Code MCP Servers](https://docs.claude.com/en/docs/claude-code/mcp)

---

**Last Updated:** December 2024
