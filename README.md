# Chaos-Resilient Pre-Deployment Simulator

> An end-to-end chaos simulation platform that validates microservice resilience **before** code ships to production — automated inside CI/CD, with full observability.

[![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=flat-square&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Chaos](https://img.shields.io/badge/Chaos-Engineering-FF4F00?style=flat-square)](https://principlesofchaos.org/)
[![Observability](https://img.shields.io/badge/Observability-Prometheus%20%2B%20Grafana-E6522C?style=flat-square&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Containers](https://img.shields.io/badge/Containers-Docker%20Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Python](https://img.shields.io/badge/Language-Python%20%2F%20Flask-3776AB?style=flat-square&logo=python&logoColor=white)](https://flask.palletsprojects.com/)

---

## 📌 Problem Statement

Most teams discover resilience failures **in production** — after a deployment has already caused an outage. Traditional pre-deployment testing validates functionality, not failure recovery.

This platform shifts resilience validation **left** — injecting real-world failure conditions (network delays, service crashes, CPU spikes, database failures) into a production-like environment *before* code is ever deployed. If the system can't recover under chaos, it doesn't ship.

---

## ⚡ Key Outcomes

| Metric | Result |
|--------|--------|
| Failure modes tested per run | **5 chaos scenarios** |
| Integration into CI/CD | **Automated on every push** |
| Observability coverage | **Metrics, latency, error rates** |
| Environment fidelity | **Production-like (Docker Compose)** |
| Resilience report | **Auto-generated per run** |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────┐
│           GitHub Actions              │
│     (Pre-Deployment Chaos Gate)       │
│                                       │
│  on: push → main                      │
│  1. Spin up environment               │
│  2. Run chaos scenarios               │
│  3. Validate recovery                 │
│  4. Generate resilience report        │
│  5. Pass or fail the deployment gate  │
└─────────────────┬────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────┐
│       Docker Compose Orchestrator     │
│                                       │
│  ┌─────────────────┐  ┌────────────────────┐
│  │  Service A      │  │  Service B         │
│  │  (API Layer)    │◄►│  (Worker Layer)    │
│  │                 │  │                    │
│  │  · Flask API    │  │  · Flask + SQLite  │
│  │  · Orchestrates │  │  · Compute logic   │
│  │  · Calls Svc B  │  │  · Simulated DB    │
│  └────────┬────────┘  └────────────────────┘
│           │                                   │
└───────────┼───────────────────────────────────┘
            │
            ▼
┌──────────────────────────────────────┐
│           Chaos Engine                │
│                                       │
│  · Random service kills & restarts   │
│  · Network latency injection          │
│  · CPU and memory spike simulation   │
│  · Temporary database failures        │
│  · Error propagation tracking         │
└─────────────────┬────────────────────┘
                  │
                  ▼
┌──────────────────────────────────────┐
│           Observability Stack         │
│                                       │
│  · Prometheus — metrics collection   │
│  · Grafana — dashboards & alerting   │
│  · Python Dash — local visualization │
│  · /reports — generated per CI run   │
└──────────────────────────────────────┘
```

---

## 🔥 Chaos Scenarios

Each scenario simulates a real-world failure class that production systems routinely face:

| Scenario | What It Does | What It Validates |
|----------|-------------|-------------------|
| **Service Kill** | Randomly terminates Service A or B | Restart policies, dependency recovery |
| **Network Latency** | Injects configurable delays between services | Timeout handling, retry logic |
| **CPU Spike** | Drives CPU to saturation | Performance degradation graceful handling |
| **Memory Spike** | Exhausts available memory | OOM resilience, garbage collection |
| **Database Failure** | Simulates SQLite errors and timeouts | Data layer error handling, fallback logic |

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| CI/CD | GitHub Actions | Pipeline orchestration and chaos gating |
| Containers | Docker Compose | Production-like local environment |
| Service A | Python / Flask | API orchestrator layer |
| Service B | Python / Flask + SQLite | Worker layer with data persistence |
| Chaos Engine | Python scripts | Failure injection across all layers |
| Metrics | Prometheus | Request counts, latency, error rates |
| Dashboards | Grafana | Visualisation and alerting |
| Local Viz | Python Dash | Quick local resilience dashboard |
| Reports | Generated JSON/HTML | Per-run resilience audit trail |

---

## 📁 Project Structure

```
Pre-Deployment-Chaos-Testing-System/
├── .github/
│   └── workflows/
│       └── chaos-test.yml          # CI/CD chaos gate pipeline
├── services/
│   ├── service_a/
│   │   ├── app.py                  # Flask API orchestrator
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   └── service_b/
│       ├── app.py                  # Flask worker + SQLite
│       ├── Dockerfile
│       └── requirements.txt
├── chaos/
│   ├── kill_service.py             # Random service termination
│   ├── network_delay.py            # Latency injection
│   ├── cpu_spike.py                # CPU saturation
│   ├── memory_spike.py             # Memory exhaustion
│   └── db_failure.py               # Database error simulation
├── observability/
│   ├── prometheus.yml              # Metrics scrape config
│   └── grafana/
│       └── dashboards/             # Pre-built Grafana dashboards
├── dashboard/
│   └── app.py                      # Python Dash local visualisation
├── reports/                        # Auto-generated resilience reports
├── docker-compose.yml              # Full environment definition
└── README.md
```

---

## 🚀 How It Works

### CI/CD Chaos Gate

Every push to `main` triggers the chaos pipeline. The deployment is **blocked** if resilience thresholds are not met.

```yaml
# .github/workflows/chaos-test.yml
name: Pre-Deployment Chaos Gate

on:
  push:
    branches: [main]

jobs:
  chaos-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Start environment
        run: docker compose up -d --build

      - name: Wait for services to stabilise
        run: sleep 15

      - name: Run chaos scenarios
        run: |
          python chaos/kill_service.py
          python chaos/network_delay.py
          python chaos/cpu_spike.py
          python chaos/memory_spike.py
          python chaos/db_failure.py

      - name: Validate recovery
        run: python chaos/validate_recovery.py

      - name: Generate resilience report
        run: python chaos/generate_report.py

      - name: Upload report artifact
        uses: actions/upload-artifact@v3
        with:
          name: resilience-report
          path: reports/
```

### Chaos Injection Example

```python
# chaos/kill_service.py
# Simulates unexpected service termination and validates restart recovery

import subprocess
import time
import requests

TARGET = "service_a"
HEALTH_ENDPOINT = "http://localhost:5000/health"
MAX_RECOVERY_SECONDS = 30

def kill_and_validate():
    print(f"[CHAOS] Terminating {TARGET}...")
    subprocess.run(["docker", "compose", "stop", TARGET], check=True)
    time.sleep(5)

    print(f"[CHAOS] Restarting {TARGET}...")
    subprocess.run(["docker", "compose", "start", TARGET], check=True)

    # Validate recovery within threshold
    for attempt in range(MAX_RECOVERY_SECONDS):
        try:
            response = requests.get(HEALTH_ENDPOINT, timeout=2)
            if response.status_code == 200:
                print(f"[PASS] {TARGET} recovered in {attempt + 1}s")
                return True
        except requests.exceptions.ConnectionError:
            time.sleep(1)

    raise Exception(f"[FAIL] {TARGET} did not recover within {MAX_RECOVERY_SECONDS}s")

if __name__ == "__main__":
    kill_and_validate()
```

### Service Observability

Both services expose Prometheus metrics automatically:

```python
# Metrics exposed by each service
# - http_requests_total (counter)
# - http_request_duration_seconds (histogram)
# - service_errors_total (counter)
# - db_operation_duration_seconds (histogram)
```

---

## 📊 Observability

### Prometheus

Metrics are scraped from both services every 15 seconds:

```yaml
# observability/prometheus.yml
scrape_configs:
  - job_name: 'service_a'
    static_configs:
      - targets: ['service_a:5000']

  - job_name: 'service_b'
    static_configs:
      - targets: ['service_b:5001']
```

### Grafana Dashboards

Pre-built dashboards track:
- Request rate and error rate during chaos injection
- P50/P95/P99 latency before, during, and after each scenario
- Recovery time per chaos event
- Service dependency health over time

### Local Dashboard (Python Dash)

For rapid local iteration without Grafana:

```bash
python dashboard/app.py
# Opens at http://localhost:8050
```

---

## ⚙️ Running Locally

### Prerequisites
- Docker and Docker Compose
- Python >= 3.10

### Start the full environment

```bash
git clone https://github.com/blessing-phiri/Pre-Deployment-Chaos-Testing-System.git
cd Pre-Deployment-Chaos-Testing-System

# Start all services
docker compose up -d --build

# Verify services are healthy
curl http://localhost:5000/health  # Service A
curl http://localhost:5001/health  # Service B
```

### Run chaos scenarios manually

```bash
# Individual scenarios
python chaos/kill_service.py
python chaos/network_delay.py
python chaos/cpu_spike.py
python chaos/memory_spike.py
python chaos/db_failure.py

# Full suite with report generation
python chaos/run_all.py
```

### View metrics

```bash
# Prometheus: http://localhost:9090
# Grafana:    http://localhost:3000  (admin / admin)
# Dash:       http://localhost:8050
```

### Tear down

```bash
docker compose down -v
```

---

## 📋 Resilience Report

Each CI run generates a report in `/reports` capturing:

```json
{
  "run_id": "2024-03-15T14:32:00Z",
  "commit": "abc1234",
  "scenarios": [
    {
      "name": "service_kill",
      "status": "PASS",
      "recovery_time_seconds": 8,
      "threshold_seconds": 30
    },
    {
      "name": "network_latency",
      "status": "PASS",
      "p99_latency_ms": 420,
      "threshold_ms": 1000
    },
    {
      "name": "cpu_spike",
      "status": "PASS",
      "degradation_percent": 34,
      "threshold_percent": 80
    }
  ],
  "overall": "PASS",
  "deployment_gate": "OPEN"
}
```

---

## 🔮 Future Improvements

- [ ] Migrate from Docker Compose to Kubernetes with Chaos Mesh for production-grade injection
- [ ] Add network partition scenarios (split-brain simulation)
- [ ] Implement configurable resilience thresholds per environment
- [ ] Integrate with PagerDuty or Slack for chaos run notifications
- [ ] Add historical trend tracking across multiple runs
- [ ] Replace SQLite with PostgreSQL for more realistic data layer testing

---

## 💡 Why This Matters

Chaos engineering is standard practice at Netflix, Amazon, and Google — but rarely adopted at the pre-deployment stage. This project demonstrates:

- **Shift-left resilience** — finding failure modes before they reach production
- **Automated quality gates** — chaos results block or allow deployment, not a human decision
- **Observability-first thinking** — every failure is measured, not just observed
- **Production fidelity** — testing against realistic failure modes, not synthetic happy paths

---

## 👤 Author

**Blessing Phiri** — Platform Engineer | RHCA | Kubestronaut

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/blessing-phiri-614b77209/)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=flat-square&logo=github&logoColor=white)](https://github.com/blessing-phiri)
[![Website](https://img.shields.io/badge/Website-000000?style=flat-square&logo=About.me&logoColor=white)](https://blessingphiri.dev)
