# DevOps KPI Scoring

**Version:** 1.0  
**Owner:** DevOps Team

## Scoring Matrix

| Category | Poor (0) | Exceptional | Max Score |
|----------|----------|-------------|-----------|
| Open Jira Issues | < 3 issues | 3+ issues | 30 |
| Department Requests Handled | < 7 requests | 7+ requests | 30 |
| New Initiatives | None | 1+ initiatives | 20 |
| Extra Commitment | None | Provided | 10 |
| Punctuality & Leave | Leaves > 9 hours | Leaves ≤ 9 hours | 10 |
| **Total** | | | **100** |

## 1. Open Jira Issues

Score based on the number of Jira issues actively worked on during the period.

| Score | Criteria |
|-------|----------|
| 0 | Less than 3 open issues assigned |
| 30 | 3 or more open issues |

**Tracking:** Jira dashboard → Filter by assignee

## 2. Department Requests Handled

Score based on completed requests from other departments.

| Score | Criteria |
|-------|----------|
| 0 | Less than 7 requests handled |
| 30 | 7 or more requests handled |

**Tracking:** Service desk / ticket system → Filter by resolver

## 3. New Initiatives

Score based on contributions to new projects, automation, or improvements.

| Score | Criteria |
|-------|----------|
| 0 | No new initiatives contributed |
| 20 | 1 or more initiatives contributed |

**Examples:**
- New automation scripts or tools
- Documentation improvements
- Infrastructure optimization
- New monitoring or alerting
- Process improvements

## 4. Extra Commitment

Score based on after-hours work, on-call support, or going beyond regular duties.

| Score | Criteria |
|-------|----------|
| 0 | No extra commitment |
| 10 | Extra commitment provided |

**Examples:**
- Weekend/night deployments
- Emergency incident response
- Covering for absent team members
- Working beyond regular hours

## 5. Punctuality & Leave

Score based on attendance and leave usage.

| Score | Criteria |
|-------|----------|
| 0 | Leaves exceed 9 hours in period |
| 10 | Leaves are 9 hours or less |

**Tracking:** Leave management system → Total leave hours

## Total Score Calculation

```bash
Total = Jira Issues (0/30) + Requests (0/30) + Initiatives (0/20) + Commitment (0/10) + Punctuality (0/10)
```

| Total | Rating |
|-------|--------|
| 80-100 | Exceptional |
| 60-79 | Good |
| 40-59 | Needs Improvement |
| 0-39 | Poor |

## Automation

### Export Scores via CLI

```bash
# Generate scorecard for current month
./scripts/kpi-scorecard.sh --month 2026-05 --user username
```

### Jira Query for Issue Count

```sql
assignee = currentUser()
  AND status not in (Done, Closed, Cancelled)
  AND created >= "2026-05-01"
  AND created <= "2026-05-31"
```

## Monthly Review Process

1. **Week 1** — Team lead extracts data from Jira and ticketing system
2. **Week 2** — Self-assessment by team member
3. **Week 3** — Review meeting with team lead
4. **Week 4** — Final scores submitted

## Scorecard Template

```
DevOps KPI Scorecard — May 2026
================================
Name: _________________________

Category               Score   Max
─────────────────────────────────────
Open Jira Issues        __/30   30
Requests Handled        __/30   30
New Initiatives         __/20   20
Extra Commitment        __/10   10
Punctuality & Leave     __/10   10
─────────────────────────────────────
TOTAL                   __/100  100

Rating: ________________________

Comments:
________________________________________
________________________________________

Reviewer: ____________________
Date:    ____________________
```
