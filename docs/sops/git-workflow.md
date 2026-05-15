---
tags:
  - git
  - workflow
  - branching
  - code-review
---

# Git Workflow SOP

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Standardize branching strategy, commit practices, and code review process across all projects.

## Branching Model

```
main          ───────●───────────────────●── (production)
                      \                 /
feature/xxx   ────────\───●──●──●───────/
                        \             /
release/x.x   ──────────\───●──────●──/
```

| Branch | Purpose | Base Branch |
|--------|---------|-------------|
| `main` | Production-ready code | — |
| `develop` | Integration branch | `main` |
| `feature/*` | New features | `develop` |
| `fix/*` | Bug fixes | `develop` |
| `release/*` | Release preparation | `develop` |
| `hotfix/*` | Urgent production fixes | `main` |

## Branch Naming

```
feature/short-description
fix/issue-description
hotfix/critical-fix-description
release/v1.2.3
```

## Commit Convention

Use conventional commits:

```
<type>(<scope>): <description>

[optional body]
```

**Types:** `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `test`, `ci`

**Examples:**

```
feat(auth): add OAuth2 login
fix(api): handle null response in user endpoint
docs(readme): update installation instructions
```

## Workflow

### Starting a Feature

```bash
git checkout develop
git pull origin develop
git checkout -b feature/my-feature
```

### Committing Changes

```bash
git add .
git commit -m "feat: add user registration form"
git push origin feature/my-feature
```

### Keeping Feature Branch Updated

```bash
git checkout develop
git pull origin develop
git checkout feature/my-feature
git rebase develop
# or git merge develop
```

### Creating a Pull Request

1. Push branch to remote
2. Open PR on GitHub targeting `develop`
3. Fill PR template with description
4. Request review from at least 1 team member

### Code Review Checklist

- [ ] Code follows project style guide
- [ ] No debug/console statements left
- [ ] Error handling is appropriate
- [ ] No hardcoded secrets
- [ ] Tests pass
- [ ] Documentation updated (if needed)

### Merging

```bash
# Squash and merge for features
# Regular merge for release branches

# After PR is merged, delete branch
git branch -d feature/my-feature
git push origin --delete feature/my-feature
```

## Hotfix Process

```bash
git checkout main
git checkout -b hotfix/urgent-fix
# Make fix, commit
git push origin hotfix/urgent-fix
# Open PR targeting main
# After merge, cherry-pick to develop
git checkout develop
git cherry-pick <commit-hash>
git push origin develop
```

## Tags & Releases

```bash
# Create annotated tag
git tag -a v1.2.3 -m "Release v1.2.3"
git push origin v1.2.3
```
