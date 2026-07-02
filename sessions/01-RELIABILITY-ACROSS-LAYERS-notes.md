# Session 1: Reliability Across Layers — Speaker Notes

---

### Slide 1: Title

- Welcome the audience and introduce yourself.
- Set the context: this session is about how reliability is a shared responsibility across code, configuration, and platform — not just an ops problem or a dev problem.
- Mention the 30-minute format — focused and practical.

---

### Slide 2: Agenda

- Walk through the agenda briefly so the audience knows what to expect.
- Emphasize that this is not just theory — every pattern shown is something we use or have used in production.
- Let them know there will be time for questions at the end, but they can ask inline if something is unclear.

---

### Slide 3: What is Reliability?

- Start with the industry definition — reliability is about probability over time, not just "is it up right now."
- Explain the three dimensions: availability is table stakes, but performance and correctness matter just as much. A system that returns wrong results at 100% uptime is not reliable.
- Walk through the nines table — most teams think they need five nines, but three nines (8.8 hours/year) is already quite demanding. Ask the audience what their current target is.

---

### Slide 4: SLI, SLO, SLA Framework

- Explain the relationship bottom-up: SLIs are what you measure, SLOs are what you aim for internally, SLAs are what you promise externally.
- Key point: your SLO should always be tighter than your SLA, otherwise you have no buffer.
- Give a concrete example: "We measure request latency (SLI), target 99.9% of requests under 100ms (SLO), and guarantee 99.5% in our contract (SLA)."

---

### Slide 5: Error Budget & MTTR/MTTF

- Walk through each metric and why it matters. MTTR is the one most teams can improve fastest — it's about detection + diagnosis + fix.
- The error budget math is critical: if your SLO is 99.9%, you only get 43.2 minutes per month. That's less than one bad deployment.
- Explain the policy: when the budget is exhausted, we freeze risky changes. This is how reliability and velocity coexist — velocity is the default, stability is the circuit breaker.

---

### Slide 6: Code Layer — Defensive Programming

- Show the bad vs. good example. The bad version will crash with a ZeroDivisionError — in production, this means a 500 error for the user.
- The good version handles the edge case and validates input types. It is not just about catching exceptions — it is about anticipating what can go wrong.
- Emphasize: defensive programming is the cheapest reliability investment. It costs minutes during development and saves hours during incidents.

---

### Slide 7: Code Layer — Circuit Breaker Pattern

- Explain the circuit breaker analogy: just like an electrical circuit breaker prevents a fire, this pattern prevents cascading failures.
- After 5 consecutive failures, the breaker opens and stops sending requests to the failing service. After 60 seconds, it tries again (half-open state).
- Key point: without a circuit breaker, one failing dependency can bring down your entire system through resource exhaustion (threads, connections, memory).

---

### Slide 8: Code Layer — Retry with Exponential Backoff

- Retries are essential but dangerous if done wrong. Immediate retries at full speed can turn a struggling service into a dead one.
- Exponential backoff spaces out retries: 2s, 4s, 8s, etc. Combined with a max attempt limit, this gives the downstream service time to recover.
- Mention jitter: in production, we add random jitter to prevent the "thundering herd" problem where all clients retry at the same time.

---

### Slide 9: Code Layer — Graceful Degradation

- This is about providing a reduced but functional experience instead of a complete failure.
- In the example, if the ML recommendation service is down, we fall back to a simpler algorithm. The user still gets recommendations — just not personalized ones.
- Ask the audience: "What is your fallback when your primary service is down? If the answer is 'an error page,' there is room to improve."

---

### Slide 10: Code Layer — Timeouts

- The simplest and most impactful reliability pattern. Without timeouts, a slow dependency will hang your threads indefinitely.
- 5 seconds is our default for external API calls. Adjust based on the operation, but always have one.
- Combine with a default response: the user gets a result (maybe cached or simplified) instead of staring at a spinner.

---

### Slide 11: Config Layer — Infrastructure as Code

- IaC means your infrastructure is versioned, reviewed, and reproducible. No more "someone changed something in the console."
- The Terraform example shows auto-scaling with a minimum of 3 instances and health checks — this is reliability built into the config.
- Environment parity is critical: if staging uses different configs than production, you are testing something other than what you ship.

---

### Slide 12: Config Layer — Deployment Strategies & Feature Flags

- Blue-green gives you instant rollback — if the new version has issues, switch traffic back in seconds, not minutes.
- Feature flags decouple deployment from release. You can deploy code without activating it, then turn it on for 1% of users first.
- Together, these mean deployments are no longer scary events. They become routine, which is exactly what you want.

---

### Slide 13: Platform Layer — Redundancy & HA

- Redundancy is the foundation of platform reliability. Single points of failure are the number one cause of outages.
- Distribute across AZs for resilience against data center failures. Distribute across regions for resilience against regional events.
- Load balancers with health checks ensure traffic only goes to healthy instances. DNS failover handles regional outages.

---

### Slide 14: Platform Layer — Self-Healing & Resource Quotas

- Liveness probes let Kubernetes detect and restart unhealthy pods automatically. This handles the "it is stuck" scenario without human intervention.
- Resource quotas prevent noisy neighbors — one team's runaway process cannot consume all cluster resources.
- These are platform-level reliability patterns that protect the entire system, not just individual applications.

---

### Slide 15: Platform Layer — Pod Disruption Budgets

- PDBs ensure that voluntary disruptions (node drains, upgrades) do not take down your service.
- `minAvailable: 2` means Kubernetes will refuse to evict a pod if it would leave fewer than 2 running.
- This is essential during cluster upgrades — without PDBs, a node drain can temporarily take your service offline.

---

### Slide 16: Platform Layer — Service Mesh & Database Reliability

- A service mesh handles cross-cutting reliability concerns (retries, circuit breaking, rate limiting) at the infrastructure layer, so application code does not need to implement them.
- Database reliability requires a primary-replica setup with automated failover. 30-second failover means most users will not notice.
- The key insight: reliability at the platform layer protects all applications running on it, giving you the best return on investment.

---

### Slide 17: Observability — The Three Pillars

- Metrics tell you what is happening (quantitative). Logs tell you why (context). Traces tell you where (flow).
- You need all three. Metrics alone tell you error rate is up, but not which service or which request. Logs tell you the error message, but not the full request path. Traces connect everything.
- Start with metrics for alerting, logs for debugging, and traces for understanding complex distributed systems.

---

### Slide 18: Monitoring & Alerting

- Three alert strategies: threshold-based is the simplest and works for well-understood patterns. Anomaly-based catches things you did not predict. SLO-based is the most mature — it alerts on user impact, not system metrics.
- The quick tip is important: page only for real incidents. If an alert does not require immediate human action, it should be a ticket, not a page.
- Alert fatigue is real — we reduced our alerts from 500 to 80 by applying this principle.

---

### Slide 19: Tools & Technologies — Observability Stack

- This is not a recommendation to use all of these — it is showing the landscape.
- Our stack: Prometheus + Alertmanager for metrics, Loki for logs, Jaeger for traces, Grafana for dashboards. This covers 90% of our needs.
- The choice depends on your environment. In disconnected environments, SaaS options like Datadog are not available, so open-source self-hosted tools are the way.

---

### Slide 20: Tools & Technologies — Testing, Quality & Infra

- Chaos testing is how you validate your reliability patterns actually work. Do not assume the circuit breaker will work in production if you have never triggered it.
- GitOps with ArgoCD is how we manage config layer reliability — everything is in Git, everything is reviewed, everything is auditable.
- Load testing should be part of your CI/CD pipeline, not a one-time activity before launch.

---

### Slide 21: Real-World Example

- Walk through the e-commerce example layer by layer. This is how all three layers work together.
- Code layer handles individual request failures. Config layer handles safe deployments. Platform layer handles infrastructure failures.
- The result — 99.99% availability — is not achieved by any single layer. It is the combination of all three.

---

### Slide 22: Best Practices Summary

- Go through the DO list quickly — these are the patterns we covered today, summarized.
- Spend more time on the DON'T list — these are the anti-patterns we have learned from painful experience.
- "Assume it works in staging" is the most common one. Staging is not production. Traffic patterns, data volumes, and failure modes are different.

---

### Slide 23: Contact & Resources

- Share contact details and invite follow-up questions via email or Slack.
- Point to the GitHub repo for the SRE workflows and additional reference material.

---

### Slide 24: Thank You

- Thank the audience for their time.
- Open the floor for any final questions.
- Preview the next session if applicable.
