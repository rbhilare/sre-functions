# Ideal SRE Workflow for Product Support

**Applicable to:** Organizations serving 80 Lakh+ (8 Million+) customers  
**Version:** 1.0  
**Last Updated:** June 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Stakeholders & Teams](#2-stakeholders--teams)
3. [High-Level Workflow Diagram](#3-high-level-workflow-diagram)
4. [Workflow Track A — Customer-Initiated (Reactive)](#4-workflow-track-a--customer-initiated-reactive)
5. [Workflow Track B — SRE-Initiated (Proactive)](#5-workflow-track-b--sre-initiated-proactive)
6. [Detailed State Machine](#6-detailed-state-machine)
7. [Escalation Matrix](#7-escalation-matrix)
8. [SLA Definitions](#8-sla-definitions)
9. [Tooling Stack](#9-tooling-stack)
10. [Observability & Alerting Framework](#10-observability--alerting-framework)
11. [Incident Management Lifecycle](#11-incident-management-lifecycle)
12. [Communication Protocols](#12-communication-protocols)
13. [Post-Incident Review (PIR)](#13-post-incident-review-pir)
14. [Metrics & KPIs](#14-metrics--kpis)
15. [Runbook Template](#15-runbook-template)

---

## 1. Executive Summary

At scale (8M+ customers), product support cannot be purely reactive. Top-tier organizations operate a **dual-track model**:

| Track | Trigger | Goal |
|-------|---------|------|
| **Track A — Reactive** | Customer raises a support ticket | Resolve customer-reported issues through tiered escalation |
| **Track B — Proactive** | SRE observability detects anomaly | Detect, mitigate, and resolve issues *before* customers notice |

Both tracks converge when an issue requires a product-level fix from the Engineering team. The workflow below codifies how tickets flow, who owns what, and how teams coordinate at each stage.

---

## 2. Stakeholders & Teams

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ORGANIZATION                                 │
│                                                                     │
│  ┌────────────┐   ┌────────────────┐   ┌────────────────────────┐   │
│  │  CUSTOMER  │   │  SUPPORT TEAM  │   │      SRE TEAM          │   │
│  │  (8M+ base)│   │  (L1 / L2)     │   │  (On-call + Platform)  │   │
│  └────────────┘   └────────────────┘   └────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────┐   ┌──────────────────────────────────┐    │
│  │  ENGINEERING TEAM    │   │  PRODUCT / PROGRAM MANAGEMENT    │    │
│  │  (Backend/Frontend/  │   │  (Prioritization, Roadmap)       │    │
│  │   Infra/Platform)    │   │                                  │    │
│  └──────────────────────┘   └──────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Team Responsibilities

| Team | Primary Responsibility |
|------|----------------------|
| **Customer** | Raises tickets, provides reproduction steps, validates resolution |
| **Support Team (L1)** | First response, triage, known-issue resolution, FAQ deflection |
| **Support Team (L2)** | Deep technical troubleshooting, log analysis, workaround identification |
| **SRE Team** | Infrastructure health, incident management, observability, reliability fixes, coordination with Engineering |
| **Engineering Team** | Product bug fixes, feature corrections, code-level root cause analysis, hotfix/release deployment |
| **Product Management** | Prioritization of fixes vs. feature work, customer communication strategy for large-scale incidents |

---

## 3. High-Level Workflow Diagram

```
                        ┌─────────────────────────────────────────────────────────┐
                        │              TRACK B — PROACTIVE (SRE-INITIATED)        │
                        │                                                         │
                        │   ┌──────────────┐    ┌──────────────┐                  │
                        │   │ Observability │───▶│  Alert Fires │                  │
                        │   │  & Monitoring │    │  (PagerDuty) │                  │
                        │   └──────────────┘    └──────┬───────┘                  │
                        │                              │                          │
                        │                              ▼                          │
                        │                     ┌────────────────┐                  │
                        │                     │ SRE On-Call     │                  │
                        │                     │ Acknowledges    │                  │
                        │                     └───────┬────────┘                  │
                        │                             │                           │
                        └─────────────────────────────┼───────────────────────────┘
                                                      │
                                                      ▼
 ┌──────────────────────────────────────────────────────────────────────────────────┐
 │                                                                                  │
 │    TRACK A — REACTIVE (CUSTOMER-INITIATED)                                       │
 │                                                                                  │
 │    ┌──────────┐    ┌───────────────┐    ┌───────────────┐                        │
 │    │ Customer  │───▶│ Support Team  │───▶│ Support Team  │                        │
 │    │ Raises    │    │ L1 Triage     │    │ L2 Deep-Dive  │                        │
 │    │ Ticket    │    │               │    │               │                        │
 │    └──────────┘    └───────┬───────┘    └───────┬───────┘                        │
 │                            │                    │                                │
 │                    Resolved│ at L1       Resolved│ at L2                          │
 │                            │                    │                                │
 │                            ▼                    ▼                                │
 │                     ┌─────────────┐      ┌─────────────┐                         │
 │                     │  Customer   │      │  Customer   │                         │
 │                     │  Updated    │      │  Updated    │                         │
 │                     └─────────────┘      └─────────────┘                         │
 │                                                                                  │
 └──────────────────────────────┬───────────────────────────────────────────────────┘
                                │
                   Not resolved │ at Support Level
                                │
                                ▼
                 ┌──────────────────────────┐
                 │       SRE TEAM           │
                 │  Investigates Issue      │
                 │                          │
                 │  ┌────────────────────┐  │
                 │  │ Infrastructure /   │  │
                 │  │ Config / Capacity? │  │
                 │  └────────┬───────────┘  │
                 │           │              │
                 │     YES   │   NO         │
                 │     ┌─────┘   └──────┐   │
                 │     ▼                ▼   │
                 │  ┌────────┐  ┌──────────┐│
                 │  │SRE     │  │Escalate  ││
                 │  │Resolves│  │to Engg   ││
                 │  └───┬────┘  └─────┬────┘│
                 │      │             │     │
                 └──────┼─────────────┼─────┘
                        │             │
                        │             ▼
                        │   ┌──────────────────────┐
                        │   │  ENGINEERING TEAM    │
                        │   │                      │
                        │   │  Root Cause Analysis │
                        │   │  Code Fix / Hotfix   │
                        │   │  Deploy to Production│
                        │   │                      │
                        │   └──────────┬───────────┘
                        │              │
                        │              │ Fix Deployed
                        │              │ + Informs SRE
                        │              │
                        ▼              ▼
                 ┌──────────────────────────┐
                 │       SRE TEAM           │
                 │  Validates Fix           │
                 │  Updates Support Team    │
                 └────────────┬─────────────┘
                              │
                              ▼
                 ┌──────────────────────────┐
                 │     SUPPORT TEAM         │
                 │  Updates Customer        │
                 │  Closes Ticket           │
                 └────────────┬─────────────┘
                              │
                              ▼
                 ┌──────────────────────────┐
                 │       CUSTOMER           │
                 │  Validates & Confirms    │
                 │  Issue Resolved          │
                 └──────────────────────────┘
```

---

## 4. Workflow Track A — Customer-Initiated (Reactive)

### Step-by-Step Flow

```
  CUSTOMER                SUPPORT L1              SUPPORT L2               SRE TEAM             ENGINEERING
     │                        │                       │                       │                       │
     │  1. Raise Ticket       │                       │                       │                       │
     │───────────────────────▶│                       │                       │                       │
     │                        │                       │                       │                       │
     │                        │ 2. Auto-Acknowledge   │                       │                       │
     │◀───────────────────────│    (SLA clock starts) │                       │                       │
     │                        │                       │                       │                       │
     │                        │ 3. Triage & Classify  │                       │                       │
     │                        │    (P1/P2/P3/P4)      │                       │                       │
     │                        │                       │                       │                       │
     │                        │ 4a. Known Issue?       │                       │                       │
     │                        │     YES ──▶ Resolve    │                       │                       │
     │◀───────────────────────│     & Update Customer  │                       │                       │
     │                        │                       │                       │                       │
     │                        │ 4b. NO ──▶ Escalate   │                       │                       │
     │                        │───────────────────────▶│                       │                       │
     │                        │                       │                       │                       │
     │                        │                       │ 5. Deep Investigation │                       │
     │                        │                       │    (Logs, Repro,      │                       │
     │                        │                       │     Environment)      │                       │
     │                        │                       │                       │                       │
     │                        │                       │ 6a. Resolved at L2?   │                       │
     │◀───────────────────────│◀──────────────────────│     YES ──▶ Update    │                       │
     │                        │                       │                       │                       │
     │                        │                       │ 6b. NO ──▶ Escalate  │                       │
     │                        │                       │     to SRE            │                       │
     │                        │                       │───────────────────────▶│                       │
     │                        │                       │                       │                       │
     │                        │                       │                       │ 7. SRE Investigates   │
     │                        │                       │                       │    (Infra? Config?    │
     │                        │                       │                       │     Capacity? Code?)  │
     │                        │                       │                       │                       │
     │                        │                       │                       │ 8a. SRE Resolves      │
     │                        │◀──────────────────────│◀──────────────────────│     (Infra/Config fix)│
     │◀───────────────────────│                       │                       │                       │
     │                        │                       │                       │                       │
     │                        │                       │                       │ 8b. Product Bug ──▶   │
     │                        │                       │                       │     Escalate to Engg  │
     │                        │                       │                       │───────────────────────▶│
     │                        │                       │                       │                       │
     │                        │                       │                       │                       │ 9. Engg RCA + Fix
     │                        │                       │                       │                       │    Code Review
     │                        │                       │                       │                       │    Deploy Hotfix
     │                        │                       │                       │                       │
     │                        │                       │                       │◀──────────────────────│ 10. Inform SRE
     │                        │                       │                       │     (Fix Deployed)    │
     │                        │                       │                       │                       │
     │                        │                       │                       │ 11. Validate Fix      │
     │                        │◀──────────────────────│◀──────────────────────│ 12. Update Support    │
     │◀───────────────────────│                       │                       │                       │
     │                        │                       │                       │                       │
     │  13. Confirm Resolution│                       │                       │                       │
     │───────────────────────▶│                       │                       │                       │
     │                        │ 14. Close Ticket      │                       │                       │
     │                        │                       │                       │                       │
```

### Detailed Stage Descriptions

#### Stage 1-2: Ticket Creation & Auto-Acknowledgement

- Customer submits ticket via portal, email, chat, or API
- System auto-acknowledges with ticket ID and expected response time
- Ticket is auto-classified using ML-based categorization (product area, severity hint)
- SLA clock starts immediately upon ticket creation

#### Stage 3: Triage & Classification (Support L1)

Support L1 classifies the ticket based on **impact and urgency**:

| Priority | Description | Example | First Response SLA | Resolution SLA |
|----------|-------------|---------|-------------------|----------------|
| **P1 — Critical** | Service down, data loss, security breach affecting large customer base | Payment gateway down for all users | 15 minutes | 1 hour |
| **P2 — High** | Major feature broken, significant degradation for subset of users | Login failures for 10% of users | 30 minutes | 4 hours |
| **P3 — Medium** | Feature partially broken, workaround available | Export feature times out for large datasets | 2 hours | 24 hours |
| **P4 — Low** | Cosmetic issue, minor inconvenience, feature request | UI alignment issue on settings page | 8 hours | 72 hours |

#### Stage 4: L1 Resolution Attempt

L1 checks against:
- **Knowledge Base** — Known issues with documented resolutions
- **FAQ & Self-Service** — Common questions with scripted answers
- **Recent Incident Log** — Active/recent incidents that match the symptom
- **Customer Account Health** — Account-specific config issues, plan limits

**Resolution rate target at L1: 60-70%**

#### Stage 5-6: L2 Deep Investigation

L2 performs:
- Log analysis (application + infrastructure logs)
- Reproduction of the issue in staging/sandbox
- Environment-specific debugging (browser, OS, API version)
- Database query analysis for data-specific issues

**Resolution rate target at L2: 80-85% (cumulative with L1)**

#### Stage 7-8: SRE Investigation

SRE investigates across:
- Infrastructure health (CPU, memory, disk, network)
- Service mesh / load balancer configuration
- Database performance (slow queries, connection pool exhaustion)
- Deployment-related regressions (recent releases)
- Configuration drift detection

**Decision tree at SRE:**

```
                    ┌──────────────────────┐
                    │  SRE Receives Issue  │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  Is it Infra-related? │
                    └──────────┬───────────┘
                          YES  │  NO
                    ┌──────────┘  └──────────┐
                    ▼                        ▼
          ┌──────────────┐        ┌──────────────────┐
          │ Apply Infra  │        │ Is it Config /   │
          │ Fix (scale,  │        │ Deployment issue?│
          │ restart,     │        └────────┬─────────┘
          │ failover)    │           YES   │   NO
          └──────┬───────┘        ┌────────┘   └────────┐
                 │                ▼                      ▼
                 │      ┌──────────────┐      ┌──────────────────┐
                 │      │ Rollback /   │      │ Escalate to      │
                 │      │ Config Fix   │      │ Engineering Team │
                 │      └──────┬───────┘      └────────┬─────────┘
                 │             │                       │
                 ▼             ▼                       ▼
          ┌──────────────────────────┐      ┌──────────────────┐
          │ Validate & Update        │      │ Coordinate with  │
          │ Support Team             │      │ Engineering      │
          └──────────────────────────┘      └──────────────────┘
```

#### Stage 9-10: Engineering Fix

- Engineering receives issue with full context package from SRE (logs, traces, timeline, impact)
- Root Cause Analysis (RCA) performed
- Code fix developed, reviewed, tested
- Hotfix deployed via expedited release pipeline (for P1/P2)
- Standard release cycle for P3/P4
- SRE team informed upon deployment

#### Stage 11-14: Resolution & Closure

- SRE validates fix in production (monitoring dashboards, synthetic tests)
- SRE updates Support team with resolution details and customer-facing explanation
- Support team updates customer with resolution and prevention measures
- Customer validates and confirms resolution
- Ticket closed with full audit trail

---

## 5. Workflow Track B — SRE-Initiated (Proactive)

This is the **differentiator** for world-class organizations. Issues are detected and resolved *before* customers notice.

```
  MONITORING SYSTEM          SRE ON-CALL             SRE TEAM              ENGINEERING            CUSTOMER
     │                           │                       │                       │                    │
     │ 1. Anomaly Detected       │                       │                       │                    │
     │   (metric breach /        │                       │                       │                    │
     │    error rate spike /     │                       │                       │                    │
     │    latency degradation)   │                       │                       │                    │
     │──────────────────────────▶│                       │                       │                    │
     │                           │                       │                       │                    │
     │                           │ 2. Acknowledge Alert  │                       │                    │
     │                           │    Open Incident      │                       │                    │
     │                           │    Channel            │                       │                    │
     │                           │                       │                       │                    │
     │                           │ 3. Assess Impact      │                       │                    │
     │                           │    (Customer-facing?   │                       │                    │
     │                           │     Blast radius?)     │                       │                    │
     │                           │                       │                       │                    │
     │                           │ 4a. Self-Healable?    │                       │                    │
     │                           │     YES ──▶ Execute   │                       │                    │
     │                           │     Runbook           │                       │                    │
     │                           │     (auto-scale,      │                       │                    │
     │                           │      restart,         │                       │                    │
     │                           │      failover)        │                       │                    │
     │                           │                       │                       │                    │
     │                           │ 4b. NO ──▶ Engage     │                       │                    │
     │                           │     wider SRE team    │                       │                    │
     │                           │──────────────────────▶│                       │                    │
     │                           │                       │                       │                    │
     │                           │                       │ 5. War Room / Swarm   │                    │
     │                           │                       │    Parallel workstreams│                    │
     │                           │                       │                       │                    │
     │                           │                       │ 6a. Infra Fix Applied │                    │
     │                           │                       │     Issue Mitigated   │                    │
     │                           │                       │                       │           NOT IMPACTED
     │                           │                       │                       │          (no ticket needed)
     │                           │                       │                       │                    │
     │                           │                       │ 6b. Product Bug ──▶   │                    │
     │                           │                       │     Escalate          │                    │
     │                           │                       │───────────────────────▶│                    │
     │                           │                       │                       │                    │
     │                           │                       │                       │ 7. Engg Fix +      │
     │                           │                       │                       │    Deploy           │
     │                           │                       │                       │                    │
     │                           │                       │◀──────────────────────│ 8. Inform SRE      │
     │                           │                       │                       │                    │
     │                           │                       │ 9. Validate &         │                    │
     │                           │                       │    Close Incident     │          NOT IMPACTED
     │                           │                       │                       │          (transparent)
     │                           │                       │                       │                    │
```

### Proactive Detection Capabilities

| Signal | Tool/Method | Threshold Example |
|--------|-------------|-------------------|
| Error rate spike | Datadog / Prometheus + Grafana | > 1% 5xx errors over 5-min window |
| Latency degradation | Distributed tracing (Jaeger/Tempo) | p99 latency > 2x baseline |
| Resource exhaustion | Node exporter + alerts | CPU > 85% for 10 min, Disk > 90% |
| Deployment regression | Canary analysis (Argo Rollouts) | Error rate increase post-deploy |
| Certificate expiry | Cert monitoring | < 30 days to expiry |
| Database health | Slow query log analysis | Queries > 5s, connection pool > 80% |
| Synthetic failures | Synthetic monitoring (Checkly/Datadog) | Uptime check fails 2 consecutive times |
| Customer experience | Real User Monitoring (RUM) | Core Web Vitals degradation |

---

## 6. Detailed State Machine

Every ticket/incident follows this state machine:

```
                                          ┌──────────────┐
                                          │              │
                                    ┌─────│   REOPENED   │◀─────────────────────────┐
                                    │     │              │                           │
                                    │     └──────────────┘                           │
                                    │                                                │
                                    ▼                                                │
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐    ┌──────────┐    │
│          │    │          │    │          │    │              │    │          │    │
│   NEW    │───▶│ TRIAGED  │───▶│   OPEN   │───▶│ IN PROGRESS  │───▶│ PENDING  │    │
│          │    │          │    │          │    │              │    │ (Engg)   │    │
└──────────┘    └──────────┘    └──────────┘    └──────────────┘    └────┬─────┘    │
                                                                        │          │
                                                                        ▼          │
                                                                  ┌──────────┐     │
                                                                  │          │     │
                                                                  │ RESOLVED │─────┘
                                                                  │          │  (if customer
                                                                  └────┬─────┘   rejects)
                                                                       │
                                                                       ▼
                                                                  ┌──────────┐
                                                                  │          │
                                                                  │  CLOSED  │
                                                                  │          │
                                                                  └──────────┘

State Transitions:
  NEW ──────────▶ TRIAGED        : L1 classifies priority and category
  TRIAGED ──────▶ OPEN           : Assigned to resolver (L1/L2/SRE)
  OPEN ─────────▶ IN PROGRESS    : Resolver begins investigation
  IN PROGRESS ──▶ PENDING (Engg) : Escalated to Engineering for code fix
  PENDING ──────▶ RESOLVED       : Fix deployed and validated
  IN PROGRESS ──▶ RESOLVED       : Resolved without Engineering involvement
  RESOLVED ─────▶ CLOSED         : Customer confirms or auto-close after 72h
  RESOLVED ─────▶ REOPENED       : Customer rejects resolution
  REOPENED ─────▶ OPEN           : Re-investigation begins
```

---

## 7. Escalation Matrix

```
┌─────────┬──────────────────┬─────────────────┬──────────────────┬──────────────────┐
│ Priority│  0-15 min        │  15 min - 1 hr  │  1 hr - 4 hr     │  4 hr+           │
├─────────┼──────────────────┼─────────────────┼──────────────────┼──────────────────┤
│         │                  │                 │                  │                  │
│   P1    │  L1 ──▶ SRE     │  SRE ──▶ Engg  │  Engg Lead +     │  VP Engineering  │
│         │  On-Call         │  On-Call        │  SRE Manager     │  + CTO           │
│         │                  │                 │                  │                  │
├─────────┼──────────────────┼─────────────────┼──────────────────┼──────────────────┤
│         │                  │                 │                  │                  │
│   P2    │  L1 ──▶ L2      │  L2 ──▶ SRE    │  SRE ──▶ Engg   │  Engg Lead +     │
│         │                  │                 │                  │  SRE Manager     │
│         │                  │                 │                  │                  │
├─────────┼──────────────────┼─────────────────┼──────────────────┼──────────────────┤
│         │                  │                 │                  │                  │
│   P3    │  L1 Triage       │  L1 ──▶ L2     │  L2 ──▶ SRE     │  SRE ──▶ Engg   │
│         │                  │                 │                  │                  │
├─────────┼──────────────────┼─────────────────┼──────────────────┼──────────────────┤
│         │                  │                 │                  │                  │
│   P4    │  L1 Triage       │  L1 works       │  L1 ──▶ L2      │  L2 ──▶ SRE      │
│         │                  │                 │                  │                  │
└─────────┴──────────────────┴─────────────────┴──────────────────┴──────────────────┘
```

### Auto-Escalation Rules

- If no acknowledgement within SLA window → auto-escalate to next tier
- If P1 ticket created → immediate page to SRE on-call (bypass L1/L2 for known critical paths)
- If 3+ tickets with same symptom in 30 min → auto-create incident, page SRE
- If customer is Enterprise/Premium tier → SLA windows halved

---

## 8. SLA Definitions

### Response SLAs

| Priority | First Response | Status Update Frequency | Resolution Target |
|----------|---------------|------------------------|-------------------|
| P1 | 15 minutes | Every 30 minutes | 1 hour |
| P2 | 30 minutes | Every 2 hours | 4 hours |
| P3 | 2 hours | Every 8 hours | 24 hours |
| P4 | 8 hours | Every 24 hours | 72 hours |

### Internal Handoff SLAs

| Handoff | Maximum Time |
|---------|-------------|
| L1 → L2 escalation | 15 minutes |
| L2 → SRE escalation | 30 minutes |
| SRE → Engineering escalation | 30 minutes |
| Engineering → SRE (fix notification) | Upon deployment completion |
| SRE → Support (resolution update) | 30 minutes after validation |
| Support → Customer (resolution update) | 1 hour after receiving from SRE |

---

## 9. Tooling Stack

Top organizations at 8M+ customer scale typically use:

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                              TOOLING ECOSYSTEM                                   │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │  TICKETING & ITSM                                                          │ │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │ │
│  │  │ Jira Service │  │  Zendesk /   │  │  ServiceNow  │  │  Freshdesk    │   │ │
│  │  │ Management   │  │  Intercom    │  │              │  │               │   │ │
│  │  └─────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │  OBSERVABILITY                                                              │ │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │ │
│  │  │  Datadog /   │  │  Prometheus  │  │  ELK Stack / │  │  Jaeger /     │   │ │
│  │  │  New Relic   │  │  + Grafana   │  │  Splunk      │  │  Tempo        │   │ │
│  │  └─────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │  INCIDENT MANAGEMENT & ON-CALL                                              │ │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │ │
│  │  │  PagerDuty   │  │  OpsGenie    │  │  Incident.io │  │  Rootly       │   │ │
│  │  └─────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │  COMMUNICATION                                                              │ │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │ │
│  │  │  Slack       │  │  MS Teams    │  │  Statuspage  │  │  Email / SMS  │   │ │
│  │  └─────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │  CI/CD & DEPLOYMENT                                                         │ │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐   │ │
│  │  │  GitHub      │  │  ArgoCD /    │  │  Terraform   │  │  Kubernetes   │   │ │
│  │  │  Actions     │  │  Spinnaker   │  │              │  │               │   │ │
│  │  └─────────────┘  └──────────────┘  └──────────────┘  └───────────────┘   │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Observability & Alerting Framework

### The Three Pillars + Extras

```
┌────────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY FRAMEWORK                          │
│                                                                    │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────┐  │
│   │  METRICS  │   │   LOGS   │   │  TRACES  │   │  PROFILING   │  │
│   │           │   │          │   │          │   │              │  │
│   │ Prometheus│   │ ELK /    │   │ Jaeger / │   │ Continuous   │  │
│   │ Datadog   │   │ Splunk / │   │ Tempo /  │   │ Profiler     │  │
│   │ CloudWatch│   │ Loki     │   │ Zipkin   │   │ (Pyroscope)  │  │
│   └─────┬────┘   └────┬─────┘   └────┬─────┘   └──────┬───────┘  │
│         │              │              │                 │          │
│         └──────────────┼──────────────┼─────────────────┘          │
│                        │              │                            │
│                        ▼              ▼                            │
│              ┌──────────────────────────────┐                     │
│              │     CORRELATION ENGINE       │                     │
│              │  (Unified dashboards,        │                     │
│              │   cross-signal correlation)  │                     │
│              └──────────────┬───────────────┘                     │
│                             │                                     │
│                             ▼                                     │
│              ┌──────────────────────────────┐                     │
│              │     ALERTING ENGINE          │                     │
│              │  PagerDuty / OpsGenie        │                     │
│              │  Slack / Email / SMS         │                     │
│              └──────────────────────────────┘                     │
└────────────────────────────────────────────────────────────────────┘
```

### Alert Severity Mapping

| Severity | Condition | Action | Notification |
|----------|-----------|--------|-------------|
| **SEV-1 (Critical)** | Service fully down or data loss | Immediate page, war room | PagerDuty + Slack + Phone |
| **SEV-2 (Major)** | Significant degradation, partial outage | Page on-call SRE | PagerDuty + Slack |
| **SEV-3 (Warning)** | Performance degradation, approaching threshold | Alert SRE channel | Slack channel notification |
| **SEV-4 (Info)** | Informational, trend deviation | Dashboard annotation | Slack bot message |

### Key Alerts for 8M+ Customer Scale

| Alert Name | Condition | Runbook |
|-----------|-----------|---------|
| High Error Rate | 5xx > 1% for 5 min | Scale pods, check recent deploys, rollback if needed |
| Latency Spike | p99 > 3s for 10 min | Check DB connections, cache hit rates, upstream deps |
| Pod CrashLoop | Pod restart > 3 in 5 min | Check OOM, config, secrets, image pull |
| DB Connection Pool Exhausted | Active connections > 90% | Kill idle connections, scale read replicas |
| Certificate Expiring | < 14 days to expiry | Trigger cert renewal pipeline |
| Disk Space Critical | > 90% utilization | Clean logs, expand volume, alert infra |
| Queue Backlog Growing | Consumer lag > 10K for 15 min | Scale consumers, check for poison messages |
| Deployment Canary Failure | Canary error rate > baseline + 5% | Auto-rollback, notify release engineer |

---

## 11. Incident Management Lifecycle

For large-scale incidents (SEV-1 / SEV-2):

```
  ┌──────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────────┐
  │          │    │              │    │             │    │              │
  │ DETECT   │───▶│   RESPOND    │───▶│  MITIGATE   │───▶│   RESOLVE    │
  │          │    │              │    │             │    │              │
  └──────────┘    └──────────────┘    └─────────────┘    └──────┬───────┘
                                                                │
                                                                ▼
  ┌──────────┐    ┌──────────────┐    ┌─────────────┐    ┌──────────────┐
  │          │    │              │    │             │    │              │
  │  IMPROVE │◀───│   REVIEW     │◀───│  DOCUMENT   │◀───│  COMMUNICATE │
  │          │    │  (Blameless   │    │  (Timeline, │    │  (Statuspage,│
  └──────────┘    │   Post-Mortem)│    │   RCA)      │    │   Customers) │
                  └──────────────┘    └─────────────┘    └──────────────┘
```

### Incident Roles (for SEV-1 / SEV-2)

| Role | Responsibility |
|------|---------------|
| **Incident Commander (IC)** | Owns the incident end-to-end, makes decisions, coordinates teams |
| **Communications Lead** | Updates statuspage, internal channels, customer-facing comms |
| **Operations Lead** | Hands-on debugging and mitigation (SRE) |
| **Subject Matter Expert (SME)** | Engineering team member with domain knowledge |
| **Scribe** | Documents timeline, actions, decisions in real-time |

### Incident Communication Cadence

| Audience | Channel | Frequency (P1) |
|----------|---------|----------------|
| Internal engineering | Slack war room | Continuous |
| SRE leadership | Slack incident channel | Every 15 min |
| Support team | Dedicated bridge + Slack | Every 30 min |
| Customers (affected) | Statuspage + Email | Every 30 min |
| Executive leadership | Email + Slack DM | Every 1 hour |

---

## 12. Communication Protocols

### Between Teams

```
┌────────────────────────────────────────────────────────────────────────────┐
│                       COMMUNICATION FLOW                                   │
│                                                                            │
│   Customer ◀───▶ Support Team                                              │
│                      │                                                     │
│                      │  Ticket system (Zendesk/Jira)                       │
│                      │  + Email updates                                    │
│                      │                                                     │
│              Support Team ◀───▶ SRE Team                                   │
│                                    │                                       │
│                                    │  Slack channel: #sre-support-bridge   │
│                                    │  + Ticket linking                     │
│                                    │  + Shared dashboards                  │
│                                    │                                       │
│                             SRE Team ◀───▶ Engineering Team                │
│                                                │                           │
│                                                │  Slack: #sre-eng-collab   │
│                                                │  JIRA linked issues       │
│                                                │  War room (for incidents) │
│                                                │  GitHub PRs for fixes     │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

### Notification Templates

**Support → Customer (Issue Acknowledged)**
> Your ticket [TICKET-ID] has been received and is being investigated. Our team is actively working on this and we'll provide an update within [SLA_TIME]. Current priority: [P1/P2/P3/P4].

**SRE → Support (Resolution Found)**
> Issue [TICKET-ID] has been resolved. Root cause: [BRIEF_RCA]. Resolution: [WHAT_WAS_DONE]. Customer impact duration: [DURATION]. No further action needed from customer.

**SRE → Engineering (Escalation)**
> Escalating [TICKET-ID] / Incident [INC-ID]. Impact: [X] customers affected. Symptom: [DESCRIPTION]. Investigation so far: [FINDINGS]. Suspected component: [SERVICE/MODULE]. Relevant logs/traces attached.

---

## 13. Post-Incident Review (PIR)

Every P1/P2 incident and every P3 with >100 customers affected gets a blameless post-incident review.

### PIR Timeline

| Milestone | Deadline |
|-----------|----------|
| Incident resolved | T+0 |
| Initial timeline documented | T+24 hours |
| PIR document drafted | T+3 business days |
| PIR meeting conducted | T+5 business days |
| Action items assigned and tracked | T+5 business days |
| Action items completed | T+30 days (tracked in sprint) |

### PIR Template Structure

```
1. Incident Summary
   - Duration, impact, severity
   - Number of customers affected
   - Revenue impact (if applicable)

2. Timeline
   - Detection time
   - Response time
   - Key milestones during incident
   - Resolution time

3. Root Cause Analysis
   - What failed and why
   - Contributing factors
   - 5-Whys analysis

4. What Went Well
   - Effective processes / tools
   - Quick wins

5. What Needs Improvement
   - Gaps in detection
   - Response bottlenecks
   - Communication issues

6. Action Items
   - Preventive measures
   - Detective improvements
   - Process improvements
   - Each item: Owner, Priority, Due Date
```

---

## 14. Metrics & KPIs

### Support Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| First Response Time | Within SLA | Avg time to first response by priority |
| Resolution Time | Within SLA | Avg time to resolution by priority |
| L1 Resolution Rate | 60-70% | % of tickets resolved at L1 |
| Customer Satisfaction (CSAT) | > 4.5/5 | Post-resolution survey |
| Ticket Reopen Rate | < 5% | % of tickets reopened within 7 days |
| Escalation Rate to SRE | < 15% | % of tickets that reach SRE |

### SRE Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| MTTD (Mean Time to Detect) | < 5 min | Time from issue start to alert firing |
| MTTA (Mean Time to Acknowledge) | < 5 min | Time from alert to human acknowledgement |
| MTTR (Mean Time to Resolve) | < 1 hr (P1) | Time from detection to full resolution |
| Availability (SLO) | 99.95% | Uptime measured by synthetic + real user monitoring |
| Error Budget Remaining | > 20% | Rolling 30-day error budget consumption |
| Proactive vs Reactive Ratio | > 60% proactive | % of incidents caught before customer report |
| Toil Ratio | < 30% | % of SRE time spent on repetitive manual work |
| PIR Action Item Completion | > 90% | % of post-incident actions completed on time |

### Engineering Metrics (Support-Related)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Hotfix Deployment Time | < 2 hrs (P1) | Time from code fix to production deployment |
| Bug Recurrence Rate | < 5% | Same root cause appearing in multiple incidents |
| Escaped Defect Rate | Trending down | Production bugs per release |

---

## 15. Runbook Template

Every known issue class should have a runbook. Template:

```
┌──────────────────────────────────────────────────────────────┐
│                        RUNBOOK                                │
│                                                               │
│  Title:        [Issue Description]                            │
│  Service:      [Affected Service Name]                        │
│  Severity:     [SEV-1 / SEV-2 / SEV-3 / SEV-4]              │
│  Last Updated: [Date]                                         │
│  Author:       [Name]                                         │
│                                                               │
│  ─────────────────────────────────────────────────────────── │
│                                                               │
│  SYMPTOMS:                                                    │
│    - [What the alert looks like]                              │
│    - [What customers might report]                            │
│    - [Dashboard indicators]                                   │
│                                                               │
│  IMPACT:                                                      │
│    - [Which customers / features are affected]                │
│    - [Revenue / business impact]                              │
│                                                               │
│  INVESTIGATION STEPS:                                         │
│    1. [Check dashboard X]                                     │
│    2. [Run query Y]                                           │
│    3. [Inspect logs for Z]                                    │
│    4. [Check recent deployments]                              │
│                                                               │
│  MITIGATION STEPS:                                            │
│    1. [Immediate action - e.g., scale up, restart]            │
│    2. [Apply workaround if available]                         │
│    3. [Rollback if deployment-related]                        │
│                                                               │
│  RESOLUTION STEPS:                                            │
│    1. [Permanent fix procedure]                               │
│    2. [Validation steps]                                      │
│    3. [Communication template]                                │
│                                                               │
│  ESCALATION:                                                  │
│    - If not resolved in [X min]: escalate to [Team/Person]    │
│    - Engineering contact: [Name / Slack handle]               │
│                                                               │
│  RELATED INCIDENTS:                                           │
│    - [INC-XXX] - [Date] - [Brief description]                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary: What Makes This World-Class

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   WORLD-CLASS SRE SUPPORT WORKFLOW (8M+ CUSTOMERS)                       │
│                                                                          │
│   1. DUAL-TRACK MODEL                                                    │
│      Reactive (customer-initiated) + Proactive (SRE-detected)            │
│                                                                          │
│   2. TIERED RESOLUTION                                                   │
│      L1 (60-70%) → L2 (15-20%) → SRE (10-15%) → Engg (5%)              │
│                                                                          │
│   3. SLA-DRIVEN ESCALATION                                               │
│      Auto-escalation ensures no ticket falls through the cracks          │
│                                                                          │
│   4. BLAMELESS POST-INCIDENT CULTURE                                     │
│      Every major incident drives systemic improvement                    │
│                                                                          │
│   5. OBSERVABILITY-FIRST                                                 │
│      Metrics + Logs + Traces + Profiling = Full visibility               │
│                                                                          │
│   6. AUTOMATION & RUNBOOKS                                               │
│      Reduce toil, speed up resolution, ensure consistency                │
│                                                                          │
│   7. CLEAR OWNERSHIP & HANDOFFS                                          │
│      Every stage has a defined owner and SLA for handoff                  │
│                                                                          │
│   8. CUSTOMER-TRANSPARENT COMMUNICATION                                  │
│      Proactive updates, statuspage, honest timelines                     │
│                                                                          │
│   9. CONTINUOUS IMPROVEMENT                                              │
│      PIR action items, error budget policies, toil reduction             │
│                                                                          │
│  10. ENGINEERING FEEDBACK LOOP                                           │
│      Production issues feed back into development priorities             │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

*This document reflects best practices from organizations like Google, Netflix, Amazon, Uber, Shopify, and Atlassian adapted for an 8M+ customer product support context.*
