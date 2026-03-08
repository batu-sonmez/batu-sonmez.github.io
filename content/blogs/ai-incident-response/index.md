---
title: "Building an AI-Powered Incident Response System with Claude"
date: 2026-04-01
draft: true
description: "How I built SREBot — an AI-powered incident response platform that uses Claude for root cause analysis, runbook matching, and postmortem generation."
tags: ["SRE", "AI", "Claude", "Incident Response", "Prometheus"]
image: /images/projects/srebot.png
---

## The Problem

Every SRE team has experienced it: 3 AM, PagerDuty goes off, and you're staring at a wall of metrics trying to figure out what went wrong. The on-call engineer is groggy, the blast radius is unclear, and every minute of downtime is costing real money.

The traditional workflow looks like this:
1. Get paged
2. Check dashboards manually
3. Dig through logs
4. Cross-reference runbooks
5. Escalate if stuck
6. Write a postmortem afterward

This works — but it's slow, error-prone under pressure, and completely dependent on the human's familiarity with the system.

## The Idea

What if an AI could do the initial triage? Not replace the on-call engineer, but give them a 60-second head start with:
- Probable root cause
- Relevant runbook
- Affected services
- Suggested first steps

That's the core idea behind SREBot.

## Architecture

*(Coming soon — architecture diagram)*

## Key Design Decisions

### Why Claude over GPT?

Claude's extended context window handles large metric dumps and log files without truncation. The structured output quality is also more reliable for the JSON-formatted incident reports SREBot generates.

### Why Prometheus?

It's the de-facto standard for Kubernetes-native monitoring. Most SRE teams already have it. SREBot queries the Prometheus HTTP API directly — no additional data pipeline needed.

### The Safety Question

The hardest design decision: should SREBot be allowed to take actions, or just provide analysis?

For v1, analysis only. Here's why: an AI making automated changes to production systems during an incident is a risk that needs careful guardrails. The trust has to be built incrementally. SREBot gives recommendations; humans make the call.

## Results

*(To be filled in after project launch)*

## What I Learned

*(To be filled in after project launch)*
