<div class="hero" markdown>
<div class="hero-badge">Red Hat SRE &bull; Hybrid Platforms</div>

# SRE Best Practices — How We Do Things

Sharing real-world Site Reliability Engineering practices from running
OpenShift Container Platform at scale

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

Team structure, RACI, on-call, blameless culture, and the principles that keep us effective.
</div>

<div class="card" markdown>
<span class="card-icon">:material-server-network:</span>

### How We Operate OCP

Daily routines, node management, CNV operations, certificates, and disconnected environment specifics.
</div>

<div class="card" markdown>
<span class="card-icon">:material-chart-bell-curve-cumulative:</span>

### How We Monitor & Alert

Alert philosophy, routing, dashboards, and how we went from 500 to 80 alerts.
</div>

<div class="card" markdown>
<span class="card-icon">:material-fire-alert:</span>

### How We Handle Incidents

6-phase workflow, ServiceNow + Teams integration, escalation, and post-incident reviews.
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
| **10** | [How We Measure Reliability](sessions/10-HOW-WE-MEASURE-RELIABILITY.md) | 30 min | :material-presentation: Content |
| **11** | [How We Manage Changes and Upgrades](sessions/11-HOW-WE-MANAGE-CHANGES-AND-UPGRADES.md) | 30 min | :material-presentation: Content |
| **12** | [Inside SRE: Ask & Discuss Anything](sessions/12-INSIDE-SRE-ASK-AND-DISCUSS.md) | 30 min | :material-forum: Q&A |
| **13** | [How We Automate and Reduce Toil](sessions/13-HOW-WE-AUTOMATE-AND-REDUCE-TOIL.md) | 30 min | :material-presentation: Content |
| **14** | [How We Plan for Capacity and DR](sessions/14-HOW-WE-PLAN-FOR-CAPACITY-AND-DR.md) | 30 min | :material-presentation: Content |

---

## Delivery Approach

!!! tip "How we deliver these sessions"

    - **Tone:** First-person experience sharing — "here's what we do," not "here's what you should do"
    - **Pace:** One to two sessions per week for absorption and practice
    - **Q&A sessions:** Partner team leads — facilitator listens more than talks
    - **Adaptation:** Q&A feedback shapes the depth of upcoming content sessions

    ```
    [Content] → [Content] → [Q&A] → [Content] → [Content] → [Q&A] → ...
    ```

---

## Presenter

:material-account-circle:{ .lg } **Rohit Bhilare** — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat

[:material-email: rbhilare@redhat.com](mailto:rbhilare@redhat.com) &nbsp;&nbsp; [:material-slack: @Rohit Bhilare](https://redhat.enterprise.slack.com/team/U0875U18FUH) &nbsp;&nbsp; [:material-github: GitHub](https://github.com/rbhilare/sre-functions)
