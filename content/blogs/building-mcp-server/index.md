---
title: "How I Built an MCP Server for Infrastructure Management"
date: 2026-03-08
draft: false
description: "A deep dive into building InfraClaude — an MCP server that gives Claude direct access to Kubernetes, Docker, Prometheus, and Terraform through natural language. 41 tools, a 4-tier safety layer, and live demos."
tags: ["MCP", "Claude Code", "Kubernetes", "TypeScript", "DevOps", "Prometheus", "Grafana"]
---

## The Problem: Copy-Paste Infrastructure Debugging

Every DevOps engineer knows the drill. Something breaks at 2 AM. You open your terminal, run `kubectl get pods`, copy the output, paste it into Claude, ask "what's wrong?", get an answer, run another command, copy, paste, repeat.

You're the middleware between Claude and your infrastructure.

**What if Claude could just... look at your infrastructure directly?**

That's what InfraClaude does.

## What is MCP?

Model Context Protocol (MCP) is Anthropic's open standard for connecting AI models to external tools and data sources. Think of it as a USB-C port for AI — a universal connector that lets Claude interact with any system through a structured interface.

When you configure an MCP server with Claude Code, Claude gains new abilities. It can call tools, read resources, and follow guided workflows — all without you having to copy-paste anything.

## What InfraClaude Does

InfraClaude is an MCP server that exposes **41 infrastructure tools** across 8 categories:

| Category | Tools | What It Does |
|----------|-------|-------------|
| **Kubernetes** | 16 | List pods, describe resources, read logs, scale deployments, rollback, cordon nodes |
| **Docker** | 10 | List containers, inspect, read logs, check stats, manage networks |
| **Prometheus** | 5 | Run PromQL queries (instant + range), check alerts, view targets |
| **Terraform** | 5 | View plans, state, outputs, validate configs (read-only) |
| **Security** | 3 | Trivy image scanning, Gitleaks secret detection, K8s security audit |
| **System** | 4 | Disk usage, processes, network connections, system logs |
| **Resources** | 3 | Auto-updating cluster info, service health, infrastructure summary |
| **Prompts** | 3 | Guided troubleshooting, capacity planning, security audit workflows |

Instead of running commands yourself, you just talk to Claude:

> **You**: "Something is crashing in the demo namespace, can you check?"
>
> **Claude**: *calls `k8s_get_pods`* → sees broken-pod with 6 restarts → *calls `k8s_describe_pod`* → finds BackOff events → *calls `k8s_get_pod_logs`* → reads "ERROR: Failed to connect to database at db:5432"
>
> **Claude**: "The pod `broken-pod` is in CrashLoopBackOff because it can't connect to a database at `db:5432`. There's no database service deployed in this namespace. You need to either deploy a database or update the connection string."

All of this happens in seconds, without you running a single command.

## The Safety Layer: Most Important Design Decision

An MCP server with infrastructure access is powerful — and dangerous if misused. I designed a **4-tier risk classification system** called CommandGuard:

```
SAFE       → Always allowed (get pods, list containers, query metrics)
CAUTION    → Allowed but logged with warning (scale deployment, read logs)
DANGEROUS  → Requires explicit awareness (delete pod, rollback, cordon node)
BLOCKED    → Never allowed (anything touching kube-system, unknown tools)
```

Every single tool call goes through this classification before execution. Key safety features:

- **Namespace protection** — `kube-system` and `kube-public` are always blocked
- **Fail-safe default** — Unknown tools are blocked, not allowed
- **Audit logging** — Every operation is written to `~/.infraclaude/audit.log` as structured JSON
- **RBAC verification** — Checks Kubernetes permissions before attempting operations

This means even if Claude decides to do something risky, the safety layer catches it before it reaches your infrastructure.

## Architecture

```
Claude Code ──MCP Protocol──→ InfraClaude Server
                                    │
                        ┌───────────┼───────────┐
                        ▼           ▼           ▼
                   CommandGuard  AuditLogger  RBAC Check
                        │
            ┌───────────┼───────────┬──────────┐
            ▼           ▼           ▼          ▼
      Kubernetes    Docker    Prometheus   Terraform
       Client       Client      API       CLI Wrapper
            │           │           │          │
            ▼           ▼           ▼          ▼
        K8s API    Docker API  Prom HTTP   terraform CLI
```

The server runs as a standard Node.js process communicating over stdio with Claude Code. Each tool category has its own module with typed interfaces, and all responses are formatted as human-readable tables.

## Live Demo: Debugging a Crashing Pod

Here's a real debugging session. I deployed a broken pod that crashes because it can't connect to a database:

**Step 1: List pods**
```
Pods in namespace 'demo':

NAME                          READY  STATUS   RESTARTS  AGE
broken-pod                    0/1    Running  6         9m
demo-api-69877c5c84-6kd95     1/1    Running  0         9m
demo-api-69877c5c84-lz8bx     1/1    Running  0         9m
demo-worker-6fbd7d4864-rshdl  1/1    Running  0         9m
prometheus-6d9f4c94bd-fqbts   1/1    Running  0         9m
```

Immediately visible: `broken-pod` has 0/1 ready and 6 restarts.

**Step 2: Describe the pod**
```
Name:         broken-pod
Namespace:    demo
Status:       Running
Containers:
  broken:
    Image:    busybox:1.36
    Ready:    false
    Restarts: 6
    Limits:   CPU=50m, Mem=32Mi

Events:
Warning  BackOff  1m  Back-off restarting failed container
```

**Step 3: Read the logs**
```
Starting broken app...
ERROR: Failed to connect to database at db:5432
```

Root cause found in three tool calls.

## Prometheus + Grafana Integration

InfraClaude doesn't just work with Kubernetes. It also connects to Prometheus for metrics querying. I set up a monitoring stack with Grafana dashboards showing:

- **Cluster health** — Which targets are up/down
- **Running containers and pods** — Real-time count from kubelet metrics
- **CPU and memory usage** — Time-series graphs with rate calculations
- **Scrape performance** — How fast Prometheus collects metrics
- **HTTP request rates** — API endpoint traffic analysis

The InfraClaude PromQL tools (`prom_instant_query`, `prom_range_query`) let Claude query these same metrics programmatically:

```
Query: rate(process_cpu_seconds_total[1m])

METRIC                              VALUE
kubernetes-nodes (minikube)          0.027
prometheus (localhost:9090)          0.001
```

This means Claude can correlate Kubernetes events with Prometheus metrics to diagnose performance issues — something that normally requires switching between multiple dashboards.

## Claude Code Hooks & Skills

Beyond the MCP server, InfraClaude includes:

**Hooks** — Automated checks that run before/after tool calls:
- Pre-tool-use: Validates operations against security policies
- Post-tool-use: Logs results and triggers alerts on failures

**Skills** — Domain knowledge files that teach Claude infrastructure best practices:
- Kubernetes troubleshooting methodology
- Incident response procedures
- Docker debugging workflows
- Security review checklists

## Tech Stack

- **TypeScript** (strict mode, ESM modules)
- **@modelcontextprotocol/sdk** — MCP server framework
- **@kubernetes/client-node** — Kubernetes API client
- **Dockerode** — Docker Engine API client
- **Prometheus HTTP API** — Metrics querying
- **Zod** — Runtime type validation
- **Vitest** — Testing (28 unit tests, 100% passing)
- **GitHub Actions** — CI/CD (lint, test, build, security scan)

## What's Next

InfraClaude v1.0 covers the core infrastructure tools. The [roadmap](https://github.com/batu-sonmez/infraclaude/blob/main/ROADMAP.md) includes expansion to **122+ tools**:

- **Database tools** — PostgreSQL, MySQL, Redis, MongoDB monitoring
- **Cloud providers** — AWS, GCP, Azure resource management
- **CI/CD** — GitHub Actions, GitLab CI, Jenkins pipeline monitoring
- **APM** — Datadog, New Relic, Jaeger distributed tracing
- **Cost optimization** — Resource right-sizing and capacity planning
- **Alerting** — PagerDuty, OpsGenie on-call management

## Try It

```bash
git clone https://github.com/batu-sonmez/infraclaude
cd infraclaude
npm install && npm run build
```

Add to your Claude Code project's `.mcp.json`:
```json
{
  "mcpServers": {
    "infraclaude": {
      "command": "node",
      "args": ["dist/index.js"]
    }
  }
}
```

Then just ask Claude: *"What pods are running in my cluster?"*

Check out the full source code and documentation on [GitHub](https://github.com/batu-sonmez/infraclaude).
