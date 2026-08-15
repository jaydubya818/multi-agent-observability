# Multi-Agent Observability

## Runtime visibility for agentic systems

Multi-Agent Observability is a real-time monitoring system for **Claude Code and multi-agent execution**. It captures agent lifecycle events, tool calls, handoffs, failures, and session activity so engineers can inspect what autonomous workers actually did rather than relying on final summaries.

As agent systems move from one interactive assistant to fleets of specialized workers, observability becomes part of the control plane.

> If you cannot reconstruct what an agent did, which tools it used, where it failed, and how work moved between agents, you cannot reliably operate the system at scale.

![Multi-Agent Observability Dashboard](images/app.png)

## What it captures

The system uses Claude Code hooks to stream runtime events including:

- session start/end
- user prompt submission
- pre/post tool use
- tool failures
- permission requests
- subagent start/stop
- notifications
- compaction lifecycle
- agent stop events

Events are stored and streamed to a live dashboard for inspection across concurrent sessions.

## Architecture

```text
Claude / Agent Sessions
        ↓
   Hook Events
        ↓
   HTTP ingestion
        ↓
     Bun server
        ↓
      SQLite
        ↓
    WebSocket
        ↓
   Vue dashboard
```

![Agent Data Flow](images/AgentDataFlowV2.gif)

## Why this matters for autonomous engineering

A one-off coding assistant can be supervised manually. An autonomous software factory cannot.

At higher levels of agent concurrency, engineers need operational answers to questions such as:

- Which agents are currently running?
- Which tool calls produced a change?
- Where did an execution fail?
- Which subagent received a delegated task?
- How long did each phase take?
- Did the runtime request additional authority?
- Can the event history explain the final result?

This project explores the **runtime telemetry layer** needed underneath larger agent orchestration and software-factory systems.

## Quick start

Requirements:

- Claude Code
- `uv`
- Bun
- optional `just`

Start the server and client:

```bash
just start
# or
./scripts/start-system.sh
```

Open:

```text
http://localhost:5173
```

Then run Claude Code in an instrumented repository and watch hook events stream into the dashboard.

## Project-level instrumentation

Copy the included `.claude` configuration into a project:

```bash
cp -R .claude /path/to/project/
```

Set a unique `source-app` identifier in the hook configuration so events can be attributed to the correct project/runtime.

## System-wide instrumentation

The hooks can also be merged into global Claude Code configuration so sessions across repositories emit to one local observability service.

The repository includes a macOS launchd definition under `deploy/` for keeping the observability service alive across sessions and restarting it after crashes.

## Relationship to Mission Control

[Mission Control](https://github.com/jaydubya818/MissionControl) is focused on the higher-level operating model for autonomous software delivery: intent, WorkOrders, bounded execution, verification, evidence, recovery, policy, and human decisions.

Multi-Agent Observability focuses on a lower-level question:

> What happened inside the agent runtime?

Together the layers look conceptually like:

```text
Mission Control
      ↓
Software Factory / WorkOrder
      ↓
Agent runtime(s)
      ↓
Multi-Agent Observability
      ↓
Events / traces / failures / lifecycle
```

## Engineering themes

- agent lifecycle telemetry
- multi-agent concurrency visibility
- tool-call tracing
- delegation/handoff observability
- durable event history
- real-time streaming
- failure inspection
- runtime attribution
- operator dashboards
- instrumentation through agent hooks

## Current scope

This repository is an observability system, not an autonomous-delivery authority. It does not decide whether work is accepted, merged, or released. Those decisions belong in higher-level governance and verification systems.

That separation is deliberate: **telemetry explains execution; evidence and policy determine whether execution should advance.**
