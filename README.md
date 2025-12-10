# 🚀 Samuel's Dotfiles

Portable development environment configuration for macOS. Designed for quick setup on new machines.

**Author:** Samuel Lambert ([samye221](https://github.com/samye221))
**Last Updated:** December 2024

---

## 📦 What's Included

- **Claude Code Configuration** (`.claude/`)
  - 9 specialized agents (accessibility, architecture, code-review, debugging, frontend, performance, python, task-analysis, testing)
  - Universal rules for TypeScript, React, Testing, Performance
  - Clean settings (only generic MCP servers)

- **Shell Configuration**
  - `.zshrc` with oh-my-zsh
  - `.gitconfig` with useful aliases

- **Package Lists**
  - Homebrew formulas (~45 packages)
  - VSCode extensions (14 extensions)
  - npm global packages (essentials only)

---

## 🎯 Quick Start

**⚡ TL;DR:** See [QUICKSTART.md](./QUICKSTART.md) for a fast-track installation guide with copy/paste commands.

**📖 Detailed Guide:** Continue reading below for step-by-step instructions.

### Prerequisites
- macOS (tested on macOS 14+)
- Internet connection
- Command Line Tools: `xcode-select --install`

---

## 📝 Step-by-Step Installation Guide

### Step 1: Clone this repository

```bash
# Clone to your home directory
cd ~
git clone https://github.com/samye221/dotfiles.git
cd dotfiles
```

### Step 2: Install Homebrew

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add Homebrew to PATH (Apple Silicon Mac)
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# Verify installation
brew --version
```

### Step 3: Install Homebrew Packages

```bash
# Option A: Install all packages from list
while IFS= read -r package; do
  [[ -n "$package" && ! "$package" =~ ^# ]] && brew install "$package"
done < packages/brew-packages.txt

# Option B: Install selectively (recommended)
# Review packages/brew-packages.txt first
# Then install only what you need:
brew install gh jq node nvm pandoc pnpm wget yarn
```

**⚠️ Note:** Some packages in the list may no longer be available or may not be needed for your setup.

### Step 4: Install Node.js via nvm

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Load nvm (or restart terminal)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Install latest LTS
nvm install --lts
nvm use --lts
nvm alias default node

# Verify
node --version
npm --version
```

### Step 5: Install npm Global Packages

```bash
# Install from list
while IFS= read -r package; do
  [[ -n "$package" && ! "$package" =~ ^# ]] && npm install -g "$package"
done < packages/npm-global.txt

# Verify
npm list -g --depth=0
```

### Step 6: Setup Shell Configuration

```bash
# Backup existing config (if any)
[ -f ~/.zshrc ] && cp ~/.zshrc ~/.zshrc.backup.$(date +%Y%m%d)
[ -f ~/.gitconfig ] && cp ~/.gitconfig ~/.gitconfig.backup.$(date +%Y%m%d)

# Install oh-my-zsh (if not already installed)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Copy configurations
cp .zshrc ~/.zshrc
cp .gitconfig ~/.gitconfig

# Configure Git identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Step 7: Setup Claude Code

```bash
# Create .claude directory
mkdir -p ~/.claude

# Copy CLAUDE.md
cp .claude/CLAUDE.md ~/.claude/CLAUDE.md

# Copy settings.json
cp .claude/settings.json ~/.claude/settings.json

# Copy agents
mkdir -p ~/.claude/agents
cp -r .claude/agents/* ~/.claude/agents/

# Copy rules
mkdir -p ~/.claude/rules
cp -r .claude/rules/* ~/.claude/rules/

# Verify
ls -la ~/.claude/
ls ~/.claude/agents/
ls ~/.claude/rules/universal/
```

### Step 8: Install VSCode (optional)

```bash
# Install VSCode via Homebrew
brew install --cask visual-studio-code

# Make 'code' command available
# VSCode > Command Palette > "Shell Command: Install 'code' command in PATH"

# Install extensions
while IFS= read -r extension; do
  [[ -n "$extension" && ! "$extension" =~ ^# ]] && code --install-extension "$extension"
done < packages/vscode-extensions.txt
```

### Step 9: Configure Secrets

See [`SECRETS.md`](./SECRETS.md) for instructions on configuring:
- SSH keys
- GitHub tokens
- API keys
- Company-specific credentials

### Step 10: Restart Terminal

```bash
# Restart terminal or reload shell
exec zsh
# or
source ~/.zshrc
```

---

## 🔧 Optional Configurations

### Claude Code MCP Servers

The `settings.json` includes only generic MCP servers:
- `browsermcp` - Browser automation
- `playwright` - E2E testing

To add more MCP servers, edit `~/.claude/settings.json`.

### Additional Packages

See `packages/npm-global-optional.txt` for optional packages like:
- `chrome-devtools-mcp`
- `whatsapp-mcp`
- `@anthropic-ai/claude-code`

---

## 📁 Repository Structure

```
dotfiles/
├── README.md                    # This file
├── SECRETS.md                   # Guide for sensitive configurations
├── .gitconfig                   # Git configuration (template)
├── .zshrc                       # ZSH configuration (cleaned)
├── .gitignore                   # Git ignore rules
├── .claude/
│   ├── CLAUDE.md                # Claude Code main config
│   ├── settings.json            # MCP servers & hooks
│   ├── agents/                  # 9 specialized agents
│   │   ├── accessibility-auditor.md
│   │   ├── architecture-designer.md
│   │   ├── code-reviewer.md
│   │   ├── debugger-polyglot.md
│   │   ├── frontend-specialist.md
│   │   ├── performance-optimizer.md
│   │   ├── python-expert.md
│   │   ├── task-analyzer.md
│   │   └── test-automation-pro.md
│   └── rules/
│       └── universal/           # Auto-loaded by Claude Code
│           ├── typescript-standards.md
│           ├── react-standards.md
│           ├── testing-standards.md
│           ├── performance-standards.md
│           └── teaching-agent-standards.md
└── packages/
    ├── brew-packages.txt        # Homebrew formulas
    ├── npm-global.txt           # Essential npm packages
    ├── npm-global-optional.txt  # Optional npm packages
    └── vscode-extensions.txt    # VSCode extensions
```

---

## 🚨 Security Notes

- **NO SECRETS IN THIS REPO!** All tokens/passwords have been removed
- Configure secrets manually after installation (see `SECRETS.md`)
- Never commit `.env` files or credentials
- Use environment variables for sensitive data

---

## 🔄 Updating Dotfiles

### Save current configuration

```bash
cd ~/dotfiles

# Update package lists from current system
brew list --formula > packages/brew-packages.txt
code --list-extensions > packages/vscode-extensions.txt
npm list -g --depth=0 | tail -n +2 | awk -F'@' '{print $1}' | sed 's/[├│└─ ]//g' | grep -v '^$' > packages/npm-global.txt

# Commit changes
git add -A
git commit -m "Update packages $(date +%Y-%m-%d)"
git push
```

---

## 📞 Support

For issues or questions:
1. Check `SECRETS.md` for common setup issues
2. Review Claude Code docs: https://docs.claude.com/en/docs/claude-code
3. Open an issue on GitHub

---

## 📄 License

MIT License - Feel free to use and modify for your own setup

---

## 🙏 Acknowledgments

- Inspired by various dotfiles repos in the community
- Claude Code configuration based on best practices from Anthropic
- Shell aliases and Git config accumulated over years of development
