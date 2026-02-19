# GitHub Actions Learning Notes

## 📚 What I Learned

### Core Concepts

**GitHub Actions** is a CI/CD platform that allows you to automate your build, test, and deployment pipeline.

#### Key Components:

1. **Workflow** - A configurable automated process defined in a YAML file
   - Location: `.github/workflows/your-workflow.yml`
   - Contains one or more jobs

2. **Event** - Specific activity that triggers a workflow
   - Examples: `push`, `pull_request`, `workflow_dispatch`, `schedule`

3. **Runner** - A server that runs your workflows
   - GitHub-hosted: `ubuntu-latest`, `windows-latest`, `macos-latest`
   - Self-hosted: Your own machines

4. **Job** - A set of steps that execute on the same runner
   - Jobs run in parallel by default
   - Can be configured to run sequentially with `needs`

5. **Step** - Individual task within a job
   - Can run commands or actions
   - Share the same runner and filesystem

---

## 🔐 Authentication Challenges & Solutions

### Two Ways to Authenticate with GitHub:

#### **Option 1: GitHub CLI (Recommended - Easiest)**
```bash
# Login via GitHub CLI
gh auth login

# Follow the prompts:
# 1. Choose: GitHub.com
# 2. Choose: HTTPS
# 3. Choose: Yes (Authenticate Git with credentials)
# 4. Choose: Login with a web browser
# 5. Copy the one-time code
# 6. Open: https://github.com/login/device
# 7. Paste code and authorize
# 8. Press Enter in terminal

# Verify authentication
gh auth status

# Logout when needed
gh auth logout
```

#### **Option 2: Personal Access Token (PAT)**
```bash
# 1. Create token at: https://github.com/settings/tokens/new
# 2. Select scopes:
#    ✅ repo (full control)
#    ✅ workflow (update workflows)
# 3. Generate and copy the token immediately

# Configure Git to use the token
git remote set-url origin https://YOUR_TOKEN@github.com/username/repo.git

# Or use Git credential helper
git config --global credential.helper store
# Then Git will prompt for username and token on first push
```

**Token Scopes Needed:**
- `repo` - Full control of private repositories (includes push/pull)
- `workflow` - Update GitHub Action workflows
- `read:org` - Read organization data (if working with org repos)

---

## 🛠️ Useful Commands

### Git Workflow Commands
```bash
# Check status
git status

# Add files
git add <file>
git add .                    # Add all changes

# Commit changes
git commit -m "Your message"

# Push to GitHub
git push origin main
git push -u origin main      # First push, set upstream

# Pull latest changes
git pull origin main

# Check commit history
git log --oneline

# Check remote URL
git remote -v
```

### GitHub CLI Commands
```bash
# Authentication
gh auth login
gh auth status
gh auth logout

# Repository operations
gh repo create <name> --public --source=. --remote=origin --push
gh repo view
gh repo list

# Workflow operations
gh workflow list             # List all workflows
gh workflow view             # View workflow details
gh workflow run <name>       # Manually trigger workflow
gh workflow enable <name>    # Enable a workflow
gh workflow disable <name>   # Disable a workflow

# Run operations
gh run list                  # List workflow runs
gh run view                  # View latest run details
gh run view <run-id>         # View specific run
gh run watch                 # Watch a run in real-time
gh run rerun <run-id>        # Re-run a workflow
gh run cancel <run-id>       # Cancel a running workflow

# Actions logs
gh run view --log            # View logs of latest run
gh run view <run-id> --log   # View logs of specific run
```

### Project Setup Commands
```bash
# Create project structure
mkdir -p hello-world-python/src
mkdir -p hello-world-python/.github/workflows
cd hello-world-python

# Initialize Git
git init
git config user.email "your.email@example.com"
git config user.name "Your Name"

# Create files
touch src/main.py
touch requirements.txt
touch README.md
touch .github/workflows/run-hello-world.yml

# First commit
git add .
git commit -m "Initial commit"

# Connect to GitHub (after creating repo)
git remote add origin https://github.com/username/repo.git
git branch -M main
git push -u origin main
```

---

## 📝 Sample Workflow File

```yaml
name: Hello World Python Workflow

# Events that trigger the workflow
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:  # Manual trigger

# Jobs to run
jobs:
  run-hello-world:
    # Runner environment
    runs-on: ubuntu-latest
    
    # Steps to execute
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.11'
        
    - name: Display Python version
      run: python --version
        
    - name: Run Hello World script
      run: python src/main.py
```

---

## 🎯 Workflow Triggers (Events)

### Common Events:

```yaml
# On push to specific branches
on:
  push:
    branches: [ main, develop ]
    paths:
      - 'src/**'  # Only trigger if files in src/ change

# On pull request
on:
  pull_request:
    branches: [ main ]
    types: [ opened, synchronize, reopened ]

# Manual trigger
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'

# Scheduled (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC

# Multiple events
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
```

---

## 🚀 Best Practices

1. **Use specific action versions**: `actions/checkout@v4` instead of `@main`
2. **Cache dependencies**: Speed up workflows with caching
3. **Use secrets**: Never hardcode credentials in workflows
4. **Add status badges**: Show workflow status in README
5. **Use matrix strategies**: Test across multiple versions/platforms
6. **Set timeouts**: Prevent workflows from running indefinitely
7. **Use concurrency**: Cancel old runs when new ones start

### Example with Best Practices:
```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    
    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v5
      with:
        python-version: ${{ matrix.python-version }}
        cache: 'pip'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Run tests
      run: pytest
```

---

## 🔍 Troubleshooting

### Common Issues:

**1. Workflow not triggering**
- Check: Actions enabled in repo settings
- Verify: Event configuration in workflow file
- Wait: Can take 5-30 seconds to start

**2. Authentication failed**
- Run: `gh auth status` to check
- Re-authenticate: `gh auth login`
- Check token scopes if using PAT

**3. Permission denied**
- Verify: Token has `repo` scope
- Check: Repository permissions in settings
- Ensure: Correct remote URL

**4. Workflow fails**
- Click: On failed run in Actions tab
- Review: Error logs
- Check: Runner logs for details

---

## 📖 Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [Awesome Actions](https://github.com/sdras/awesome-actions)

---

**Created**: February 19, 2026  
**Repository**: https://github.com/redhatpeter/github-action-hello-world
