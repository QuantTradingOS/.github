# QuantTradingOS Roadmap

This document describes the planned evolution of QuantTradingOS in phases. Missing components (core engine, backtesting, live execution) are **intentional** and sequenced here—not oversights.

---

## Phase 1: Agent Foundations — Completed / Active

**Status:** In place. This is what exists today.

**Deliverables**

- Intelligence agents: market regime, sentiment shift, equity insider signals.
- Control agents: execution discipline evaluation, capital guardian (and allocation) for risk gating.
- Review agents: trade journal coach, portfolio analyst.
- Shared conventions and (where present) utilities via `trading-os-framework`.

**Why it matters**

Phase 1 proves out the agent model and boundaries: intelligence produces context, control evaluates or gates, review supports humans. No execution or orchestration yet—each repo is usable standalone. This lets systematic traders and researchers adopt individual agents without waiting for a full OS.

---

## Phase 2: Core Trading Engine — In Progress / Planned

**Status:** Not yet implemented. Next major focus.

**Deliverables**

- A **core runtime** that can load and orchestrate agents (regime, sentiment, discipline, etc.) and strategy logic.
- Clear contracts: how agents publish signals/context, how the engine consumes them and produces decisions.
- Configuration and composition: which agents and strategies run together, without hardcoding.
- No broker connectivity or live orders in this phase—engine runs in "dry" or simulation mode only.

**Why it matters**

Today, agents are isolated. The core engine is what turns "a set of agents" into "a trading operating system": one place that runs the stack, respects risk and discipline layers, and (in later phases) passes orders to execution. Without it, integration is ad hoc.

---

## Phase 3: Backtesting & Simulation

**Status:** Planned after Phase 2.

**Deliverables**

- A **backtesting / simulation** path that replays historical (or synthetic) data through the core engine and agents.
- Reproducible runs: same config and data produce same results.
- Metrics and outputs suitable for strategy and execution-quality analysis (e.g. discipline, regime alignment).
- No live trading; simulation only.

**Why it matters**

Systematic trading requires testing before live capital. Backtesting validates that the engine, agents, and strategies behave as intended on historical data. It also informs how execution and broker connectivity (Phase 4) should be designed.

---

## Phase 4: Execution & Broker Connectivity

**Status:** Planned after Phase 3.

**Deliverables**

- **Broker connectivity** (specific brokers TBD): auth, order submission, status, fills.
- **Paper trading** mode end-to-end: engine + agents + control, with orders sent to a paper environment.
- **Live execution** as an explicit, opt-in capability with safeguards (e.g. kill switches, limits).
- Risk and discipline layers enforced in the execution path before orders are sent.

**Why it matters**

Execution is the last link: the OS can only be "full stack" when it can route orders through control to a broker. Paper first, then optional live, keeps risk manageable and aligns with "human-in-the-loop" and systematic (not HFT) use.

---

## Phase 5: UX, Tooling, and Developer Experience

**Status:** Ongoing and post-Phase 4.

**Deliverables**

- **Documentation**: setup, configuration, and "run your first backtest / paper run" guides.
- **Tooling**: CLI or minimal UI for config, run, and inspect (e.g. logs, simple dashboards).
- **Developer experience**: clear APIs, examples, and contribution guidelines so new repos and agents are easy to add.
- No promise of a polished retail UI; focus is on builders and systematic traders.

**Why it matters**

Usability and clarity determine whether the OS is actually used and extended. Phase 5 makes the system approachable without overselling it as a turnkey product.

---

## Summary

| Phase | Focus | Status |
|-------|--------|--------|
| 1 | Agent foundations (intelligence, control, review) | Completed / active |
| 2 | Core trading engine (orchestration, no live orders) | Planned |
| 3 | Backtesting & simulation | Planned |
| 4 | Execution & broker connectivity (paper, then live) | Planned |
| 5 | UX, tooling, developer experience | Ongoing |

What exists today is Phase 1. What is missing (core engine, backtesting, execution) is intentional and planned in this order. For the latest repo-level status, see the [organization profile](README.md) and the repository map there.
