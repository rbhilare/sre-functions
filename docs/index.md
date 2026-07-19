<div class="hero" markdown>
<div class="hero-badge">Hybrid Platforms — ROSA, Red Hat</div>

# SRE Best Practices — How We Do Things

Sharing Site Reliability Engineering practices from running
OpenShift Container Platform

</div>

<div class="stats" markdown>
<div class="stat">
<span class="stat-number">14</span>
<span class="stat-label">Sessions</span>
</div>
<div class="stat">
<span class="stat-number">7.5</span>
<span class="stat-label">Total Hours</span>
</div>
<div class="stat">
<span class="stat-number">10</span>
<span class="stat-label">Content Topics</span>
</div>
<div class="stat">
<span class="stat-number">4</span>
<span class="stat-label">Q&A Sessions</span>
</div>
</div>

---

## What This Is (and What It Is NOT)

!!! info ""

    | :material-check-bold:{ .green } **This IS** | :material-close-thick:{ .red } **This is NOT** |
    |-------------|-----------------|
    | Sharing our processes and practices | Teaching OCP architecture |
    | "Here's how we handle upgrades" | "Here's what an operator is" |
    | Real examples from our team | Textbook SRE theory |
    | What works for us (adapt to your context) | A prescription you must follow |
    | 30-minute focused conversations | 90-minute lectures |

    **Assumption:** The audience already knows OpenShift. We skip "what is a pod" and go straight to "here's how we run our pods reliably."

<div class="section-divider"><span>Session Topics</span></div>

<div class="grid-cards" markdown>

<div class="card" markdown>
<span class="card-icon">:material-layers-triple:</span>

### Reliability Across Layers

Code, Config, and Platform — how reliability is a shared responsibility across every layer of the stack.
</div>

<div class="card" markdown>
<span class="card-icon">:material-earth:</span>

### Follow-the-Sun Model

24/7 coverage without overnight shifts — regional team structure, handoff process, and tooling.
</div>

<div class="card" markdown>
<span class="card-icon">:material-account-group:</span>

### How We Run SRE

Team structure, culture principles, dual operating model, on-call with automation, and blameless culture.
</div>

<div class="card" markdown>
<span class="card-icon">:material-server-network:</span>

### How We Operate OCP

Cluster access, lifecycle operations, node maintenance workflow, upgrade monitoring, and diagnostics.
</div>

<div class="card" markdown>
<span class="card-icon">:material-chart-bell-curve-cumulative:</span>

### How We Monitor & Alert

Five-level severity model, SLO burn-rate alerting, PagerDuty + Slack routing, Dead Man's Snitch, and runbook discipline across 2,700+ alerts.
</div>

<div class="card" markdown>
<span class="card-icon">:material-fire-alert:</span>

### How We Handle Incidents

Four-phase incident lifecycle, three core roles, severity-driven check-ins, customer communication, handover, and blameless post-mortem reviews.
</div>

<div class="card" markdown>
<span class="card-icon">:material-target:</span>

### How We Measure Reliability

SLIs, SLOs, error budgets — real targets with real PromQL, not abstract theory.
</div>

<div class="card" markdown>
<span class="card-icon">:material-arrow-up-bold-circle:</span>

### How We Manage Changes

Change categorization, OCP upgrades, pre/post checklists, rollback, and disconnected constraints.
</div>

<div class="card" markdown>
<span class="card-icon">:material-robot:</span>

### How We Automate

Toil identification, automation ROI, GitOps with ArgoCD, self-healing, and what NOT to automate.
</div>

<div class="card" markdown>
<span class="card-icon">:material-shield-check:</span>

### Capacity, DR & Improvement

Backup strategy, DR testing, capacity planning, and the SRE flywheel for continuous improvement.
</div>

</div>

<div class="section-divider"><span>Session Schedule</span></div>

## Full Session Schedule

| Session | Title | Duration | Type |
|:-------:|-------|:--------:|:----:|
| **1** | [Reliability Across Layers — Code, Config, Platform](sessions/01-RELIABILITY-ACROSS-LAYERS.md) | 30 min | :material-presentation: Content |
| **2** | [Follow-the-Sun (FTS) Model for SRE Environment](sessions/02-FOLLOW-THE-SUN-MODEL.md) | 30 min | :material-presentation: Content |
| **3** | [Inside SRE: Ask & Discuss Anything](sessions/03-INSIDE-SRE-ASK-AND-DISCUSS.md) | 30 min | :material-forum: Q&A |
| **4** | [How We Run SRE](sessions/04-HOW-WE-RUN-SRE.md) | 40 min | :material-presentation: Content |
| **5** | [How We Operate OCP Clusters](sessions/05-HOW-WE-OPERATE-OCP-CLUSTERS.md) | 40 min | :material-presentation: Content |
| **6** | [Inside SRE: Ask & Discuss Anything](sessions/06-INSIDE-SRE-ASK-AND-DISCUSS.md) | 30 min | :material-forum: Q&A |
| **7** | [How We Monitor and Alert](sessions/07-HOW-WE-MONITOR-AND-ALERT.md) | 30 min | :material-presentation: Content |
| **8** | [How We Handle Incidents](sessions/08-HOW-WE-HANDLE-INCIDENTS.md) | 30 min | :material-presentation: Content |
| **9** | [Inside SRE: Ask & Discuss Anything](sessions/09-INSIDE-SRE-ASK-AND-DISCUSS.md) | 30 min | :material-forum: Q&A |
| **10** | How We Measure Reliability | 30 min | :material-presentation: Content |
| **11** | How We Manage Changes and Upgrades | 30 min | :material-presentation: Content |
| **12** | Inside SRE: Ask & Discuss Anything | 30 min | :material-forum: Q&A |
| **13** | How We Automate and Reduce Toil | 30 min | :material-presentation: Content |
| **14** | How We Plan for Capacity and DR | 30 min | :material-presentation: Content |

---

## Presenter

:material-account-circle:{ .lg } **Rohit Bhilare** — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat

[:material-email: rbhilare@redhat.com](mailto:rbhilare@redhat.com) &nbsp;&nbsp; [:material-github: GitHub](https://github.com/rbhilare/sre-functions)
