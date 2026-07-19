# Session 7: How We Monitor and Alert

**Our Monitoring Stack, Alert Philosophy, and How We Eliminated Alert Fatigue**

**Presenter:** Rohit Bhilare — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat
**Duration:** 30 minutes | **Format:** Peer conversation, not a lecture
**Series:** SRE Best Practices — How We Do Things (Session 7 of 14)
**Focus:** Monitoring architecture, alert severity model, routing, noise reduction, SLO-based alerting, runbook discipline, and alert lifecycle

---

## Agenda

- Our Monitoring Stack — What We Use
- Alert Severity Definitions — Our Five-Level Model
- Mandatory Alert Labels — The Two Labels Every Alert Must Have
- Our Alert Philosophy — Everything Alerts, But at the Right Severity
- Alert Routing — Who Gets Paged for What
- SLO-Based Alerting — Multi-Window, Multi-Burn-Rate
- Monitoring the Monitor — The Dead Man's Switch
- Automated Alert Investigation
- How We Reduced Alert Noise
- Alert Lifecycle — From New Alert to Production
- Every Alert Must Have a Runbook
- Key Takeaways / Best Practices

---

## Our Monitoring Stack — What We Use

We rely on the **OCP built-in monitoring stack** — deliberately avoiding additional components that create maintenance overhead.

| Component | What It Does |
|-----------|-------------|
| **Prometheus** | Collects and stores metrics from all cluster components |
| **Alertmanager** | Routes alerts based on severity and service labels to the right destination |
| **OCP Console Dashboards** | Built-in visualization for cluster health, node resources, etcd, API server |
| **User Workload Monitoring** | Application teams instrument their own services; we scrape via ServiceMonitor/PodMonitor |

### External Integrations

| Source | Destination | Purpose |
|--------|-------------|---------|
| Alertmanager | **PagerDuty** | Pages on-call for critical alerts |
| Alertmanager | **Slack** | Posts alert summaries to team channels in real time |
| Alertmanager | **Dead Man's Snitch** | Heartbeat check — alerts us if the monitoring pipeline itself fails |
| CI/CD pipelines | **Slack** | Pipeline failure notifications — team triages during working hours |

### At Scale

We manage **2,700+ alert rules** across **450+ PrometheusRule files**, with **89% runbook coverage** and **86 SLO documents** across services. This isn't a small-scale setup — it's a mature, battle-tested alerting platform.

---

## Alert Severity Definitions — Our Five-Level Model

Every alert rule **must** have a severity label. We use a five-level severity model — each level has a defined escalation path and response expectation.

| Severity | Meaning | Escalation | Response Time |
|----------|---------|------------|---------------|
| **Critical** | Something is broken **right now** — customer experience is degraded or imminent | Pages on-call via PagerDuty + posts to Slack | Minutes |
| **Critical-FTS** | Same as critical, but only during business hours (Follow-the-Sun) | PagerDuty FTS policy only — no off-hours page | During working hours |
| **High** | Needs attention from both the service team and SRE | Posts to team Slack channel + SRE Slack channel | Same day — both teams aware |
| **Medium** | Service team can handle independently — SRE doesn't need to act | Posts to team Slack channel only | Same day — team triages |
| **Info** | Informational — useful context, no action expected | Slack notification (recommend a separate channel to avoid noise) | Best effort |

> **The key question for every alert:** "How fast must someone act, and who needs to act?" That determines severity.

### The Graduation Rule

All newly added alerts **start at `info` severity** unless an exception is approved during review. This allows us to observe behavior in practice, validate thresholds, and build confidence before promoting to higher severity.

---

## Mandatory Alert Labels — The Two Labels Every Alert Must Have

Every alert rule in our system requires exactly two labels:

| Label | Purpose |
|-------|---------|
| **`service`** | Identifies which service owns the alert — drives routing to the correct team channel |
| **`severity`** | Determines escalation path — PagerDuty, Slack, or both (see severity table above) |

Without both labels, an alert cannot be routed correctly. This is enforced during code review.

### Standard Alert Annotations

Beyond labels, every alert should include these annotations:

| Annotation | Purpose |
|------------|---------|
| **`message`** | Human-readable description shown in Slack/PagerDuty |
| **`runbook`** | Link to the standard operating procedure for this alert |
| **`dashboard`** | Link to the relevant Grafana dashboard (pre-selected variables preferred) |

---

## Our Alert Philosophy — Everything Alerts, But at the Right Severity

> "Every alert must be actionable. The difference is how urgently someone needs to act."

We alert on all of the conditions below — nothing is "just a dashboard." The distinction is **severity and response time**, not whether we alert at all.

| Condition | Severity | Response Expectation |
|-----------|----------|---------------------|
| API server errors > 1% for 5 min | **Critical** | Immediate — on-call investigates within minutes |
| API server latency trending up | **High** | Same day — both SRE and service team investigate |
| etcd no leader | **Critical** | Immediate — control plane is at risk |
| etcd db size growing | **High** | Same day — both teams plan compaction or remediation |
| Multiple nodes NotReady | **Critical** | Immediate — workloads are impacted |
| Single node high CPU | **High** | Same day — both teams check workload distribution, consider scaling |
| MachineConfigPool degraded | **Critical** | Immediate — node configuration drift |
| MachineConfigPool progressing | **Info** | Awareness — expected during upgrades, investigate if unexpected |
| Certificate expiring < 7 days | **Critical** | Immediate — services will break |
| Certificate expiring < 30 days | **High** | This week — both teams aware, plan rotation |
| Storage backend in critical state | **Critical** | Immediate — data at risk |
| Storage health warning | **High** | Same day — both teams investigate capacity and OSD health |
| SLO burn rate 14.4× (2% budget in 1 hour) | **Critical** | Immediate — error budget exhausting rapidly |
| SLO burn rate 6× (5% budget in 6 hours) | **Critical** | Immediate — sustained error budget burn |
| SLO burn rate 1× (10% budget in 3 days) | **High** | Same day — error rate not dropping, both teams investigate |

### What We Learned About Severity

- **Critical is reserved for "act now or things break."** Must relate to a degraded customer experience that's imminent or already ongoing.
- **Critical-FTS is for business-hours-only paging.** Same urgency, but doesn't wake anyone up at 3 AM.
- **High means both teams need to be in the loop.** The service team acts, SRE stays aware for potential escalation.
- **Medium is the service team's responsibility.** SRE doesn't act on these unless the team reaches out.
- **Info is context.** Useful during troubleshooting, but not worth interrupting anyone.

### Inhibition Rules

When a **critical** alert fires, matching **medium** alerts with the same `alertname`, `cluster`, and `service` are automatically suppressed. This prevents lower-severity noise from piling up during a major incident.

```yaml
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'medium'
    equal: ['alertname', 'cluster', 'service']
```

### Alerts We Removed Entirely

- **Pod restart alerts** — too noisy; most pod restarts are self-healing and expected
- **Disk usage warnings at 70%** — fires too early, creates noise weeks before any action is needed; we alert at 85% instead
- **Individual pod OOMKilled** — this is an application-level concern; app teams own their resource requests and limits
- **RDS burst balance predictions** — routed to Jira instead; not actionable as a Slack alert

---

## Alert Routing — Who Gets Paged for What

We use **PagerDuty** for paging and **Slack** for team-level notifications. Alertmanager routes all cluster alerts based on the `service` and `severity` labels.

For **CI/CD pipeline failures**, we use **Slack notifications** — these are not PagerDuty events. Pipeline failures post directly to the relevant Slack channel.

### Routing by Severity

| Severity | PagerDuty | Slack | Response |
|----------|-----------|-------|----------|
| **Critical** | Pages on-call immediately | Posted to team channel + SRE channel | Immediate |
| **Critical-FTS** | Pages FTS on-call only (business hours) | Posted to team channel + SRE channel | During working hours |
| **High** | No page | Posted to team channel + SRE channel | Same day — both teams aware |
| **Medium** | No page | Posted to team channel only | Same day — team triages |
| **Info** | No page | Team channel (separate channel recommended) | Best effort |
| **Deadman** | Via Dead Man's Snitch | N/A | Automatic — see DMS section |

### Alertmanager Routing (Simplified)

Alerts route based on `service` label first (to the correct team), then `severity` (to the correct escalation path).

```yaml
route:
  receiver: default-slack-alerts
  group_by: ['job', 'cluster', 'service']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 24h
  routes:
    - match:
        severity: critical
      receiver: pagerduty-sre
      continue: true
    - match:
        severity: critical-fts
      receiver: pagerduty-sre-fts-only
      continue: true
    - match_re:
        severity: ^(critical|critical-fts|high)$
      receiver: slack-sre-alerts
    - match:
        severity: deadman
      receiver: deadmanssnitch
      group_wait: 30s
      group_interval: 1m
      repeat_interval: 1m
```

### The `continue: true` Pattern

Critical alerts use `continue: true` — this means they match the PagerDuty route **and** continue to the Slack route. The on-call gets paged, and the team sees it in Slack simultaneously.

---

## SLO-Based Alerting — Multi-Window, Multi-Burn-Rate

Instead of alerting on raw thresholds ("error rate > 5%"), we alert on **how fast we're burning through our error budget**. This approach comes from the Google SRE Workbook and gives us precise, low-noise alerts tied directly to our SLO commitments.

### How Burn Rate Works

If your SLO is 99% availability over 30 days, your error budget is 1%. The **burn rate** measures how fast you're consuming that budget:

| Burn Rate | Meaning | Budget Exhausted In |
|-----------|---------|---------------------|
| **1×** | Consuming at expected rate | 30 days (full window) |
| **6×** | 6× faster than expected | 5 days |
| **14.4×** | Extremely fast burn | ~2 days |

### Our Alerting Thresholds

We use multi-window, multi-burn-rate alerts — a short window confirms the issue is still active, a long window confirms the trend:

| Long Window | Short Window (1/12th) | Burn Rate | Severity | What It Means |
|-------------|----------------------|-----------|----------|---------------|
| **1 hour** | **5 minutes** | **14.4×** | Critical | 2% of error budget consumed in 1 hour — page immediately |
| **6 hours** | **30 minutes** | **6×** | Critical | 5% of error budget consumed in 6 hours — page immediately |
| **3 days** | **6 hours** | **1×** | Medium | 10% of error budget consumed in 3 days — investigate, don't page |

### Why This Is Better Than Threshold Alerts

- **Threshold alert:** "Error rate > 5% for 5 min" — fires regardless of whether the SLO is actually at risk
- **Burn-rate alert:** "Consuming 14.4× error budget over 1 hour" — only fires when the SLO is genuinely threatened

> **We maintain 86 SLO documents across services**, each defining availability and/or latency objectives. Many generate burn-rate alerts automatically using recording rules.

---

## Monitoring the Monitor — The Dead Man's Switch

### The Problem

If Prometheus or Alertmanager goes down, there is nothing inside the cluster to alert on that failure. Worst case — an entire cluster goes down and PagerDuty is never notified. Silence looks the same as "everything is fine."

### How It Works: Watchdog + Dead Man's Snitch

OCP ships a built-in alert called **Watchdog** that fires continuously — it's always active. We route this alert from Alertmanager to **Dead Man's Snitch** (a PagerDuty integration) with a short repeat interval.

Dead Man's Snitch expects check-ins from each cluster. If it stops receiving the Watchdog alert, it creates a PagerDuty incident automatically.

```
Watchdog alert (always firing) → Alertmanager → [sends every 1 min] → Dead Man's Snitch
                                                                          ↓
                                                             If check-in stops → PagerDuty incident
```

The Alertmanager route for Dead Man's Snitch uses `repeat_interval: 1m` and `group_interval: 1m` — much shorter than normal alerts — so any disruption is detected quickly.

### What This Covers

- **Prometheus goes down** — Watchdog stops firing → DMS pages us
- **Alertmanager goes down** — Watchdog can't be sent → DMS pages us
- **Entire cluster goes down** — everything stops → DMS pages us
- **Network partition** — Alertmanager can't reach DMS → DMS pages us

A unique Dead Man's Snitch is configured per cluster, so we know exactly which cluster went silent.

> **This closes the blind spot:** in-cluster monitoring can't alert on its own failure. An external dead man's switch can.

---

## Automated Alert Investigation

### The Problem We Solved

Before automation, every alert required a human to log in, gather context, and start investigating — even for well-known patterns with documented resolutions.

### How It Works Now

We have an **automation layer** between PagerDuty and the SRE on-call. When certain alerts fire, the automation:

1. Receives the PagerDuty webhook
2. Runs automated investigation checks on the cluster
3. Posts investigation notes directly to the PagerDuty incident
4. If the cause is a known pattern (e.g., customer misconfiguration), it can auto-resolve or place the cluster in limited support

```
Alert fires → PagerDuty → Automation layer → Investigation runs → Notes posted to incident
                                                                    ↓
                                                            SRE gets a pre-investigated alert
```

| What Automation Handles | What SRE Handles |
|------------------------|-----------------|
| Cluster gone missing — check cloud provider status | Complex multi-component failures |
| Provisioning failure — gather install logs | Escalations requiring engineering |
| Known customer misconfigurations | Novel or undocumented alert patterns |
| Triggering must-gather for diagnostics | Incidents requiring customer communication |

> **Result:** The on-call engineer receives alerts that are already partially investigated. For many common patterns, the automation resolves the issue before a human needs to look.

---

## How We Reduced Alert Noise

### Where We Started

- Hundreds of active alert rules across our fleet
- Dozens of pages per week to the on-call engineer
- Low alert-to-action ratio — most alerts required no action
- On-call rotation was dreaded; team was burning out

### What We Did

| Step | Action | Impact |
|------|--------|--------|
| **1. Audit** | Reviewed every alert: "Can someone act RIGHT NOW?" If no → delete or downgrade severity | Cut alert count significantly |
| **2. Runbook requirement** | Every remaining alert MUST link to a runbook. No runbook = delete the alert | Forced clarity on what each alert means |
| **3. Ownership boundaries** | Platform alerts → SRE. Application alerts → app teams. We stopped carrying alerts for things we couldn't fix | Reduced noise from alerts we couldn't act on |
| **4. Five-level severity model** | Moved from 2-3 severity levels to five, each with a defined escalation path and response expectation | Right-sized the response to every alert |
| **5. Inhibition rules** | Critical suppresses matching medium alerts — less noise during incidents | Prevented alert pileup during outages |
| **6. Quarterly review** | "Alert gardening" session every quarter — review what fired, what was actionable, what to prune | Prevents alert sprawl from growing back |

### Where We Are Now

- Alert-to-action ratio significantly improved — most alerts require real action
- On-call is sustainable; engineers no longer dread the rotation
- Pages per week dropped by ~80%

---

## Alert Lifecycle — From New Alert to Production

New alerts don't go straight to critical. We use an **alert graduation pipeline**:

```
info → medium → high → critical
```

| Stage | What Happens |
|-------|-------------|
| **Info** | Alert fires and posts to the team's channel. We observe how often it fires and whether it correlates with real issues |
| **Medium** | Alert creates a Slack notification to the service team. We validate that the runbook is accurate and the alert is actionable |
| **High** | Alert notifies both the service team and SRE. Both teams validate the escalation path and response |
| **Critical** | Alert pages on-call. Only promoted after we've confirmed the alert is reliable, actionable, and has a tested runbook |

For SLO burn-rate alerts specifically, we use a **soaking PagerDuty service** — new burn-rate alerts route to a silent PD service first. If they fire correctly and correlate with real incidents, we promote them to the production PD service.

### Before Adding a Custom Alert

We ask these questions:

1. **Can this alert be contributed upstream to OCP?** (Preferred — the community maintains it)
2. **Does this alert overlap with an existing OCP alert?** (Don't duplicate)
3. **Can we write a runbook for it?** (If not, the alert isn't actionable)
4. **Should this alert block cluster upgrades?** (Very few should)

> **Our goal:** Minimize custom alerts we maintain. Every custom alert is technical debt.

---

## Every Alert Must Have a Runbook

### The Rule

No alert goes to production without a corresponding runbook (SOP). Across our fleet, **89% of our 2,700+ alerts** link to a runbook — and we're actively closing the remaining gap.

### What a Runbook Contains

| Section | Content |
|---------|---------|
| **Alert description** | What the alert means, what metric it monitors |
| **Impact** | What breaks if this is not addressed |
| **Investigation steps** | Specific commands and checks to diagnose the issue |
| **Remediation** | Step-by-step resolution |
| **Escalation** | When to escalate and to whom |

### When a Runbook Is Missing

If an alert fires and the runbook link is broken or missing:

1. Create a new runbook from the standard template
2. Fill in what you know from investigating
3. If you don't know the full resolution, flag it for the team to complete
4. No alert should fire more than once without a runbook

> **Why this matters:** When the on-call engineer gets paged at 3 AM, the runbook is the difference between a 5-minute resolution and a 45-minute investigation.

---

## Key Takeaways / Best Practices

1. **Stick with the built-in monitoring stack** — every additional component is something you have to maintain
2. **Five severity levels, each with a defined escalation path** — critical pages, high notifies both teams, medium is team-only, info is context
3. **Two mandatory labels on every alert: `service` and `severity`** — without them, routing breaks
4. **Every alert must have a runbook** — if you can't write one, delete the alert
5. **Use SLO-based burn-rate alerting** — alert on error budget consumption, not raw thresholds
6. **New alerts start at `info` and graduate up** — info → medium → high → critical
7. **Use a dead man's switch** — Watchdog + Dead Man's Snitch catches monitoring failures in under a minute
8. **Automate investigation for known patterns** — the on-call should receive pre-investigated alerts
9. **Use inhibition rules** — critical suppresses medium to reduce noise during incidents
10. **Review and prune alerts quarterly** — they grow back if you don't maintain them

**Up Next:** Session 8 — How We Handle Incidents
