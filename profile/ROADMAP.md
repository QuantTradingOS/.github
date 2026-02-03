# QuantTradingOS Roadmap

This document describes the planned evolution of QuantTradingOS in phases. Aligned with the [organization profile](README.md) and repo-level status.

---

## Phase 1: Agent Foundations — Completed / Active

**Status:** In place.

**Deliverables**

- Intelligence agents: market regime, sentiment shift, equity insider signals.
- Control agents: execution discipline evaluation, capital guardian (and allocation) for risk gating.
- Review agents: trade journal coach, portfolio analyst.
- Shared conventions and (where present) utilities via `trading-os-framework`.
- **Orchestrator** (orchestrator repo): one pipeline tying agents together—regime → portfolio → [execution-discipline] → allocation → [guardian]—plus FastAPI API, CLI, and scheduler (interval/cron). Single entry point for the full stack.

**Why it matters**

Phase 1 proves out the agent model and boundaries: intelligence produces context, control evaluates or gates, review supports humans. Each repo is usable standalone; the orchestrator provides a coherent run path (one API, one CLI, optional scheduled runs) without requiring a separate "core engine" process.

---

## Phase 2: Core Trading Engine & Orchestration — In Progress

**Status:** Partially in place. Orchestration and core runtime exist; config/composition and live broker are later phases.

**Deliverables**

- **Orchestration** — Done. Orchestrator runs the pipeline, exposes /decision and agent endpoints (execution-discipline, guardian, sentiment, insider, trade-journal, portfolio-report), and supports scheduling (APScheduler).
- **Core runtime (qtos-core)** — EventLoop, Strategy, RiskManager, Portfolio; backtesting (OHLCV → metrics); execution layer (PaperBrokerAdapter, LiveBrokerAdapter sandbox-first). No live broker API wiring yet (Phase 4).
- **Contracts** — Adapters map regime/portfolio outputs to Capital-Allocation inputs; execution-discipline score cached when trades+plan not provided.
- **Still planned:** Configuration and composition (which agents/strategies run together) without hardcoding; optional event bus or LangGraph for conditional flows.

**Why it matters**

The orchestrator and qtos-core give you one place that runs the stack and respects risk/discipline layers. Paper and sandbox execution work; live broker connectivity is Phase 4.

---

## Phase 3: Backtesting & Simulation

**Status:** Partially in place (qtos-core); agent-integrated API in place.

**Definitions:** **Backtesting** = historical simulation: replay OHLCV through the engine and agents, no live orders. **Simulation** (paper/sandbox) = real-time run with simulated fills (Phase 4 / qtos-core PaperBrokerAdapter).

**Deliverables**

- **Done (qtos-core):** Backtesting: load OHLCV (CSV/DataFrame) → EventLoop → strategy → risk → simulated fills → metrics (PnL, Sharpe, CAGR, max drawdown). Reproducible runs; agent hooks (Advisors, Validators, Observers) ready. Paper/sandbox execution (real-time simulation) in qtos-core via PaperBrokerAdapter and LiveBrokerAdapter sandbox.
- **Done (orchestrator):** **POST/GET /backtest** — run qtos-core backtest; data from CSV or data-ingestion-service. Agents can call the API and gate alerts on metrics (e.g. only alert if sharpe_ratio > threshold). See orchestrator BACKTEST-PHASE3.md.
- **Planned:** Integration with VectorBT/Backtrader or extended qtos-core backtester; more strategies exposed via /backtest.

**Why it matters**

Systematic trading requires testing before live capital. qtos-core backtesting validates engine and strategies on historical data; paper/sandbox validates in real time with simulated fills. Phase 3 extends backtesting so agents and the orchestrator can trigger backtests and gate decisions on results.

---

## Phase 4: Execution & Broker Connectivity

**Status:** Planned. Paper and sandbox exist in qtos-core; live broker wiring is placeholder.

**Deliverables**

- **Done (qtos-core):** PaperBrokerAdapter (working); LiveBrokerAdapter sandbox-first with safety gate (QTOS_LIVE_TRADING_ENABLED). No live broker API calls yet.
- **Planned:** Broker connectivity (Alpaca, IBKR, etc.): auth (e.g. OAuth2), order submission, status, fills, cancel. Live execution as explicit opt-in with safeguards (kill switches, limits). Risk and discipline enforced before orders are sent.

**Why it matters**

Execution is the last link: the OS is "full stack" when it can route orders through control to a real broker. Paper and sandbox work today; Phase 4 adds live broker adapters and order lifecycle.

---

## Phase 5: UX, Tooling, and Developer Experience

**Status:** Ongoing and post-Phase 4.

**Deliverables**

- **Documentation**: setup, configuration, and "run your first backtest / paper run" guides.
- **Tooling**: CLI or minimal UI for config, run, and inspect (e.g. logs, simple dashboards).
- **AI interface (optional):** **Done.** MCP server (mcp-server repo) exposes tools (run_backtest, get_prices, get_news, get_insider, run_decision). Chatbot (chatbot/ in workspace): Streamlit + LangGraph ReAct agent; loads MCP tools via stdio; run backtests, fetch data, or run the pipeline in natural language. Full-stack Docker: data-service runs one-off price ingestion on startup (TRACKED_SYMBOLS) so backtests work out of the box.
- **Developer experience**: clear APIs, examples, and contribution guidelines so new repos and agents are easy to add.
- No promise of a polished retail UI; focus is on builders and systematic traders.

**Why it matters**

Usability and clarity determine whether the OS is actually used and extended. Phase 5 makes the system approachable without overselling it as a turnkey product.

---

## Summary

| Phase | Focus | Status |
|-------|--------|--------|
| 1 | Agent foundations + orchestrator (pipeline, API, scheduler) | Completed / active |
| 2 | Core engine (qtos-core) + orchestration | In progress (orchestration done; config/composition optional later) |
| 3 | Backtesting & simulation | Partial (qtos-core backtester); /backtest API for agent-triggered backtests in place |
| 4 | Execution & broker connectivity (paper, then live) | Planned (paper/sandbox done; live broker wiring next) |
| 5 | UX, tooling, developer experience | Ongoing |

What exists today: Phase 1 (agents + orchestrator) and Phase 2 orchestration; qtos-core (backtesting, paper/sandbox execution); Phase 3 agent-triggered backtests via orchestrator POST/GET /backtest; Phase 5 MCP server + chatbot (Streamlit + LangGraph), full-stack Docker with data-service one-off price ingestion on startup. Next: Phase 4 (live broker wiring), Phase 5 (cloud docs, optional VectorBT/Backtrader). For repo-level status, see the [organization profile](README.md) and the repository map there.
