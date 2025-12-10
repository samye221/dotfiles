# ⚡ Quick Start Guide - New Mac Setup

Fast-track installation guide for setting up your development environment on a new Mac using these dotfiles.

**Total Time:** ~30-45 minutes

---

## 🎯 TL;DR - Copy/Paste Installation

```bash
# 1. Install Command Line Tools
xcode-select --install

# 2. Clone dotfiles
cd ~ && git clone https://github.com/samye221/dotfiles.git && cd dotfiles

# 3. Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# 4. Install essential Homebrew packages
brew install gh jq node nvm pandoc pnpm wget yarn

# 5. Install Node.js
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts && nvm use --lts && nvm alias default node

# 6. Install npm packages
npm install -g corepack pnpm yarn wrangler

# 7. Setup shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
cp .zshrc ~/.zshrc && cp .gitconfig ~/.gitconfig

# 8. Setup Claude Code
mkdir -p ~/.claude/{agents,rules}
cp .claude/CLAUDE.md ~/.claude/
cp .claude/settings.json ~/.claude/
cp -r .claude/agents/* ~/.claude/agents/
cp -r .claude/rules/* ~/.claude/rules/

# 9. Configure Git
git config --global user.name "Samuel Lambert"
git config --global user.email "your.email@example.com"

# 10. Restart terminal
exec zsh
```

---

## 📋 Step-by-Step Guide

### Phase 1: Prerequisites (5 min)

```bash
# Install Command Line Tools (required for git)
xcode-select --install
# Wait for installation to complete

# Clone dotfiles
cd ~
git clone https://github.com/samye221/dotfiles.git
cd dotfiles
```

### Phase 2: Homebrew (10 min)

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add to PATH (Apple Silicon)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# Install packages (choose one):

# Option A: Install all packages from list
while IFS= read -r pkg; do
  [[ -n "$pkg" && ! "$pkg" =~ ^# ]] && brew install "$pkg"
done < packages/brew-packages.txt

# Option B: Install essentials only (recommended)
brew install gh jq node nvm pandoc pnpm wget yarn
```

### Phase 3: Node.js (5 min)

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Load nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Install Node.js LTS
nvm install --lts
nvm use --lts
nvm alias default node

# Install npm packages
while IFS= read -r pkg; do
  [[ -n "$pkg" && ! "$pkg" =~ ^# ]] && npm install -g "$pkg"
done < packages/npm-global.txt
```

### Phase 4: Shell Configuration (2 min)

```bash
# Backup existing configs (if any)
[ -f ~/.zshrc ] && cp ~/.zshrc ~/.zshrc.backup
[ -f ~/.gitconfig ] && cp ~/.gitconfig ~/.gitconfig.backup

# Install oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Copy configurations
cp .zshrc ~/.zshrc
cp .gitconfig ~/.gitconfig

# Configure Git identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Phase 5: Claude Code (3 min)

```bash
# Create directories
mkdir -p ~/.claude/{agents,rules}

# Copy configurations
cp .claude/CLAUDE.md ~/.claude/
cp .claude/settings.json ~/.claude/
cp -r .claude/agents/* ~/.claude/agents/
cp -r .claude/rules/* ~/.claude/rules/

# Verify
ls ~/.claude/agents/    # Should show 9 agents
ls ~/.claude/rules/universal/  # Should show 5 rules
```

### Phase 6: Personal Agent (1 min)

```bash
# Clone personal dotfiles (PRIVATE repo)
cd ~
git clone https://github.com/samye221/dotfiles-personal.git

# Copy personal agent
cp dotfiles-personal/.claude/agents/juriste-expert-complet.md ~/.claude/agents/
```

### Phase 7: VSCode (Optional, 5 min)

```bash
# Install VSCode
brew install --cask visual-studio-code

# Install 'code' command
# Open VSCode > Command Palette (Cmd+Shift+P) > "Shell Command: Install 'code' command in PATH"

# Install extensions
while IFS= read -r ext; do
  [[ -n "$ext" && ! "$ext" =~ ^# ]] && code --install-extension "$ext"
done < packages/vscode-extensions.txt
```

### Phase 8: Secrets & Credentials (15 min)

```bash
# Follow SECRETS.md guide
cat SECRETS.md

# Key actions:
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
ssh-add ~/.ssh/id_ed25519

# 2. Add SSH key to GitHub
pbcopy < ~/.ssh/id_ed25519.pub
# Go to https://github.com/settings/keys

# 3. Authenticate GitHub CLI
gh auth login

# 4. Test SSH connection
ssh -T git@github.com
```

### Phase 9: Final Steps (1 min)

```bash
# Restart terminal
exec zsh

# Verify everything works
brew --version
node --version
git --version
gh auth status
```

---

## ✅ Verification Checklist

After installation, verify:

```bash
# Core tools
✓ brew --version
✓ git --version
✓ node --version
✓ pnpm --version
✓ gh auth status

# Git configuration
✓ git config --global user.name
✓ git config --global user.email
✓ git st  # Test alias

# SSH
✓ ssh -T git@github.com

# Claude Code
✓ ls ~/.claude/CLAUDE.md
✓ ls ~/.claude/agents/ | wc -l  # Should be 10
✓ ls ~/.claude/rules/universal/ | wc -l  # Should be 5

# VSCode (if installed)
✓ code --version
✓ code --list-extensions | wc -l  # Should be ~14
```

---

## ⏱️ Estimated Time

| Task | Time |
|------|------|
| Prerequisites | 5 min |
| Homebrew + packages | 10 min |
| Node.js + npm | 5 min |
| Shell config | 2 min |
| Claude Code | 3 min |
| Personal agent | 1 min |
| VSCode (optional) | 5 min |
| Secrets/SSH | 15 min |
| **Total (minimal)** | **~30 min** |
| **Total (complete)** | **~45 min** |

---

## 🆘 Troubleshooting

### Homebrew installation fails
```bash
# Check if XCode Command Line Tools are installed
xcode-select -p
# If not: xcode-select --install
```

### nvm not found after installation
```bash
# Manually load nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
# Or restart terminal: exec zsh
```

### SSH key not working
```bash
# Check if key is added to agent
ssh-add -l
# Re-add if needed
ssh-add ~/.ssh/id_ed25519
```

### Git aliases not working
```bash
# Verify .gitconfig was copied
cat ~/.gitconfig | grep alias
# If empty, recopy: cp ~/dotfiles/.gitconfig ~/.gitconfig
```

---

## 📚 Additional Resources

- [Full README](./README.md) - Detailed documentation
- [SECRETS.md](./SECRETS.md) - Complete security guide
- [GitHub repo](https://github.com/samye221/dotfiles)

---

## 🔄 Updating Dotfiles

To save changes from your current system:

```bash
cd ~/dotfiles

# Update package lists
brew list --formula > packages/brew-packages.txt
code --list-extensions > packages/vscode-extensions.txt
npm list -g --depth=0 | tail -n +2 | awk -F'@' '{print $1}' | sed 's/[├│└─ ]//g' | grep -v '^$' > packages/npm-global.txt

# Commit and push
git add -A
git commit -m "Update packages $(date +%Y-%m-%d)"
git push
```

---

**Last Updated:** December 2024
