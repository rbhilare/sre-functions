# Session 6: Inside SRE — Ask & Discuss Anything

**Duration:** 30 minutes | **Format:** Open Q&A and discussion
**Series:** SRE Best Practices — How We Do Things (Session 6 of 14)
**Covers:** Sessions 4–5 follow-up

---

## Purpose

Open forum for the partner team to ask questions, challenge ideas, and share their own experiences related to topics covered in Sessions 4–5:

- **Session 4:** How We Run SRE — team structure, dual operating model, culture principles, on-call, incident response, blameless culture, anti-patterns
- **Session 5:** How We Operate OCP Clusters — cluster access, lifecycle operations, node maintenance, upgrades, diagnostic data collection

---

## Format

| Time | Activity |
|------|----------|
| 5 min | Quick recap of Sessions 4–5 key takeaways |
| 15 min | Open Q&A — anything from the previous sessions |
| 5 min | Discussion — "How does your team handle this?" |
| 5 min | Action items and preview of Sessions 7–8 |

---

## Quick Recap — Sessions 4–5 Key Takeaways

### From Session 4: How We Run SRE

- Every SRE operates in two modes: **reactive** (on-call, incidents) and **proactive** (engineering, automation)
- We follow five culture principles: It's OK to fail, Assume positive intent, Start with trust, Disagree and commit, Communicate
- On-call is PagerDuty-driven with automated investigation before SRE engagement
- Blameless culture is practiced through structured post-incident reviews, not just words
- We eliminated anti-patterns like hero culture, tribal knowledge, and manual toil

### From Session 5: How We Operate OCP Clusters

- Cluster access follows least-privilege with per-command elevation and full audit trail
- Node maintenance follows the standard OCP workflow: cordon → drain → maintain → uncordon → verify
- We prefer replacing nodes over debugging them — fresh machine, fresh OS, clean state
- Upgrades are monitored stage by stage — pre-health check through completion
- must-gather is collected during incidents, not after

---

## Discussion Prompts

**On SRE team structure and culture:**

- How is your SRE or operations team structured today? Do you have a similar dual operating model (reactive + proactive)?
- Which of the five culture principles resonated with you? Are there any you'd add or change?
- How do you handle blameless post-incident reviews? Do you have a formal process?

**On OCP operations:**

- How do you manage cluster access today? Shared kubeconfigs, RBAC groups, or something else?
- What does your node maintenance process look like? Do you follow cordon → drain → uncordon?
- How do you handle upgrades? Who decides when to upgrade, and how do you validate success?
- Have you used must-gather before? Is it part of your incident response runbooks?

**On adoption:**

- Which practices from Sessions 4–5 could you adopt right away?
- What is your biggest operational pain point right now?
- Are there areas where you'd like us to go deeper in upcoming sessions?

