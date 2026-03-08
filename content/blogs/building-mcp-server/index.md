---
title: "How I Built an MCP Server for Infrastructure Management"
date: 2026-04-15
draft: true
description: "A deep dive into building InfraClaude — an MCP server that lets Claude manage Kubernetes, Docker, and Prometheus through natural language."
tags: ["MCP", "Claude Code", "Kubernetes", "TypeScript", "DevOps"]
image: /images/projects/infraclaude.png
---

## What is MCP and Why Should You Care?

Model Context Protocol (MCP) is Anthropic's open standard for connecting AI models to external tools and data sources. Think of it as a USB standard for AI — instead of every AI integration being custom-built, MCP gives you a universal connector.

For infrastructure engineers, this is significant. It means you can give Claude direct, structured access to your systems and let it operate them through natural language.

## The Vision

I wanted to be able to say:
- "What's the CPU usage on the prod cluster right now?"
- "Scale the payment service to 5 replicas"
- "Show me all pods that have restarted in the last hour"
- "Run a security scan on this Docker image"

And have Claude actually do it — not just tell me the commands to run.

## Designing the Tool Interface

The key question: what operations should Claude be allowed to perform?

I organized tools into three categories:

**Read-only (always allowed):**
- `kubectl_get` — get resource info
- `prometheus_query` — query metrics
- `docker_inspect` — inspect containers

**Write with confirmation:**
- `kubectl_scale` — scale deployments
- `kubectl_restart` — restart pods

**Restricted (human approval required):**
- Any destructive operations
- Production namespace changes

## The Safety Layer

This was the most important design decision in the entire project. An MCP server with infrastructure access is powerful — and dangerous if misused.

The safety layer works at multiple levels:
1. **Namespace restrictions** — Claude can't touch `kube-system` by default
2. **Dry-run mode** — write operations show a preview before executing
3. **Audit logging** — every action is logged with timestamp and context
4. **Claude Code hooks** — pre-execution hooks validate operations before they run

## Building with TypeScript

*(Technical implementation details — coming soon)*

## Demo

*(Demo GIF — coming soon)*

## What's Next

- Integration with PagerDuty for incident-driven operations
- Terraform plan/apply support
- Multi-cluster support
