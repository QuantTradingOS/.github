# QuantTradingOS

**QuantTradingOS** is a modular, agent-based trading operating system: a set of AI agents, control layers, and (eventually) a core engine for systematic trading. Today it is a collection of standalone and composable repositories; the vision is a coherent OS where intelligence, risk, and execution work together with clear boundaries.

We are early-stage and transparent about what exists and what does not.

---

## Project Status

**Currently implemented**

- **Core trading engine (qtos-core)** — Event-driven runtime: EventLoop, Strategy, RiskManager, Portfolio, Order, Signal. Deterministic, no broker or AI in the core; agents plug in as advisors, validators, or observers.
- **Backtesting framework (qtos-core)** — Load OHLCV (CSV/DataFrame), run strategies through the engine, simulate fills, compute metrics (PnL, Sharpe, CAGR, max drawdown). Agent hooks (Advisors, Validators, Observers) ready for MarketRegime, Sentiment, CapitalGuardian, etc.
- **Intelligence agents** — Market regime detection, sentiment monitoring, insider-signal analysis. These agents produce signals and context; they do not execute.
- **Risk & discipline** — Execution discipline evaluation and (where present) pre-trade risk governors (e.g. capital guardian). They assess or gate decisions; qtos-core provides the interfaces to enforce them in backtests.
- **Post-trade review** — Trade journal coaching and portfolio analytics. Human-in-the-loop tools for learning and oversight.

**Not yet implemented (intentional)**

- **Live execution & broker connectivity** — No order routing, no broker APIs. Nothing in this org places live trades. The core and backtesting are library-only.

If you are looking for turnkey autotrading or live broker execution, it does not exist here yet. We do offer a deterministic backtesting framework in qtos-core. If you are building or researching agent-based trading and want to reuse or contribute to these layers, this is the place.

---

## Architecture at a Glance

Intended interaction model (target state; only the agent/review layers are real today):

```
┌─────────────────────────────────────────────────────────────────┐
│  INTELLIGENCE (signals & context)                                │
│  Market regime · Sentiment · Insider · (your models)             │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│  CORE ENGINE (qtos-core) — EventLoop, Strategy, RiskManager,     │
│  Portfolio; backtesting with metrics & agent hooks               │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│  CONTROL (risk & discipline) — gating, sizing, execution quality  │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│  EXECUTION (planned) — broker connectivity, live/paper orders    │
└─────────────────────────────────────────────────────────────────┘

REVIEW (parallel): Trade journal, portfolio analyst — support humans.
```

Agents feed context and signals. The core engine (qtos-core) runs strategies and passes decisions through control in backtests; it provides interfaces for risk and discipline. Live execution and broker connectivity are not yet implemented.

---

## Repository Map

| Repository | Category | Status | Description |
|------------|----------|--------|-------------|
| `qtos-core` | Core | Active | Deterministic event-driven trading core: EventLoop, Strategy, RiskManager, Portfolio. Modular backtesting (OHLCV, metrics, agent hooks). No AI, UI, or broker. |
| `trading-os-framework` | Core | Active | Shared libraries, utilities, and conventions for agents. |
| `market-regime-agent` | Intelligence | Active | Market regime detection and classification. |
| `sentiment-shift-alert-agent` | Intelligence | Active | Financial news and sentiment monitoring. |
| `equity-insider-intelligence-agent` | Intelligence | Active | Insider activity and related signal analysis. |
| `capital-guardian-agent` | Control | Experimental | Pre-execution risk governor: drawdown, regime, exposure. |
| `capital-allocation-agent` | Control | Experimental | Position sizing, risk limits, trade gating. |
| `execution-discipline-agent` | Control | Active | Automated evaluation of trade execution discipline. |
| `trade-journal-coach-agent` | Review | Active | Trade journal analysis and coaching insights. |
| `portfolio-analyst-agent` | Review | Active | Portfolio performance and risk analytics. |

**Category key:** **Intelligence** = signals/context; **Control** = risk & discipline; **Review** = post-trade and human support; **Core** = shared infra and (future) engine.

---

## Who This Is For

- **Systematic traders** who want to plug regime, sentiment, or discipline logic into their own stack.
- **Researchers** exploring agent-based trading, market regime, or execution quality.
- **Builders** who prefer modular, open components over a single black box.

---

## Who This Is Not For

- **Black-box autotrading** — We do not offer a "set and forget" bot. Human oversight and your own strategy logic are assumed.
- **High-frequency trading (HFT)** — Latency and tick-level execution are out of scope.
- **Turnkey retail bots** — No one-click deployment or guaranteed returns. This is a toolkit and a platform-in-progress.

---

## Getting Started

1. Pick a repository from the table above.
2. Clone: `git clone https://github.com/QuantTradingOS/<repository-name>.git`
3. Follow that repo's README for setup and usage.

See **[ROADMAP.md](ROADMAP.md)** for phased plans and what we are building next.

---

## Repo README Header Convention

For consistency across the organization, each repository README should include a short **header block** at the top (below the title) with:

| Field | Purpose |
|-------|---------|
| **Status** | e.g. Active, Experimental, Planned — matches the org repository map. |
| **Layer** | One of: Intelligence, Control, Review, Core. |
| **Integration** | `Standalone` — usable on its own; or `OS-integrated` — expects or plugs into the (future) core engine. |

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
