---
title: "SRE in the Age of AI: What Changes and What Doesn't"
date: 2026-05-01
draft: true
description: "How AI is transforming Site Reliability Engineering — and why the fundamentals still matter more than ever."
tags: ["SRE", "AI", "Career", "Opinion"]
---

## The Hype vs Reality

Everyone's talking about AI replacing jobs. As an SRE who actively builds AI-powered infrastructure tools, here's my honest take: AI is changing SRE significantly — but not in the ways most people think.

It's not replacing SREs. It's raising the floor on what a good SRE can accomplish.

## What AI Changes

### Incident Response

This is where the impact is most immediate. AI can:
- Analyze Prometheus metrics and surface probable root causes in seconds
- Match incidents to historical postmortems automatically
- Generate first-draft postmortems that capture 80% of the content
- Suggest runbook steps based on alert context

What used to take 20 minutes of manual log-diving can now happen in 2 minutes. This is genuinely transformative at 3 AM.

### Monitoring & Observability

AI makes it practical to monitor things you previously couldn't. Writing PromQL queries used to require expertise — now Claude can generate them from plain English. Interpreting anomalous patterns in metrics is something AI does surprisingly well.

### Toil Elimination

The repetitive, manual work of SRE — certificate rotations, capacity reviews, deployment validations — is exactly what AI excels at automating. This is good. SREs should be spending time on design and reliability work, not toil.

## What AI Doesn't Change

### Understanding Systems

You still need to deeply understand distributed systems to build reliable ones. AI can help you debug faster, but it can't replace the mental model of how your system behaves under failure. That comes from reading, building, and being on-call.

### Judgment Under Pressure

When a P0 incident hits and you have three possible root causes, none of them obvious — that judgment call is still human. AI gives you better information; the decision is still yours.

### Human Communication

Incident management is 50% technical and 50% communication. Keeping stakeholders calm, setting expectations, coordinating a response across teams — this is fundamentally human work.

### System Design

Designing a system to be reliable, observable, and maintainable requires thinking about failure modes, operational complexity, and long-term evolution. AI is a useful thinking partner here, but the design is still yours.

## My Prediction

The SREs who will thrive in the next five years are those who:
1. Deeply understand the fundamentals (distributed systems, reliability patterns, observability)
2. Know how to effectively use AI tools to multiply their output
3. Can build AI-powered tooling for their teams, not just use it

The SREs who will struggle are those waiting for AI to do the hard thinking for them. It won't. It'll make the easy parts easier — which means the hard parts are where all the value will be.

Build the fundamentals. Build AI fluency. The combination is rare and valuable.
