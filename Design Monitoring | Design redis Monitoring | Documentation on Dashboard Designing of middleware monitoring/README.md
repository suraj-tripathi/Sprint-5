# Redis Monitoring Dashboard Design Documentation

# Middleware Monitoring Dashboard Engineering Design

## Document Details

| Author | Created on | Version | Last updated by | Last edited on | Pre Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer |
|--------|------------|---------|-----------------|----------------|--------------|-------------|-------------|-------------|
| Suraj Tripathi | 20-04-2026 | v1.0 | Suraj Tripathi | 13-04-2026 |              | Aniruddh    | Faisal      | Ashwani     |

---

# Table of Contents

- [1. Introduction](#1-introduction)
- [2. What is Middleware Monitoring](#2-what-is-middleware-monitoring)
- [3. Why Monitoring is Important](#3-why-monitoring-is-important)
- [4. Why Redis Monitoring is Critical](#4-why-redis-monitoring-is-critical)
- [5. Monitoring Architecture](#5-monitoring-architecture)
  - [Architecture Diagram](#architecture-diagram)
- [6. Monitoring Flow Design](#6-monitoring-flow-design)
  - [Step-by-Step Monitoring Flow](#step-by-step-monitoring-flow)
- [7. Dashboard Design Principles](#7-dashboard-design-principles)
  - [Dashboard Hierarchy](#dashboard-hierarchy)
  - [Visualization Best Practices](#visualization-best-practices)
- [8. Redis Monitoring Categories](#8-redis-monitoring-categories)
  - [Pillar 1: Performance & Latency](#pillar-1-performance--latency)
  - [Pillar 2: Memory & Saturation](#pillar-2-memory--saturation)
  - [Pillar 3: Connections & Health](#pillar-3-connections--health)
  - [Pillar 4: High Availability & Persistence](#pillar-4-high-availability--persistence)
- [9. Dashboard Layout Design](#9-dashboard-layout-design)
  - [Dashboard Layout](#dashboard-layout)
- [10. Dashboard Panels and Visualizations](#10-dashboard-panels-and-visualizations)
  - [A. Global Health Panels](#a-global-health-panels)
    - [Redis Status](#redis-status)
  - [B. Performance Panels](#b-performance-panels)
    - [Commands Per Second](#commands-per-second)
    - [P99 Latency](#p99-latency)
  - [C. Memory Panels](#c-memory-panels)
    - [Used Memory](#used-memory)
    - [Memory Saturation Percentage](#memory-saturation-percentage)
  - [D. Cache Performance Panels](#d-cache-performance-panels)
    - [Cache Hit Ratio](#cache-hit-ratio)
  - [E. Connection Panels](#e-connection-panels)
    - [Connected Clients](#connected-clients)
    - [Blocked Clients](#blocked-clients)
  - [F. Eviction Monitoring](#f-eviction-monitoring)
    - [Evicted Keys](#evicted-keys)
  - [G. Replication Panels](#g-replication-panels)
    - [Replica Lag](#replica-lag)
  - [H. Persistence Monitoring](#h-persistence-monitoring)
    - [AOF Rewrite Status](#aof-rewrite-status)
- [11. Important Redis Metrics](#11-important-redis-metrics)
- [12. Alerting Strategy](#12-alerting-strategy)
  - [Alert Categories](#alert-categories)
  - [Sample Alert Rules](#sample-alert-rules)
    - [Redis Down](#redis-down)
    - [High Memory Usage](#high-memory-usage)
    - [Replication Lag](#replication-lag)
- [13. Step-by-Step Implementation](#13-step-by-step-implementation)
  - [Step 1: Install Redis](#step-1-install-redis)
  - [Step 2: Verify Redis](#step-2-verify-redis)
  - [Step 3: Install Redis Exporter](#step-3-install-redis-exporter)
  - [Step 4: Install Prometheus](#step-4-install-prometheus)
  - [Step 5: Start Prometheus](#step-5-start-prometheus)
  - [Step 6: Install Grafana](#step-6-install-grafana)
  - [Step 7: Add Prometheus Datasource](#step-7-add-prometheus-datasource)
  - [Step 8: Create Dashboard](#step-8-create-dashboard)
- [14. Advanced Redis Monitoring](#14-advanced-redis-monitoring)
- [15. Troubleshooting](#15-troubleshooting)
  - [Exporter Not Working](#exporter-not-working)
  - [Prometheus Target Failed](#prometheus-target-failed)
  - [Grafana Showing No Data](#grafana-showing-no-data)
- [16. Best Practices](#16-best-practices)
- [17. Future Enhancements](#17-future-enhancements)
- [18. Contact Information](#18-contact-information)
- [19. Conclusion](#19-conclusion)
---

# 1. Introduction

This document explains the complete engineering design for Redis Middleware Monitoring Dashboards using:

- Redis Exporter
- Prometheus
- Grafana
- Alertmanager

The document is written from beginner to advanced level so that even a first-time viewer can understand:

- What monitoring is
- Why monitoring is required
- Dashboard designing concepts
- Redis metrics
- Grafana panel layouts
- Visualization strategy
- Alerting concepts
- Middleware monitoring architecture

---

# 2. What is Middleware Monitoring

Middleware is software that sits between:

- Application Layer
- Database Layer

Examples of middleware:

- Redis
- Kafka
- RabbitMQ

Middleware monitoring means continuously tracking:

- Performance
- Availability
- Resource utilization
- Errors
- Throughput
- Health status

---

# 3. Why Monitoring is Important

Without monitoring:

- Failures cannot be detected quickly
- Performance degradation remains hidden
- Application outages increase
- Troubleshooting becomes difficult
- User experience degrades

Monitoring helps:

| Benefit | Description |
|---|---|
| Early issue detection | Detect failures before outage |
| Performance tracking | Monitor latency and throughput |
| Capacity planning | Predict future scaling |
| Root cause analysis | Identify actual issue |
| SLA monitoring | Maintain uptime |
| Alerting | Notify teams instantly |

---

# 4. Why Redis Monitoring is Critical

Redis is an in-memory database.

Since Redis works directly in RAM:

- Memory saturation is dangerous
- Slow commands affect all requests
- High latency impacts applications immediately
- Replication failures may cause data loss

Redis is single-threaded internally.

A single blocking command can impact the entire server.

---

# 5. Monitoring Architecture

## Architecture Diagram

<img width="1184" height="630" alt="_- visual selection (22)" src="https://github.com/user-attachments/assets/fedb7d09-d5ee-4975-890f-2b6166880878" />


---

# 6. Monitoring Flow Design

## Step-by-Step Monitoring Flow

| Step | Component | Action Performed | Purpose |
|---|---|---|---|
| Step 1 | Redis Server | Redis generates internal performance and health metrics | Provides monitoring data like memory, latency, connections, and throughput |
| Step 2 | Redis Exporter | Redis Exporter collects Redis metrics and exposes them in Prometheus format | Converts Redis metrics into readable monitoring metrics |
| Step 3 | Prometheus | Prometheus scrapes metrics periodically from Redis Exporter | Collects real-time monitoring data |
| Step 4 | Prometheus TSDB | Prometheus stores collected metrics inside TSDB (Time Series Database) | Maintains historical monitoring data |
| Step 5 | Grafana | Grafana queries Prometheus using PromQL | Fetches monitoring data for visualization |
| Step 6 | Grafana Dashboard | Grafana visualizes data using charts, graphs, gauges, and stat panels | Provides real-time operational visibility |
| Step 7 | Alertmanager | Alertmanager sends alerts when thresholds exceed configured limits | Enables proactive issue detection and notification |
---

# 7. Dashboard Design Principles

A middleware dashboard must be:

- Easy to read
- Fast to understand
- Operationally useful
- Real-time
- Minimal but informative

---

## Dashboard Hierarchy

```text
+------------------------------------------------------+
| Tier 1 : Global Health & SLI Metrics                 |
+------------------------------------------------------+
| Tier 2 : Throughput & Saturation                     |
+------------------------------------------------------+
| Tier 3 : Resource Utilization                        |
+------------------------------------------------------+
| Tier 4 : Diagnostics & Logs                          |
+------------------------------------------------------+
```

---

## Visualization Best Practices

| Good Practice | Reason |
|---|---|
| Use line charts | Easy trend visibility |
| Use stat panels | Quick status visibility |
| Use consistent colors | Better readability |
| Avoid pie charts | Hard to interpret |
| Avoid excessive gauges | Consumes space |
| Use drill-down panels | Faster debugging |

---

## 8. Redis Monitoring Categories

Redis monitoring should be divided into 4 major pillars:

---

## Pillar 1: Performance & Latency

Tracks Redis speed and responsiveness.

Important Metrics:

| Metric | Purpose |
|---|---|
| redis_command_latency_seconds_p99 | Tail latency |
| redis_ops_per_second | Throughput |
| keyspace_hit_rate | Cache efficiency |

---

## Pillar 2: Memory & Saturation

Tracks RAM usage and memory pressure.

Important Metrics:

| Metric | Purpose |
|---|---|
| redis_memory_used_bytes | RAM consumption |
| redis_memory_fragmentation_ratio | Memory fragmentation |
| redis_evicted_keys_total | Evicted keys |

---

## Pillar 3: Connections & Health

Tracks client and server health.

Important Metrics:

| Metric | Purpose |
|---|---|
| redis_connected_clients | Active clients |
| redis_blocked_clients | Blocked operations |
| redis_rejected_connections | Failed connections |

---

## Pillar 4: High Availability & Persistence

Tracks replication and persistence health.

Important Metrics:

| Metric | Purpose |
|---|---|
| redis_replica_lag_seconds | Replication lag |
| redis_connected_slaves | Replica count |
| rdb_last_bgsave_status | Backup status |
| aof_last_rewrite_status | AOF health |

---

# 9. Dashboard Layout Design

## Dashboard Layout

```text
+------------------------------------------------------+
|                REDIS OVERVIEW DASHBOARD              |
+------------------------------------------------------+

+-------------+-------------+-------------+------------+
| Redis Status| Uptime      | Role        | Version    |
+-------------+-------------+-------------+------------+

+----------------------+-----------------------------+
| CPU Usage            | Memory Usage                |
+----------------------+-----------------------------+

+----------------------+-----------------------------+
| Connected Clients    | Commands/sec                |
+----------------------+-----------------------------+

+----------------------+-----------------------------+
| Cache Hit Ratio      | Evicted Keys                |
+----------------------+-----------------------------+

+----------------------------------------------------+
| Network Traffic                                    |
+----------------------------------------------------+

+----------------------------------------------------+
| Replication Monitoring                             |
+----------------------------------------------------+

+----------------------------------------------------+
| Redis Logs & Slow Queries                          |
+----------------------------------------------------+
```

---

# 10. Dashboard Panels and Visualizations

## A. Global Health Panels

### Redis Status

#### Visualization:
Stat Panel

#### Query:
```promql
redis_up
```

#### Color Logic:
- Green = UP
- Red = DOWN

---

## B. Performance Panels

### Commands Per Second

#### Visualization:
Time Series

#### Query:
```promql
rate(redis_commands_processed_total[1m])
```

---

### P99 Latency

#### Visualization:
Time Series

#### Query:
```promql
histogram_quantile(0.99, rate(redis_command_call_duration_seconds_bucket[5m]))
```

---

## C. Memory Panels

### Used Memory

#### Visualization:
Gauge

#### Query:
```promql
redis_memory_used_bytes
```

---

### Memory Saturation Percentage

#### Visualization:
Gauge

#### Query:
```promql
(redis_memory_used_bytes / redis_memory_max_bytes) * 100
```

---

## D. Cache Performance Panels

### Cache Hit Ratio

#### Visualization:
Gauge

#### Query:
```promql
sum(rate(redis_keyspace_hits_total[5m]))
/
(
sum(rate(redis_keyspace_hits_total[5m]))
+
sum(rate(redis_keyspace_misses_total[5m]))
)
* 100
```

---

## E. Connection Panels

### Connected Clients

#### Visualization:
Stat + Time Series

#### Query:
```promql
redis_connected_clients
```

---

### Blocked Clients

#### Visualization:
Time Series

#### Query:
```promql
redis_blocked_clients
```

---

## F. Eviction Monitoring

### Evicted Keys

#### Visualization:
Time Series

#### Query:
```promql
rate(redis_evicted_keys_total[5m])
```

---

## G. Replication Panels

### Replica Lag

#### Visualization:
Time Series

#### Query:
```promql
redis_replica_lag_seconds
```

---

## H. Persistence Monitoring

### AOF Rewrite Status

#### Visualization:
Stat Panel

#### Query:
```promql
redis_aof_last_rewrite_status
```

---

# 11. Important Redis Metrics

| Metric | Warning Threshold |
|---|---|
| Memory Usage | >75% |
| Cache Hit Ratio | <90% |
| Replication Lag | >10 sec |
| P99 Latency | >4ms |
| Rejected Connections | >0 |
| Evicted Keys | >0 |

---

# 12. Alerting Strategy

## Alert Categories

| Severity | Meaning |
|---|---|
| Warning | Needs attention |
| Critical | Immediate action required |

---

## Sample Alert Rules

### Redis Down

```yaml
- alert: RedisDown
  expr: redis_up == 0
  for: 1m
```

---

### High Memory Usage

```yaml
- alert: RedisHighMemory
  expr: (redis_memory_used_bytes / redis_memory_max_bytes) > 0.85
```

---

### Replication Lag

```yaml
- alert: RedisReplicationLag
  expr: redis_replica_lag_seconds > 10
```

---

# 13. Step-by-Step Implementation

## Step 1: Install Redis

```bash
sudo apt update
sudo apt install redis-server -y
```

---

## Step 2: Verify Redis

```bash
redis-cli ping
```

Expected Output:

```text
PONG
```

---

## Step 3: Install Redis Exporter

```bash
wget https://github.com/oliver006/redis_exporter/releases/download/v1.62.0/redis_exporter-v1.62.0.linux-amd64.tar.gz
```

Extract:

```bash
tar -xvf redis_exporter-v1.62.0.linux-amd64.tar.gz
```

Run exporter:

```bash
./redis_exporter
```

---

## Step 4: Install Prometheus

Edit prometheus.yml:

```yaml
scrape_configs:
  - job_name: redis
    static_configs:
      - targets:
        - localhost:9121
```

---

## Step 5: Start Prometheus

```bash
./prometheus
```

---

## Step 6: Install Grafana

```bash
sudo apt install grafana -y
```

Start Grafana:

```bash
sudo systemctl start grafana-server
```

---

## Step 7: Add Prometheus Datasource

Grafana URL:

```text
http://localhost:3000
```

Add:
- Prometheus datasource
- URL: http://localhost:9090

---

## Step 8: Create Dashboard

Add panels one by one using PromQL queries.

---

# 14. Advanced Redis Monitoring

Advanced monitoring includes:

- Redis Cluster Monitoring
- Sentinel Monitoring
- Slow Query Analysis
- Hot Key Detection
- Keyspace Monitoring
- Network Monitoring
- AI-based anomaly detection

---

# 15. Troubleshooting

## Exporter Not Working

```bash
curl localhost:9121/metrics
```

---

## Prometheus Target Failed

Check:

```text
http://localhost:9090/targets
```

---

## Grafana Showing No Data

Verify:
- Datasource connection
- PromQL syntax
- Exporter running status

---

# 16. Best Practices

| Best Practice | Reason |
|---|---|
| Use alert thresholds | Prevent outages |
| Separate dashboards | Better visibility |
| Use labels | Easier filtering |
| Enable retention | Historical analysis |
| Use annotations | Incident tracking |
| Use variables | Dynamic dashboards |

---

# 17. Future Enhancements

Future improvements:

- Kubernetes Redis monitoring
- AI anomaly detection
- Predictive scaling
- Auto-remediation
- Multi-cluster dashboards
- SLO dashboards

---

# 18. Contact Information

| Contact Type | Details                                                             |
| ------------ | ------------------------------------------------------------------- |
| Name         | Suraj Tripathi                                                      |
| Role         | DevOps Trainee                                                      |
| Email        | [suraj.tripathi.snaatak@mygurukulam.co](mailto:suraj.tripathi.snaatak@mygurukulam.co) |

---

# 19. Conclusion

Redis monitoring is extremely important for maintaining:

- High availability
- Fast performance
- Stable applications
- Healthy middleware infrastructure

Using:

- Redis Exporter
- Prometheus
- Grafana
- Alertmanager

---
