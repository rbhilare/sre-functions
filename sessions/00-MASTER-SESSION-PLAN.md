# SRE Best Practices — How We Do Things

**Platform:** OpenShift Container Platform (OCP)
**Total Sessions:** 14
**Total Duration:** ~7.5 hours across 14 sessions

---

## What This Is (and What It Is NOT)

| This IS | This is NOT |
|---------|-------------|
| Sharing our processes and practices | Teaching OCP architecture |
| "Here's how we handle upgrades" | "Here's what an operator is" |
| Real examples from our team | Textbook SRE theory |
| What works for us (adapt to your context) | A prescription you must follow |
| 30-minute focused conversations | 90-minute lectures |

**Assumption:** The audience already knows OpenShift. We skip "what is a pod" and go straight to "here's how we run our pods reliably."

---

## Session Overview

| Session | Title | Duration | Type |
|:-------:|-------|:--------:|:----:|
| 1 | [Reliability Across Layers — Code, Config, Platform](01-RELIABILITY-ACROSS-LAYERS.md) | 30 min | Content |
| 2 | [Follow-the-Sun (FTS) Model for SRE Environment](02-FOLLOW-THE-SUN-MODEL.md) | 30 min | Content |
| 3 | [Inside SRE: Ask & Discuss Anything](03-INSIDE-SRE-ASK-AND-DISCUSS.md) | 30 min | Q&A |
| 4 | [How We Run SRE](04-HOW-WE-RUN-SRE.md) | 40 min | Content |
| 5 | How We Operate OCP Clusters | 40 min | Content |
| 6 | Inside SRE: Ask & Discuss Anything | 30 min | Q&A |
| 7 | How We Monitor and Alert | 30 min | Content |
| 8 | How We Handle Incidents | 30 min | Content |
| 9 | Inside SRE: Ask & Discuss Anything | 30 min | Q&A |
| 10 | How We Measure Reliability | 30 min | Content |
| 11 | How We Manage Changes and Upgrades | 30 min | Content |
| 12 | Inside SRE: Ask & Discuss Anything | 30 min | Q&A |
| 13 | How We Automate and Reduce Toil | 30 min | Content |
| 14 | How We Plan for Capacity and DR | 30 min | Content |

---

## Session Structure

The series alternates between **content sessions** and **Q&A sessions** in blocks of two:

```
[Content] → [Content] → [Q&A] → [Content] → [Content] → [Q&A] → ...
```

**Content sessions** share specific SRE practices, processes, and lessons learned — told from a first-person "here's what we do" perspective, not generic SRE theory.

**Q&A sessions (Inside SRE)** are open discussion forums where the partner team asks questions, shares their own experiences, and discusses how to adapt the practices to their environment. These are placed after every two content sessions to allow for deeper exploration.

---

## Session Details

### Session 1: [Reliability Across Layers — Code, Config, Platform](01-RELIABILITY-ACROSS-LAYERS.md) (30 min)

How we think about reliability as a shared responsibility across the application code, configuration management, and platform layers — and where failures actually propagate.

---

### Session 2: [Follow-the-Sun (FTS) Model for SRE Environment](02-FOLLOW-THE-SUN-MODEL.md) (30 min)

How we implement Follow-the-Sun for 24/7 coverage without overnight shifts — regional team structure, handoff process, and ServiceNow/Teams integration for cross-region transitions.

---

### Session 3: [Inside SRE — Ask & Discuss Anything](03-INSIDE-SRE-ASK-AND-DISCUSS.md) (30 min)

Open Q&A covering Sessions 1–2. Discussion prompts around reliability layers and FTS models.

---

### Session 4: [How We Run SRE](04-HOW-WE-RUN-SRE.md) (40 min)

Team structure, RACI, on-call rotations, blameless culture, working principles ("ticket first, call second"), and anti-patterns we've eliminated.

---

### Session 5: How We Operate OCP Clusters (40 min)

Daily/weekly/monthly operational routines, node and MachineConfigPool management, CNV operations, certificate lifecycle, and mirror registry management in disconnected environments.

---

### Session 6: Inside SRE — Ask & Discuss Anything (30 min)

Open Q&A covering Sessions 4–5. Discussion prompts around SRE team structure and OCP operations.

---

### Session 7: How We Monitor and Alert (30 min)

Monitoring stack (Prometheus, Alertmanager, Loki), alert philosophy ("alert on actionables"), routing by severity, dashboards, alert fatigue elimination (how we went from 500 to 80 alerts), and our top 10 alert rules.

---

### Session 8: How We Handle Incidents (30 min)

Severity classification, 6-phase incident workflow, ServiceNow and Teams integration, status board, escalation paths, and post-incident reviews (PIRs).

---

### Session 9: Inside SRE — Ask & Discuss Anything (30 min)

Open Q&A covering Sessions 7–8. Discussion prompts around monitoring, alert fatigue, and incident management.

---

### Session 10: How We Measure Reliability (30 min)

Our 5 actual SLIs (with PromQL), SLO targets and burn rates, error budget policy, decision-making with SLOs, and mistakes we've made.

---

### Session 11: How We Manage Changes and Upgrades (30 min)

Change categorization, OCP upgrade process, pre/post-upgrade checklists, operator upgrades, rollback and recovery, and disconnected environment considerations.

---

### Session 12: Inside SRE — Ask & Discuss Anything (30 min)

Open Q&A covering Sessions 10–11. Discussion prompts around SLOs, error budgets, and change management.

---

### Session 13: How We Automate and Reduce Toil (30 min)

Toil identification and measurement, automation ROI framework, GitOps (ArgoCD) repo structure, self-healing patterns, and what we've learned not to automate.

---

### Session 14: How We Plan for Capacity and DR (30 min)

Backup strategy, DR runbooks and testing, capacity planning, monthly metrics review, SRE flywheel for continuous improvement, and series wrap-up with key takeaways.

---

## Delivery Guidelines

- **Tone:** First-person experience sharing — "here's what we do," not "here's what you should do"
- **Pace:** One to two sessions per week allows time for the partner team to absorb and practice
- **Q&A sessions:** Let the partner team lead — facilitator listens more than talks
- **Adaptation:** Use Q&A feedback to adjust depth of upcoming content sessions
- **Final wrap-up:** Session 14 includes a series summary and recommended next steps for the partner team
