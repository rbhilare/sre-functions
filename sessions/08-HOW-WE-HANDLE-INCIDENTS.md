# Session 8: How We Handle Incidents

**Our Incident Lifecycle, Roles, Communication, and Post-Incident Review Process**

**Presenter:** Rohit Bhilare — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat
**Duration:** 30 minutes | **Format:** Peer conversation, not a lecture
**Series:** SRE Best Practices — How We Do Things (Session 8 of 14)
**Focus:** Incident lifecycle, severity classification, roles, communication, escalation, handover, post-incident review, and blameless culture

---

## Agenda

- The Alert Is Not the Incident — A Critical Distinction
- Our Incident Lifecycle — Four Phases
- Severity Classification — Sev1, Sev2, Sev3
- Roles During an Incident — Three Core Roles
- Phase 1: Identify — When to Call an Incident
- Phase 2: Create — Declaring the Incident
- Phase 3: Resolve — Active Response
- Customer Communication — Who Gets Told What
- Escalation — When and How to Ask for Help
- Incident Handover — Passing the Baton Across Regions
- Phase 4: Post-Incident Response — Learning from Every Incident
- Key Takeaways / Best Practices

---

## The Alert Is Not the Incident — A Critical Distinction

Before we go further, an important distinction:

| Term | What It Means |
|------|---------------|
| **PagerDuty alert** | An automated page triggered by a Prometheus alert rule — happens dozens of times per week |
| **Incident** | A formally declared event with customer impact that requires coordinated response — happens rarely |

Most PagerDuty alerts are handled by the on-call engineer following a runbook. They resolve without ever becoming an incident.

An **incident** is declared when the situation exceeds what one engineer can handle following a runbook — it requires coordination, communication, and formal tracking.

```
Alert fires → PagerDuty → On-call investigates → Resolves via runbook (most cases)
                                                → Declares incident (when criteria met)
```

---

## Our Incident Lifecycle — Four Phases

```
IDENTIFY → CREATE → RESOLVE → POST-INCIDENT
```

| Phase | Goal | Key Action |
|-------|------|------------|
| **Identify** | Recognize that an incident is occurring | Assess scope, impact, urgency |
| **Create** | Formally declare the incident with the right severity | Create incident record, assign roles |
| **Resolve** | Fix the issue, communicate status, record actions | Investigate, mitigate, communicate, resolve |
| **Post-Incident** | Learn from the incident and prevent recurrence | PMR, RCA, corrective actions |

---

## Severity Classification — Sev1, Sev2, Sev3

Severity determines response urgency, check-in frequency, and post-incident requirements.

| Severity | Impact | Examples | Check-in Interval |
|----------|--------|----------|-------------------|
| **Sev1 — Critical** | Brand-impacting, major customer impact, fleet- or service-wide | Multi-region outage, irrecoverable data loss, credential breach | Every **20 minutes** |
| **Sev2 — Major** | Single-service SLA impact, production outage or degradation | Monitoring/deploy failures, single-customer outage, imminent Sev1 risk | Every **30 minutes** |
| **Sev3 — Minor** | High business impact, **no SLA impact** | Urgent maintenance, compliance violations, loss of redundancy | Every **60 minutes** |

### Check-in intervals are non-negotiable

These intervals are aligned with our production SLA. During each check-in, the Incident Owner pauses the investigation and asks:

1. **People:** Do we have the right SMEs? Does anyone need a break?
2. **Progress:** Are we making progress? What's our goal for the next period?
3. **Decisions:** Has severity increased? Do we need to hand over to another region? Should we pause?

### Re-classification during an incident

Severity can **increase** during an incident as impact grows or time passes. The check-in explicitly asks: "Has severity increased because of increased impact or length of incident?"

---

## Roles During an Incident — Three Core Roles

We use **three simplified roles**. They are filled based on expertise and need, and are often organically assumed. One person can hold multiple roles. By default, the person who declares the incident holds all roles until they delegate.

| Role | Who | Responsibilities |
|------|-----|-----------------|
| **Tech Lead** | SRE or engineering | High-level technical understanding; structures the team; arranges handover between regions; tracks debug changes that must be reverted |
| **Incident Owner** | SRE or engineering; **manager for high severity** | Customer perspective; ensures team collaboration and communication; escalation point for stakeholders; **owns all post-incident actions** |
| **Scribe** | Any team member | Updates incident status; records logs, key decisions, and check-in outcomes |

### Why three roles?

We tried more complex role structures in the past — Incident Commander, Communications Lead, Liaison, etc. Most roles went unfilled because they were too granular for our team size. We simplified to three roles that actually get used.

### For high-severity incidents

The Incident Owner **must be delegated to a manager** as soon as one is available. This frees engineers to focus on technical resolution while the manager handles stakeholder communication.

---

## Phase 1: Identify — When to Call an Incident

Call an incident **if ANY of the following are true:**

| Criteria | Description |
|----------|-------------|
| **Scope** | Fleet-wide or service-wide; dependent service outage (e.g., cloud provider) |
| **Impact** | SLA impacted; significant customer or organizational impact |
| **Urgency** | Must be addressed immediately |
| **SLO** | Internal standards have been breached |
| **Complexity** | Requires coordinated effort of more than one engineer |

> **When in doubt, call the incident.** Any delay in declaring an incident directly impacts MTTR. It's always better to declare and stand down than to delay and let the situation escalate.

---

## Phase 2: Create — Declaring the Incident

### How to Declare

We use an **incident management tool** that automates the initial setup:

1. Declare the incident via the tool (web UI or Slack command)
2. The tool **automatically creates** a dedicated Slack channel for the incident
3. The tool **automatically creates** a video bridge (Google Meet) for the incident team
4. The tool **automatically announces** the incident in the announcements channel

### After Declaration

1. Review severity criteria and assign the correct severity
2. Form the incident team — assign Tech Lead, Incident Owner, and Scribe
3. **Start customer communication immediately** — send an initial notification even before root cause is known

### The "Unknown Failure" Notification

We send customer notifications at the start of every incident, even before we know the root cause. The message is simple: "We've identified an issue affecting your cluster/service. We're investigating and will provide updates."

This is better than silence. Customers would rather know you're aware and working on it than wonder if anyone has noticed.

---

## Phase 3: Resolve — Active Response

### The Resolution Flow

```
Assess Impact → Communicate → Investigate → Mitigate → Resolve → Wrap Up
```

| Step | What Happens |
|------|-------------|
| **Assess impact** | Record a customer impact statement as soon as possible. Update as understanding evolves |
| **Communicate** | Notify customers per the communication matrix (see next section). Post check-in updates |
| **Investigate** | Follow runbooks, gather diagnostics (`must-gather`), post all findings in the incident channel |
| **Mitigate** | Restore service FIRST. A running service with an unknown root cause is better than a down service with perfect understanding |
| **Resolve** | Apply the permanent fix. Validate it holds |
| **Wrap up** | Resolve the incident, update customer channels, post root cause or hypothesis |

### Rules We Enforce During Incidents

- **Everything in the incident channel.** No side DMs, no unrecorded conversations. If you discuss something on the bridge call, post a summary in the channel afterward.
- **Read before you ask.** If you join mid-incident, read the channel history before asking questions.
- **Mitigate first, RCA second.** Restart the pod, failover the workload, cordon the node — whatever stops the bleeding. Root cause analysis comes after mitigation.
- **Collect diagnostics early.** Run `must-gather` during the incident, not after. Evidence disappears.

### Unusual Scenarios

- **Forked investigations:** Split into sub-teams when the problem is complex. Parallelizing investigation shortens MTTR.
- **Long-running root cause:** If the root cause is exceptionally difficult to find, create a hypothesis document and systematically eliminate each hypothesis.

---

## Customer Communication — Who Gets Told What

Communication during an incident is determined by the **scope of impact**:

| Impact / Scope | Customer Channels | Who Approves |
|----------------|-------------------|-------------|
| **Service-wide or cloud provider** | Public status page + knowledge base article + email | PM + senior management |
| **Fleet-wide** | Public status page + knowledge base article + email | PM + senior management |
| **Partial fleet** (e.g., specific version) | Service log (per-cluster notification) + knowledge base article | PM + senior management |
| **Single cluster/customer** | Service log or proactive support case | Incident Owner |

### Key Rules

- **Communicate early.** Send the initial "we're investigating" notification before you have the full picture.
- **Communicate regularly.** Updates at each check-in interval (20/30/60 min depending on severity).
- **Update the status page for fleet-wide issues.** This is the public-facing record.
- **Post resolution communication.** When resolved, send a resolution notification with root cause or hypothesis and updated impact statement.

---

## Escalation — When and How to Ask for Help

Escalation is not failure. Sitting stuck without asking for help IS failure.

| Situation | Action |
|-----------|--------|
| **Need technical help from another team** | Engage additional SMEs via the incident channel |
| **Impact or severity has increased** | Consult management and customer-facing teams |
| **Incident roles are not being filled** | Internal escalation to management |
| **30+ minutes on a Sev1 with no clear next step** | Escalate immediately — no judgment, no questions asked |

### Before Escalating to Vendor Support

Always have these ready:

1. **Cluster ID** — identifies the specific cluster
2. **Diagnostic data** — `must-gather` output collected from the cluster
3. **Timeline** — what has been tried so far and when
4. **Specific errors** — error messages and affected components

### Our "Don't Stall" Rule

> If you are 30 minutes into a Sev1 or Sev2 and you do not have a clear next step, you **must** escalate. No judgment, no questions asked. Escalating early is a sign of maturity, not weakness.

---

## Incident Handover — Passing the Baton Across Regions

For incidents that span regions (Follow-the-Sun), we have a structured handover process:

### 1. Executive Summary in the Incident Channel

The outgoing team posts a written summary including:

- Current technical status of the issue
- Time and content of the most recent customer update
- Key stakeholders engaged outside SRE
- Current hypotheses regarding root cause
- Recommended next steps

### 2. Verbal Handover on the Bridge

The outgoing Incident Owner pulls the incoming region leads into the bridge **15 minutes before handover** to:

- Identify the incoming Incident Owner
- Walk through the executive summary live
- Answer real-time questions and clarifications

### 3. Clean Disengagement

The outgoing team disengages **no later than 30 minutes** after the regional handoff time. No lingering, no split ownership.

---

## Phase 4: Post-Incident Response — Learning from Every Incident

The Incident Owner is responsible for **all** post-incident activities.

### Timeline

| Step | Deadline (Days Post-Incident) | Required For | Artifact |
|------|-------------------------------|-------------|----------|
| **Debrief session** (optional) | 2 days | Incident Owner's discretion | Debrief notes |
| **RCA draft** | 5 days | Sev1 | Customer-facing RCA document |
| **Host PMR** (Post-Mortem Review) | 5 days | Sev1, Sev2 | PMR recording |
| **Corrective actions filed** | 7 days | All severities | Jira issues linked to incident |
| **Completed RCA** | 7 days | Sev1 | RCA PDF attached to support case |

### Post-Mortem Review (PMR)

**Goals:**

- Ensure the RCA accurately describes the incident
- Surface all corrective actions
- Provide an opportunity to raise concerns **in a blameless manner**
- Review customer-facing RCA language with stakeholders

**Format:**

1. Review each section of the RCA
2. Review all corrective actions
3. Discuss what went well and what didn't

**Invitees:** All incident participants, product managers, account representatives, support engineers, SRE teams, engineering teams. Use a group calendar to maximize accessibility.

### Corrective Actions

Corrective actions are the **primary way to prevent recurrence**. They go into the team's backlog as tracked items.

- Focus on **critical items only** — avoid "nice-to-have" items that will never get prioritized
- Link corrective actions to the incident record for traceability
- Review progress regularly — unfinished actions mean repeat incidents

### Debrief Session (Optional)

An informal meeting held within 48 hours to:

1. Recognize team strengths observed during the incident
2. Identify areas for improvement
3. Collaborate on drafting the RCA while memory is fresh

### Our Blameless Culture

> We ask "What allowed this to happen?" — not "Who did this?"

Systems fail. Humans make mistakes in context. Our job is to fix the system so the same mistake can't happen again — not to find someone to blame.

---

## Key Takeaways / Best Practices

1. **A PagerDuty alert is not an incident** — most alerts resolve via runbooks; incidents are formally declared when coordination is needed
2. **When in doubt, call the incident** — delays hurt MTTR; it's better to declare and stand down than to delay
3. **Three roles, organically assumed** — Tech Lead, Incident Owner, Scribe; one person can hold multiple roles
4. **Check-ins are non-negotiable** — Sev1 every 20 min, Sev2 every 30 min, Sev3 every 60 min
5. **Communicate early and often** — customers would rather know you're working on it than hear silence
6. **Mitigate first, RCA second** — restore service before investigating root cause
7. **Escalating is not failure** — stalling without asking for help IS failure
8. **Structured handover** — executive summary + 15-min verbal handover + clean disengagement
9. **Every Sev1/2 gets a blameless PMR within 5 days** — corrective actions are tracked, not forgotten
10. **Collect diagnostics during the incident** — evidence disappears; run `must-gather` early

**Up Next:** Session 9 — Inside SRE: Ask & Discuss (Sessions 7–8)
