# Server Monitoring System

A FastAPI-based server monitoring backend that collects system metrics and file hashes from remote agents, stores and analyzes them, and raises alerts when thresholds or security indicators are breached.  
It provides APIs for viewing live metrics, logs, and alerts, and supports lightweight storage via SQLite or any configured database. Alerts can also be sent via SMTP (for example, Gmail).

---

## Overview

This project enables centralized monitoring of servers and agents by collecting metrics (CPU, RAM, disk, etc.) and hash data for security integrity checks.  
The backend provides a REST API for dashboards or external tools and supports alerting when thresholds are exceeded or malicious hashes are detected.

Core workflow:
1. Agents periodically send metrics and file hash data to the backend.
2. The backend writes the data to a database (logs.db or as defined by DATABASE_URL).
3. An in-memory cache holds the most recent metrics for quick access.
4. Alerts are generated for resource and hash anomalies.
5. Optional SMTP email notifications can be sent when alerts are raised.

---

## Features

- Metrics collection: POST /collect-metrics - receive and store system metrics.
- File integrity monitoring: POST /collect-hashes - detect hash anomalies against an IOC table.
- Real-time data APIs:
  - GET /api/metrics - current and historical server metrics.
  - GET /api/alerts - list of triggered alerts.
  - GET /api/logs - recent log entries.
- Alerting system:
  - Resource alerts for CPU and RAM threshold breaches.
  - Hash alerts when matching known Indicators of Compromise.
  - SMTP email notifications for alerts (via Gmail or configured SMTP).
- Caching of recent metrics in memory for efficient API responses.
- Docker Compose configuration for easy deployment and reproducibility.

---

## Running with Docker Compose

### Prerequisites
- Docker
- Docker Compose

### 1. Set environment variables

Create a .env file in the project root or edit the existing one:

```bash
DATABASE_URL=sqlite:///logs.db
CPU_HIGH=90
RAM_RATIO_HIGH=0.9
IOC_TABLE_NAME=ioc_hashes
IOC_SCHEMA=public
IOC_COL_SHA=sha256
DISABLE_IOC_SEED=false
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_password
ALERT_EMAIL_RECIPIENT=recipient@example.com