# Session 4: How We Run SRE

**Our Practices, Processes & Lessons Learned**

**Presenter:** Rohit Bhilare — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat
**Duration:** 40 minutes | **Format:** Peer conversation, not a lecture
**Series:** SRE Best Practices — How We Do Things (Session 4 of 14)
**Focus:** Team structure, ownership, on-call, culture, and working principles

---

## Agenda

- Our SRE Organization — Structure and Dual Operating Model
- Principles of Our SRE Culture
- Domain-Based Functional Teams
- How We Run On-Call (PagerDuty + CAD Automation)
- Incident Response — From Alert to Resolution
- Our Blameless Culture — How We Practice It
- Anti-Patterns We Eliminated
- Key Takeaways / Best Practices

---

## Our SRE Organization — Structure and Dual Operating Model

Every engineer on the team operates in **two modes**:

```
                     +----------------------------+
                     |       SRE Platform          |
                     |    (Service Delivery)       |
                     +----------------------------+
                                 |
              +------------------+------------------+
              |                                     |
    +---------+----------+              +-----------+---------+
    | Functional Work    |              | Operations (Ops)    |
    | (70% of sprint)    |              | (On-call shifts)    |
    +--------------------+              +---------------------+
    | Platform features  |              | Alert response      |
    | Automation         |              | Incident management |
    | Operator dev       |              | Customer escalation |
    | Tooling            |              | Shift improvement   |
    +--------------------+              +---------------------+
```

### Sprint Capacity Allocation

| Category | % of Sprint | Examples |
|----------|-------------|---------|
| Feature / Epic work | 70% | Automation, operator development, platform improvements |
| Shift improvement | 10% | Fixing alert noise, improving runbooks, automating toil |
| Maintenance | 10% | Dependency updates, PR reviews, Gardening |
| Learning goals | 5% | Certifications, tech exploration, skill development |
| Buffer (PTO / ad-hoc) | 5% | Unplanned work, coverage gaps |

> **Key rule:** During ops shifts, engineers do NOT pick up functional sprint tickets — focus stays on operations, peer reviews, and identifying shift improvements.

---

## Principles of Our SRE Culture

**These are the core values that shape how we work, how we treat each other, and how we make decisions as a team.**

| # | Principle | What It Means In Practice |
|---|-----------|---------------------------|
| 1 | **"It's OK to fail."** |  Learn from it; don't repeat it. Failure is expected in complex systems. We treat every failure as a learning opportunity, not a reason for blame. What matters is that we extract lessons and put safeguards in place so the same failure does not happen again. |
| 2 | **"Assume positive intent."** | Information may be confidential. Ask questions — you may not get all the answers and may be asked to do some work without full context. Assume that people are acting with good intentions, even when the picture is incomplete. |
| 3 | **"Start with trust and extend trust."** | We encourage curiosity and questions. But we do not make assumptions about people or their decisions. Trust is the default, not something that has to be earned through gatekeeping. |
| 4 | **"Disagree and commit."** | Make a decision, move forward, learn from it, and course correct. Do not get caught up in analysis paralysis. It is better to act, learn, and adjust than to debate endlessly and never ship. |
| 5 | **"Communicate."** | When in doubt, share more context, not less. Silence creates uncertainty. Whether it is an incident update, a design decision, or a process change — communicate early, communicate often, and communicate clearly. |

---

## Domain-Based Functional Teams

We organize into **domain-based functional teams**, each owning a specific area of the platform:

| Domain | Focus Areas |
|--------|-------------|
| Managed Platform | Cluster lifecycle, upgrade orchestration, fleet management, notifications |
| Networking & Identity | Network verification, ingress, account operations |
| Observability | Alerting, monitoring stack, backup & restore |
| IAM & Access | Cluster access, RBAC, validating webhooks |

### Roles Within Team

| Role | Responsibility |
|------|---------------|
| **Functional Lead (FL)** | Sprint planning, epic breakdown, stakeholder sync |
| **Region Lead (RL)** | Regional ops coordination; approves sensitive actions |
| **Team Lead (TL)** | Cross-team priorities; escalation decisions |

### Why Domain Teams Work

- **Before:** Everyone did everything — no deep expertise, constant context switching
- **After:** Each team builds deep domain expertise while still rotating through ops
- People move between tracks over time for knowledge sharing
- Dedicated communication channels per domain — questions go to the right team, not a generic queue

---

## How We Run On-Call

### PagerDuty-Driven Rotation

- **Weekly rotation:** One primary, one secondary
- Rotation changes on Monday mornings with a **structured handoff**
- Automated bot announces shift changes so the whole team knows who is on point/on-call
- Engineers enter on-call shifts in on-call calendar **at least 2 month ahead**

### Shift Handover Process

```bash
# Check current on-call
ESCALATION_POLICY_ID=<id>
pd ep:oncall -i ${ESCALATION_POLICY_ID} --sort Level \
  --since '15 minutes ago' --until '15 minutes'

# Reassign open incidents to incoming on-call
pd incident:list --me --pipe | \
  pd incident:assign --assign_to_ep_id ${ESCALATION_POLICY_ID} --pipe
```

Handoff includes: open incidents, known issues, upcoming changes, anything unusual.

### CAD — Our Automation Layer Between PagerDuty and SRE

```
Alert fires → PagerDuty (critical) → CAD webhook
                                        |
                      +-----------------+------------------+
                      |                 |                  |
              Auto-silence      Gather data &       No investigation
            (known customer     escalate to SRE      available →
             failure → SRE     with notes attached   escalate to SRE
             never paged)
```

**Configuration Anomaly Detection (CAD)** runs codified investigations:

- Receives PagerDuty webhook on PagerDuty alerts
- Auto-silences alerts for known customer-caused failures — **SRE never paged**
- Gathers investigation data and attaches to PagerDuty notes before escalating
- Can auto-place clusters in **limited support** when root cause is clear

### What On-Call IS and IS NOT

| On-call IS | On-call IS NOT |
|------------|----------------|
| Respond to firing alerts within 15 minutes | Investigating every issue alone |
| Triage critical issues and escalate if needed | Being awake 24/7 — secondary exists for a reason |
| Update the team in our incident channel | Fixing everything yourself — mitigate and hand off |
| Log all actions in the ticket | Deep-diving into root cause during an active outage |

---

## Incident Response — From Alert to Resolution

### Declaring an Incident

1. Run **`/rca new`** in the SRE channel or create incident in WebRCA UI console
2. WebRCA auto-creates:
   - Dedicated incident channel
   - Video/Meet bridge link
   - Announcement in the incidents channel
3. Join bridge; enable **AI transcription** for automated note-taking
4. Start customer communication (service logs or status page update)

### During the Incident

- Record **all** communication and actions in the incident channel
- CEE Team leads customer-facing communication when on bridge
- Post periodic executive summaries
- Alert storms: Run `pd-ack-all` script to bulk-acknowledge

### Cross-Region Handover (Follow-the-Sun)

Executive summary must include:

- Technical status and current hypothesis
- Time and content of last customer update
- External stakeholders engaged
- Next steps and open action items

### After Recovery

- Update tracking ticket
- Complete RCA using `incident-rca` utility
- Schedule **PMR (Post-Mortem Review) within 5/7 business days**
- Drive follow-up actions to completion
- Archive incident channel

---

## Our Blameless Culture — How We Practice It

**Blameless culture is not a slogan on our wiki. It is a set of specific practices we enforce.**

### Post-Incident Reviews Focus on Systems, Not People

- We ask: *"What allowed this to happen?"* — never *"Who did this?"*
- Root cause is always a process gap, a missing check, or a tooling failure — never a person

### Psychologically Safe Retrospectives

- Retros use **anonymous Google Forms** — a facilitator reads comments aloud as "anonymous contributor"
- Start/Stop/Continue format with action items assigned to owners
- Quarterly team health checks with anonymous feedback to management

### Shadow On-Call for New Team Members

- New engineers shadow for **two full rotations** before going primary
- They observe, ask questions, and build confidence without the pressure of being the decision-maker

### AI-Assisted RCA

- We use automated tools to summarize incident channels for RCA
- RCA generation utilities pull events and follow-ups, outputting structured data for the RCA template
- This reduces the manual burden of reconstructing timelines so engineers can focus on analysis

---

## Anti-Patterns We Eliminated

| What others Do | What We Do  |
|--------------------|----------------|
| Random ad-hoc calls for every issue | Structured workflow: PagerDuty → CAD → SRE → WebRCA if needed |
| One person held all the knowledge | 200+ documented runbooks, enforced rotation across all systems |
| Hundreds of noisy alerts firing constantly | Alert graduation pipeline (soaking → warning → critical) + runbook requirement |
| Manual investigation for every alert | CAD auto-investigates and auto-silences known failures |
| Upgrades feared and delayed for months | Progressive delivery: INT → STAGE → PROD Canary → Phase 2 → Phase 3 |
| Post-mortems blamed individuals | PMR within 5/7 days, anonymous retros, psychologically safe |
| Knowledge locked in DMs and heads | "Ask in public channels" rule + regular knowledge sharing |

### Toil Reduction Loop

```
Alert fires → SRE investigates → Creates shift-improvement card for any kind of improvement

```

---

## Key Takeaways / Best Practices

1. **Adopt a dual operating model** — separate functional work from ops, with clear sprint allocation (70/10/10/5/5)
2. **Organize by domain** — functional teams build deep expertise while rotating through shared on-call
3. **Automate the investigation layer** — tools like CAD between PagerDuty and SRE dramatically reduce page volume
4. **Enforce the runbook rule** — no runbook = no alert. This forces intentional, actionable alerting
5. **Make on-call sustainable** — page budgets, structured handoffs, and "fix the system, not the engineer" mindset
6. **Practice blameless culture with specific habits** — anonymous retros, no names in reports, near-miss sharing
7. **Use progressive delivery for changes** — INT → STAGE → PROD with soak periods eliminates risky big-bang releases
8. **Close the toil reduction loop** — every manual investigation should feed back into automation so it never pages again

---

**Up Next:** Session 5 — How We Operate OCP Clusters
