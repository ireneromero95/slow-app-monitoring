# slow-app-monitoring

Black-box monitoring of a deliberately slow Node.js microservice using Prometheus, Alertmanager, cAdvisor and Nginx — with email alerts via SMTP.

---

## Stack

- **App**: `devops.noderino` (branch `slow`) — ~6s response time
- **Load balancer**: Nginx (round-robin across 3 replicas)
- **Metrics**: nginx-prometheus-exporter + cAdvisor → Prometheus
- **Alerts**: Alertmanager → Gmail SMTP

---

## Load generator math

The loadgen sends 1 request every 0.2s (~5 req/s).

```
in-flight ≈ rate × service_time = 5 req/s × 6s = 30 connections
```

`HighInFlightConnections` fires when `sum(reading + writing) > 20` for 10s.

---

## Memory limit

Each replica is limited to **128 MiB**. Under load the working set reaches ~60–100 MiB, giving a ratio of ~0.5–0.8.

`HighMemoryUsage` fires when `working_set / limit > 0.1` for 10s.

---

## Known macOS limitation

On macOS with Docker Desktop, cAdvisor does not attach Docker Compose service labels (`container_label_com_docker_compose_service`) to container metrics. This is because Docker Desktop runs containers inside a Linux VM where cAdvisor cannot access Docker daemon metadata in the same way as on a native Linux host.

As a workaround, the `HighMemoryUsage` alert filters by `id=~"/docker/.*"` instead of by service name. On a native Linux host the original label-based filter would work as intended.

---

## Quick start

```bash
git clone -b slow https://github.com/professordiogodev/devops.noderino app
docker compose --compatibility up --build -d
```

- App: http://localhost:8000/work
- Prometheus: http://localhost:9090
- Alertmanager: http://localhost:9093
