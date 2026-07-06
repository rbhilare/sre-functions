# Session 2: Follow-the-Sun (FTS) Model for SRE Environment

**Duration:** 30 minutes
**Series:** SRE Best Practices — How We Do Things (Session 2 of 14)

---

## Overview

Three regional SRE teams provide 24/7 coverage by handing off active investigations as the sun moves across time zones. No single team works overnight.

```
APAC (UTC+5:30 to +9)      EMEA (UTC+0 to +2)        NASA (UTC-5 to -8)
06:00–14:00 UTC             14:00–22:00 UTC            22:00–06:00 UTC
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
       │    Handoff 1    │       │    Handoff 2    │       │    Handoff 3
       └────────────────▶│       └────────────────▶│       └────────────▶
```

---

## Quick Steps

### 1. Issue Detected

- Alert fires or user reports an issue
- ServiceNow/WebRCA incident created (auto or manual)
- Assigned to the **currently active regional team**

### 2. Investigate and Document

- Active team investigates following standard runbooks
- **All findings posted in the shared Teams/Slack investigation thread/channel**
- ServiceNow/WebRCA work notes updated every 30 min (Sev-1/2) or 2 hrs (Sev-3/4)

### 3. Handoff Preparation (30 min before shift end)

The outgoing team prepares a structured handoff note in the Teams/Slack thread/channel:

```
--- HANDOFF: [APAC → EMEA] — [Date, Time UTC] ---
Incident:      SNOW-INC00412/ITN-2026-000010
Severity:      Sev-2
Cluster:       prod-spoke-01
Status:        Investigating / Mitigated / Blocked

Summary:       etcd leader elections every ~30s, traced to high disk I/O on master-2
What we tried: Checked etcd logs, confirmed leader churn, ruled out network
Current theory: Failing SSD on master-2
Next steps:    Run disk diagnostics, consider draining master-2
Blockers:      None / Waiting for [X]
Escalations:   Case #12345 open (awaiting response)
Contacts:      @JaneDoe available on Teams for 1 hr after handoff if needed
```

### 4. Handoff Meeting (15 min overlap)

- **Outgoing team** briefs **incoming team** (live call or recorded async)
- Walk through every open incident using the handoff note
- Incoming team asks clarifying questions
- Incoming team **acknowledges ownership** in the Teams/Slack thread/channel

### 5. Incoming Team Takes Over

- Update ServiceNow/WebRCA: reassign to incoming team / engineer
- Post acknowledgement in the investigation thread/channel:

```
--- ACKNOWLEDGED: EMEA taking over — [Date, Time UTC] ---
Received handoff from APAC. Continuing investigation on INC00412/ITN-2026-000010.
Next update in 30 min.
```

- Continue investigation from where outgoing team left off

### 6. Resolve or Hand Off Again

- If resolved: update ServiceNow/WebRCA → Resolved, post resolution in thread/channel, notify stakeholders
- If not resolved by shift end: repeat Steps 3–5 to next region

---

## Handoff Rules

| Rule | Detail |
|------|--------|
| Never drop an incident silently | Every open incident must be explicitly handed off |
| 15-min overlap is mandatory | Outgoing and incoming teams must have a live sync |
| Handoff note is non-negotiable | No verbal-only handoffs — it must be written in the thread/channel |
| Outgoing team stays available 1 hr | For follow-up questions after handoff |
| Severity reassessment at each handoff | Incoming team re-evaluates severity with fresh eyes |
| Escalations carry forward | Cases, management notifications — include in handoff |

---

## Escalation Across Regions

| Scenario | Action |
|----------|--------|
| Sev-1 during handoff | Both teams stay until mitigated — handoff only after stabilization |
| Sev-1 overnight (no active team available) | Page the on-call engineer in the nearest active region |
| Need SME from another region | @mention in the thread/channel — they respond when their shift starts |
| Case needs follow-up | Include case number, last update, and expected response time in handoff |

---

## Quick Reference

```
ISSUE DETECTED
     │
     ▼
ACTIVE TEAM INVESTIGATES
(document everything in Teams/Slack thread/channel + ServiceNow/WebRCA)
     │
     ├── Resolved? ──▶ YES ──▶ Close incident, update thread
     │
     NO (shift ending)
     │
     ▼
PREPARE HANDOFF NOTE (30 min before shift end)
     │
     ▼
HANDOFF CALL (15 min overlap)
     │
     ▼
INCOMING TEAM ACKNOWLEDGES & CONTINUES
     │
     ├── Resolved? ──▶ YES ──▶ Close incident, update thread
     │
     NO (shift ending)
     │
     ▼
REPEAT HANDOFF TO NEXT REGION
```
