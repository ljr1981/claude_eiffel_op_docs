# G01: Git Repository Initialization

## Objective
Initialize git repository and push to Simple Eiffel GitHub organization.

## Prerequisites
- Library directory exists at /d/prod/{lib_name}
- Code is complete and tested
- GitHub CLI (gh) is authenticated

## Steps

### 1. Check Current Git Status
```bash
cd /d/prod/{lib_name}
git status 2>/dev/null || echo "Not a git repo"
```

### 2. Initialize if Needed
```bash
# Only if not already a git repo:
git init
```

### 3. Create .gitignore
```
# Eiffel compilation output
EIFGENs/

# IDE files
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Backup files
*.bak
*.orig
```

### 4. Stage All Files
```bash
git add -A
git status
```

### 5. Create Initial Commit (if first commit)
```bash
git commit -m "Initial commit: {lib_name} library

Simple Eiffel ecosystem library for {description}.

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

### 6. Create GitHub Repository
```bash
# Check if repo exists
gh repo view simple-eiffel/{lib_name} 2>/dev/null

# If not, create it:
gh repo create simple-eiffel/{lib_name} --public --description "{one-line description}"
```

### 7. Add Remote and Push
```bash
git remote add origin https://github.com/simple-eiffel/{lib_name}.git 2>/dev/null || true
git branch -M main
git push -u origin main
```

## Verification
- [ ] Repository exists at github.com/simple-eiffel/{lib_name}
- [ ] All source files pushed
- [ ] .gitignore in place
- [ ] No EIFGENs/ or build artifacts committed

## Output
```
Repository: https://github.com/simple-eiffel/{lib_name}
Status: Pushed
```

## Next Step
→ G02-README-STANDARD.md
