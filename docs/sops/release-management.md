# Release Management

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Standardize software releases with consistent versioning, changelogs, and release notes.

## Versioning (SemVer)

```
MAJOR.MINOR.PATCH

MAJOR — Breaking changes (API incompatible)
MINOR — New features (backward compatible)
PATCH — Bug fixes (backward compatible)
```

Pre-release: `1.0.0-alpha`, `1.0.0-beta`, `1.0.0-rc.1`

## Release Branches

```bash
git checkout develop
git checkout -b release/v1.2.0

# Bump version, update changelog
# Commit changes
git commit -m "chore: bump version to v1.2.0"

# Open PR to main
# After approval, merge
git checkout main
git merge release/v1.2.0
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge main
```

## Changelog Format

Keep a `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/):

```markdown
# Changelog

## [1.2.0] - 2024-01-15

### Added
- New user dashboard
- OAuth2 login support

### Changed
- Improved query performance (40% faster)
- Updated dependencies

### Fixed
- Fixed crash on empty search results
- Resolved memory leak in background jobs

## [1.1.0] - 2024-01-01

### Added
- Email notification system
```

## Release Checklist

### Pre-Release (T-7 days)

- [ ] Feature complete — no new features after code freeze
- [ ] All PRs targeting release merged
- [ ] Version bumped in codebase

### Testing (T-3 days)

- [ ] All tests pass (unit + integration)
- [ ] Staging deployment successful
- [ ] QA sign-off obtained
- [ ] Performance benchmarks acceptable

### Release Day

- [ ] Final approval from lead
- [ ] CHANGELOG.md updated
- [ ] Tag created
- [ ] Deployment to production
- [ ] Smoke tests pass
- [ ] Monitoring checked (no new alerts)

### Post-Release (T+1 day)

- [ ] Release notes published
- [ ] Customers notified (if applicable)
- [ ] Rollback plan still valid
- [ ] Retrospective scheduled (if needed)

## Release Notes Template

```markdown
# Release v1.2.0

**Release Date:** January 15, 2024

## Highlights
- New user dashboard with real-time analytics
- 40% faster query performance

## What's New
- [Feature] OAuth2 login — users can now login with Google/GitHub
- [Feature] Email notifications for daily reports
- [Enhancement] Optimized database queries

## Bug Fixes
- Fixed crash when searching with special characters
- Fixed memory leak in report generation
- Fixed incorrect timezone in logs

## Breaking Changes
None

## Upgrade Notes
- Run database migrations: `php artisan migrate`
- Clear cache: `wp cache flush`
```

## Rollback Plan

```bash
# Git rollback
git revert <release-tag>
git push origin main

# Docker rollback
docker compose down
# Revert docker-compose.yml to previous version
docker compose up -d

# Database rollback
# Run down migrations
php artisan migrate:rollback
```
