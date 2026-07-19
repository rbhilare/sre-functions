# Session 9: Inside SRE — Ask & Discuss Anything

**Duration:** 30 minutes | **Format:** Open Q&A and discussion
**Series:** SRE Best Practices — How We Do Things (Session 9 of 14)
**Covers:** Sessions 7–8 follow-up

---

## Purpose

Open forum for the partner team to ask questions, discuss ideas, and share their own experiences related to topics covered in Sessions 7–8:

- Session 7: How We Monitor and Alert
- Session 8: How We Handle Incidents

---

## Format

| Time | Activity |
|------|----------|
| 5 min | Quick recap of Sessions 7–8 key points |
| 15 min | Open Q&A — anything from the previous sessions |
| 5 min | Discussion — "How does your team handle this?" |
| 5 min | Action items and preview of Sessions 10–11 |

---

## Quick Recap — Session 7: How We Monitor and Alert

- Five-level severity model: critical, critical-FTS, high, medium, info — each with a defined response path
- Every alert requires two mandatory labels: `service` and `severity` — no exceptions
- SLO-based multi-window, multi-burn-rate alerting (14.4×, 6×, 3×, 1× thresholds)
- Alert routing: critical → PagerDuty, high → PagerDuty + Slack, medium/info → Slack only
- Dead Man's Snitch via the Watchdog alert — monitors the monitoring system itself
- 89% runbook coverage across 2,700+ alerts; alert graduation pipeline (info → medium → high → critical)
- ~80% alert noise reduction through inhibition rules, severity tuning, and the graduation pipeline

## Quick Recap — Session 8: How We Handle Incidents

- A PagerDuty alert is NOT an incident — most alerts resolve via runbook; incidents require coordination
- Four-phase lifecycle: Identify → Create → Resolve → Post-Incident
- Three roles: Tech Lead (technical lead), Incident Owner (customer perspective + post-incident accountability), Scribe (records everything)
- Severity check-ins: Sev1 every 20 min, Sev2 every 30 min, Sev3 every 60 min — non-negotiable
- Customer communication determined by impact scope (fleet-wide → status page; single cluster → service log)
- Structured handover: executive summary + 15-min verbal overlap + clean disengagement within 30 min
- Blameless PMR within 5 days for Sev1/2; corrective actions tracked and reviewed

---

## Discussion Prompts

### Monitoring and Alerting (Session 7)

- How many severity levels does your alerting model use? Do you have a defined response path for each?
- How many of your alerts have runbooks? What happens when someone gets paged without one?
- Do you use SLO-based burn-rate alerting, or raw threshold alerts?
- Do you have a dead man's switch for your monitoring stack? How do you know when monitoring itself is down?
- How often do you review and prune your alert rules? Do you have an alert graduation pipeline?

### Incident Management (Session 8)

- Do you distinguish between "alert" and "incident"? Where do you draw the line?
- How do you declare an incident? Is there a formal process, or is it ad-hoc?
- How many incident roles does your process define? How many actually get filled during a real incident?
- Do you have defined check-in intervals during incidents, or do updates happen when someone remembers?
- How do you hand off an incident across shifts or regions? Is it structured or informal?
- Do you run blameless post-mortem reviews? What happens with the corrective actions afterward?
- Does your team feel empowered to escalate early, or do engineers hesitate because they feel like they should solve it themselves?

---

## Preview: Sessions 10–11

- **Session 10: How We Measure Reliability** — our actual SLIs (with PromQL), SLO targets and burn rates, error budget policy, decision-making with SLOs, and mistakes we've made.
- **Session 11: How We Manage Changes and Upgrades** — change categorization, OCP upgrade process, pre/post-upgrade checklists, operator upgrades, rollback and recovery.
