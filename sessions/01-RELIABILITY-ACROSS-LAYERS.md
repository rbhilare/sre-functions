# Session 1: Reliability Across Layers

**Code, Config and Platform**

**Presenter:** Rohit Bhilare — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat
**Duration:** 30 minutes
**Series:** SRE Best Practices — How We Do Things (Session 1 of 14)
**Focus:** Building resilient systems

---

### 1: Title

# Reliability Across Layers

**Code, Config and Platform**

Rohit Bhilare — Principal Site Reliability Engineer, Hybrid Platforms — ROSA, Red Hat

---

### 2: Agenda

- What is Reliability? (Definitions & Frameworks)
- Code Layer — Writing Reliable Code
- Config Layer — Configuration Management
- Platform Layer — Infrastructure & Orchestration
- Observability — Monitoring & Alerting
- Tools & Technologies
- Real-world Examples
- Best Practices & Takeaways

---

### 3: What is Reliability?

**Industry Standard:** The probability that a system will function as expected over a given time period.

**Three Dimensions:**

| Dimension | Definition |
|-----------|------------|
| **Availability** | System is up and responding |
| **Performance** | System meets SLO targets |
| **Correctness** | System produces right results |

**Downtime by Availability Target:**

| Target | Downtime/Year |
|--------|---------------|
| 99% (two nines) | 3.7 days |
| 99.9% (three nines) | 8.8 hours |
| 99.99% (four nines) | 52 minutes |
| 99.999% (five nines) | 5.2 minutes |

---

### 4: SLI, SLO, SLA Framework

**Trinity of reliability engineering:**

| Term | What It Is | Example |
|------|-----------|---------|
| **SLA** (Service Level Agreement) | Legal commitment to customers | 99.5% guaranteed |
| **SLO** (Service Level Objective) | Internal target we aim for | 99.9% of requests meet SLI |
| **SLI** (Service Level Indicator) | Actual measurement | Request latency < 100ms, Error rate < 0.1% |

---

### 5: Error Budget & MTTR/MTTF

**Key Metrics:**

| Metric | Definition | Our Target |
|--------|-----------|------------|
| **MTTR** (Mean Time To Repair) | Avg time to fix issues | 15 mins |
| **MTTF** (Mean Time To Failure) | Avg time between failures | 30 days |
| **RTO** (Recovery Time Objective) | Max acceptable downtime | 1 hour |
| **RPO** (Recovery Point Objective) | Max acceptable data loss | 15 mins |

**Error Budget Example (SLO = 99.9%):**

- Monthly error budget = 0.1% × (30 × 24 × 60) = **43.2 minutes** of downtime
- Once exhausted: No risky deployments, freeze feature releases, focus on stability only

---

### 6: Code Layer — Defensive Programming

```python
# BAD - Crashes on invalid input
def calculate_percentage(value, total):
    return (value / total) * 100

# GOOD - Handles edge cases
def calculate_percentage(value, total):
    if total == 0:
        return 0
    if not isinstance(value, (int, float)):
        raise ValueError("Value must be numeric")
    return (value / total) * 100
```

---

### 7: Code Layer — Circuit Breaker Pattern

```python
from pybreaker import CircuitBreaker

breaker = CircuitBreaker(
    fail_max=5,
    reset_timeout=60,
    listeners=[log_listener]
)

@breaker
def call_external_api():
    return requests.get("https://api.example.com")
```

---

### 8: Code Layer — Retry with Exponential Backoff

```python
import tenacity

@tenacity.retry(
    wait=tenacity.wait_exponential(multiplier=1, min=2, max=10),
    stop=tenacity.stop_after_attempt(5),
    retry=tenacity.retry_if_exception_type(ConnectionError)
)
def fetch_data():
    return requests.get("https://api.example.com")
```

---

### 9: Code Layer — Graceful Degradation

```python
def get_user_recommendations():
    try:
        # Attempt to get ML recommendations
        return get_ml_recommendations()
    except Exception as e:
        logger.error(f"ML service failed: {e}")
        # Fall back to simpler logic
        return get_simple_recommendations()
```

---

### 10: Code Layer — Timeouts

```python
try:
    response = requests.get(
        url,
        timeout=5  # Fail fast instead of hang forever
    )
except requests.Timeout:
    return default_response()
```

---

### 11: Config Layer — Infrastructure as Code (IaC)

```hcl
resource "aws_autoscaling_group" "api_servers" {
  min_size         = 3
  max_size         = 10
  desired_capacity = 5
  health_check_type = "ELB"
}
```

**Environment Parity:** Development = Staging = Production (same configs, versions, resources)

---

### 12: Config Layer — Deployment Strategies & Feature Flags

**Blue-Green Deployment:** Old (Blue) → New (Green) → Traffic Switch (Instant rollback)

**Feature Flags:**

```python
if feature_flag("new_payment_system"):
    use_new_payment()
else:
    use_legacy_payment()
```

---

### 13: Platform Layer — Redundancy & High Availability

Load balancer distributing traffic across multiple **Regions** and **Availability Zones (AZs)**.

| Component | Strategy |
|-----------|----------|
| Compute | Multiple replicas across AZs |
| Storage | Cross-region replication |
| Network | Redundant load balancers |
| DNS | Health-checked failover routing |

---

### 14: Platform Layer — Self-Healing & Resource Quotas

**Self-Healing with Liveness Probes:**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
  failureThreshold: 3  # Restart if fails 3 times
```

**Resource Quotas:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: api-quota
spec:
  hard:
    requests.cpu: "100"
    requests.memory: "200Gi"
    pods: "100"
```

---

### 15: Platform Layer — Pod Disruption Budgets

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  minAvailable: 2  # Always keep 2 pods running
  selector:
    matchLabels:
      app: api
```

---

### 16: Platform Layer — Service Mesh & Database Reliability

**Service Mesh (Traffic Management):**

- Automatic retries
- Circuit breaking
- Rate limiting
- Traffic shifting
- Observability

**Database Reliability:**

| Role | Purpose |
|------|---------|
| Primary | Active writes |
| Replica 1 | Read + failover candidate |
| Replica 2 | Read + backup |

Automated failover in < 30 seconds.

---

### 17: Observability — The Three Pillars

| Pillar | Type | What It Captures |
|--------|------|-----------------|
| **Metrics** | Quantitative | CPU, Memory, Disk, Network; Requests/sec, Latency, Error rate, Cache hit |
| **Logs** | Context | INFO, WARN, ERROR logs with timestamps |
| **Traces** | Flow | Request path through services and database queries |

---

### 18: Monitoring & Alerting

**Alert Strategies:**

| Strategy | Examples |
|----------|----------|
| **Threshold-based** | CPU > 80%, Error rate > 1%, Latency P99 > 500ms |
| **Anomaly-based** | Unusual traffic patterns, Unexpected error spikes |
| **SLO-based** | Error budget exhausted, Burn rate > 10x |

> **Quick tip:** Page only for real incidents.

---

### 19: Tools & Technologies — Observability Stack

| Category | Tools |
|----------|-------|
| Metrics | Prometheus, InfluxDB, Datadog |
| Logs | ELK Stack, Loki, Splunk |
| Traces | Jaeger, Zipkin, Datadog |
| Dashboards | Grafana, Kibana |
| Alerting | AlertManager, PagerDuty |

---

### 20: Tools & Technologies — Testing, Quality & Infra

| Category | Tools |
|----------|-------|
| Load Testing | JMeter, Locust, K6 |
| Chaos Testing | Chaos Monkey, Gremlin |
| Security & Performance | OWASP ZAP, SonarQube, New Relic / Dynatrace |
| IaC | Terraform, Pulumi |
| GitOps | ArgoCD, Flux |

---

### 21: Real-World Example — E-Commerce System Reliability

**Result: 99.99% availability maintained**

| Layer | Patterns Applied |
|-------|-----------------|
| **Code** | Timeout on payment API, Circuit breaker, Retry with exponential backoff, Graceful degradation |
| **Config** | Blue-green deployment, Feature flags, Canary (5% traffic) |
| **Platform** | 3 replicas minimum (1 per AZ), Auto-scaling, Database read replicas in 2 regions, RTO: 5 min, RPO: 1 min |

---

### 22: Best Practices Summary

**DO:**

- Write defensive code with proper error handling
- Test failure scenarios (chaos testing)
- Use feature flags for safe deployments
- Monitor the three pillars (metrics, logs, traces)
- Set realistic SLOs based on user impact
- Practice incident response regularly
- Automate everything (deployments, scaling, healing)
- Keep runbooks updated

**DON'T:**

- Deploy all changes at once
- Ignore error budgets
- Skip graceful degradation patterns
- Assume "it works in staging"
- Make manual changes in production
- Delay post-incident reviews
- Neglect documentation
- Ignore security and other risks in reliability

---

### 23: Contact & Resources

**Questions?**

| | |
|---|---|
| **Email** | rbhilare@redhat.com |
| **GitHub** | [SRE Product Support Workflow](https://github.com/rbhilare/sre-functions/blob/main/ideal-sre-workflow/SRE-PRODUCT-SUPPORT-WORKFLOW.md) |

---

### 24: Thank You

> Red Hat is the world's leading provider of enterprise open source software solutions. Award-winning support, training, and consulting services make Red Hat a trusted adviser to the Fortune 500.
