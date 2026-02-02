# QuantTradingOS

**QuantTradingOS** is a modular, agent-based trading operating system: a set of AI agents, control layers, and a core engine (qtos-core) for systematic trading. Repos remain standalone and composable; the **orchestrator** ties them together in one pipeline (regime → portfolio → allocation, optional discipline and guardian) and exposes a single FastAPI API, CLI, and scheduler. So today you get both modular building blocks and a coherent run path where intelligence, risk, and execution work together with clear boundaries.

We are early-stage and transparent about what exists and what does not.

---

## Project Status

**Currently implemented**

- **Orchestration layer (orchestrator)** — One pipeline: regime → portfolio → [execution-discipline] → allocation → [guardian]. FastAPI API with Swagger: POST/GET /decision plus agent endpoints (execution-discipline, guardian, sentiment-alert, insider-report, trade-journal, portfolio-report). Scheduler (APScheduler): run pipeline on an interval or cron, standalone or with the API. CLI: `python -m orchestrator.run`.
- **Core trading engine (qtos-core)** — Event-driven runtime: EventLoop, Strategy, RiskManager, Portfolio, Order, Signal. Deterministic, no broker or AI in the core; agents plug in as advisors, validators, or observers.
- **Backtesting framework (qtos-core)** — Load OHLCV (CSV/DataFrame), run strategies through the engine, simulate fills, compute metrics (PnL, Sharpe, CAGR, max drawdown). Agent hooks (Advisors, Validators, Observers) ready for MarketRegime, Sentiment, CapitalGuardian, etc.
- **Execution layer (qtos-core)** — Broker abstraction (BrokerAdapter): PaperBrokerAdapter and LiveBrokerAdapter. Paper simulates fills in real time; Live is sandbox-first (sandbox=True by default), with QTOS_LIVE_TRADING_ENABLED safety gate for real orders. Same strategy interface as backtesting; swap adapters without changing engine or agents. Safety: daily PnL limit, max position per trade, kill switch. Real broker API calls (Alpaca, IBKR, etc.) are placeholders in LiveBrokerAdapter; interface is ready.
- **Intelligence agents** — Market regime detection, sentiment monitoring, insider-signal analysis. These agents produce signals and context; they do not execute.
- **Risk & discipline** — Execution discipline evaluation and (where present) pre-trade risk governors (e.g. capital guardian). They assess or gate decisions; qtos-core provides the interfaces to enforce them in backtests and execution.
- **Post-trade review** — Trade journal coaching and portfolio analytics. Human-in-the-loop tools for learning and oversight.

**Not yet implemented (intentional)**

- **Live broker API wiring** — LiveBrokerAdapter exists (sandbox-first, safety gate); real broker SDK calls (Alpaca, IBKR, etc.) are placeholders. Paper and live-sandbox execution work in qtos-core; wiring to a specific broker is per-adapter.

If you are looking for turnkey autotrading, it does not exist here yet. We do offer backtesting, paper execution, and live execution in sandbox mode in qtos-core. If you are building or researching agent-based trading and want to reuse or contribute to these layers, this is the place.

---

## Architecture at a Glance

Interaction model (core, backtesting, and paper execution in qtos-core; live brokers planned):

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/c043c75e-428b-4045-be54-694e708d1cd0" />

```
REVIEW (parallel): Trade journal, portfolio analyst — support humans.
```

**Orchestrator** runs the pipeline (regime → portfolio → allocation, optional discipline and guardian) and exposes it plus all agents via a single FastAPI API and CLI; optional scheduler runs the pipeline on a timer.

Agents feed context and signals. The core engine (qtos-core) runs strategies and passes decisions through control in backtests and in execution (paper mode). Execution layer provides BrokerAdapter: PaperBrokerAdapter and LiveBrokerAdapter (sandbox-first; real broker API calls are placeholders). Same advisor/validator/observer hooks in backtest and execution.

---

## Repository Map

| Repository | Category | Status | Description |
|------------|----------|--------|-------------|
| `orchestrator` | Core | Active | One pipeline (regime → portfolio → execution-discipline → allocation → guardian). FastAPI API: /decision + agent endpoints (execution-discipline, guardian, sentiment, insider, trade-journal, portfolio-report). Scheduler (interval/cron). CLI: `python -m orchestrator.run`. |
| `qtos-core` | Core | Active | Event-driven core: EventLoop, Strategy, RiskManager, Portfolio. Backtesting (OHLCV, metrics). Execution: PaperBrokerAdapter and LiveBrokerAdapter (sandbox-first, QTOS_LIVE_TRADING_ENABLED gate); safety (PnL limit, kill switch). Live broker API wiring is placeholder. No AI, UI. |
| `trading-os-framework` | Core | Active | Shared libraries, utilities, and conventions for agents. |
| `market-regime-agent` | Intelligence | Active | Market regime detection and classification. |
| `sentiment-shift-alert-agent` | Intelligence | Active | Financial news and sentiment monitoring. |
| `equity-insider-intelligence-agent` | Intelligence | Active | Insider activity and related signal analysis. |
| `capital-guardian-agent` | Control | Experimental | Pre-execution risk governor: drawdown, regime, exposure. |
| `capital-allocation-agent` | Control | Experimental | Position sizing, risk limits, trade gating. |
| `execution-discipline-agent` | Control | Active | Automated evaluation of trade execution discipline. |
| `trade-journal-coach-agent` | Review | Active | Trade journal analysis and coaching insights. |
| `portfolio-analyst-agent` | Review | Active | Portfolio performance and risk analytics. |

**Category key:** **Intelligence** = signals/context; **Control** = risk & discipline; **Review** = post-trade and human support; **Core** = shared infra, engine (qtos-core), and orchestrator.

---

## Who This Is For

- **Systematic traders** who want to plug regime, sentiment, or discipline logic into their own stack—or run the full pipeline and API via the orchestrator.
- **Researchers** exploring agent-based trading, market regime, or execution quality.
- **Builders** who prefer modular, open components over a single black box.

---

## Who This Is Not For

- **Black-box autotrading** — We do not offer a "set and forget" bot. Human oversight and your own strategy logic are assumed.
- **High-frequency trading (HFT)** — Latency and tick-level execution are out of scope.
- **Turnkey retail bots** — No one-click deployment or guaranteed returns. This is a toolkit and a platform-in-progress.

---

## Getting Started

**Run the full pipeline and API:** Clone a workspace that includes the **orchestrator** repo and its sibling agents (Market-Regime-Agent, Portfolio-Analyst-Agent, Capital-Allocation-Agent, etc.). From the workspace root: `python -m orchestrator.run` (CLI) or `uvicorn orchestrator.api:app` (API). See the [orchestrator](https://github.com/QuantTradingOS/orchestrator) README for layout, scheduler, and agent endpoints.

**Use or contribute to individual repos:** Pick a repository from the table above, clone `https://github.com/QuantTradingOS/<repository-name>.git`, and follow that repo's README.

See **[ROADMAP.md](ROADMAP.md)** for phased plans and what we are building next.

---

## Repo README Header Convention

For consistency across the organization, each repository README should include a short **header block** at the top (below the title) with:

| Field | Purpose |
|-------|---------|
| **Status** | e.g. Active, Experimental, Planned — matches the org repository map. |
| **Layer** | One of: Intelligence, Control, Review, Core. |
| **Integration** | `Standalone` — usable on its own; or `OS-integrated` — used by the orchestrator pipeline and/or qtos-core. |

Example:

```markdown
# Some-Agent

**Status:** Active · **Layer:** Intelligence · **Integration:** Standalone

...
```

This lets visitors see at a glance where a repo fits and how mature it is, without opening the org profile.

---

## License

This organization and its repositories are licensed under the MIT License unless stated otherwise in a specific repo.

---

*To update what appears on [github.com/QuantTradingOS](https://github.com/QuantTradingOS): copy this file to the root `README.md` of the [QuantTradingOS/.github](https://github.com/QuantTradingOS/.github) repository and push there.*
