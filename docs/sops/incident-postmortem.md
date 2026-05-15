# Incident Postmortem SOP

**Version:** 1.0  
**Owner:** DevOps Team

## Purpose

Standardize the postmortem process to learn from incidents and prevent recurrence.

## When to Write a Postmortem

- Any SEV1 or SEV2 incident
- Any incident with customer impact > 5 minutes
- Recurring incidents (same issue 3+ times)
- Security incidents

## Postmortem Timeline

| Phase | Timeframe | Activity |
|-------|-----------|----------|
| Incident resolved | T+0 | Gather raw data, logs, timelines |
| Initial draft | T+24h | Write postmortem document |
| Review | T+48h | Team review and feedback |
| Finalize | T+72h | Publish and track action items |

## Postmortem Template

```markdown
# Postmortem: [Title]

**Date:** YYYY-MM-DD  
**Severity:** SEV1 / SEV2 / SEV3  
**Duration:** HH:MM (detection to resolution)  
**Impact:** [Number] users affected, [X] minutes downtime

## Summary

One paragraph summary of what happened.

## Timeline

| Time (UTC) | Event |
|------------|-------|
| 14:00 | Alert fired: 5xx errors > 5% |
| 14:02 | On-call acknowledged |
| 14:05 | Root cause identified: bad deploy |
| 14:10 | Rollback initiated |
| 14:15 | Service restored |
| 14:30 | All systems healthy |

## Root Cause

What was the underlying cause?

## Detection

How was this detected? Alert, customer report, manual check?

## Resolution

What was done to fix it?

## What Went Well

- Fast detection
- Good team communication
- Rollback worked as expected

## What Went Wrong

- No canary deployment
- Missing monitoring on new endpoint
- Alert fatigue delayed response

## Action Items

| # | Action | Owner | Due Date | Status |
|---|--------|-------|----------|--------|
| 1 | Add canary deployment to pipeline | DevOps | 2024-02-01 | Open |
| 2 | Add alert for new endpoint | Alice | 2024-01-25 | Open |
| 3 | Review alert thresholds | Bob | 2024-01-30 | Open |

## Lessons Learned

What should the team remember for next time?
```

## Distribution

- Post to `#postmortems` Slack channel
- Email to engineering team
- Add to postmortem archive
- Tag relevant managers for action items

## Action Item Tracking

- Each action item must have an owner and due date
- Track in project management tool (GitHub Issues / Jira)
- Weekly review of open action items
- Escalate overdue items to team lead

## Blameless Culture

!!! important
    Postmortems are blameless. The goal is to fix systems, not assign fault. Focus on process improvements, not individual mistakes.

## Archive

Store all postmortems in `docs/postmortems/YYYY/` for future reference.
