# Quant Trading OS

**A modular trading operating system for systematic quants: one pipeline from regime and data through backtesting and execution, with clear boundaries between intelligence, risk, and orders.**

---

## The problem this solves

Most quant workflows end up as a mess of notebooks, one-off scripts, and glued-together backtest vs live paths. Regime logic lives in one repo, portfolio in another; execution is an afterthought. Going from backtest to paper to live means rewriting integration points and hoping nothing breaks. QuantTradingOS gives you a single architecture: the same pipeline and engine for research, backtesting, and execution, with agents (regime, sentiment, discipline) as pluggable layers and a single API and CLI to run it.

---

## Safety‑First by design (across the board)

QuantTradingOS treats **safety and governance as first‑class features**, not add‑ons. The core, orchestration, and AI interfaces all enforce guardrails before anything touches execution:

- **Safety‑First MCP tooling** — policy‑aware tools that inject Institutional Policy context (pgvector semantic search) before trading actions.
- **Hard‑Limit circuit breakers** — p90 volatility + exposure limits block unsafe orders with explicit error codes.
- **Strict schemas + corrective feedback** — stop‑loss and limit‑price are mandatory; missing fields trigger corrective feedback loops (adversarial robustness).
- **Enterprise decision logging** — every agentic decision logs Intent Category + Policy Result into ML‑ready streams (ephemeral + immutable) to monitor drift.
- **Sandbox‑first execution** — live broker wiring is gated and explicitly opt‑in.

This makes the system **audit‑friendly, deterministic, and resilient** in both research and execution environments.

---

## Who this is for

- **Solo quants and small teams** who want a coherent stack instead of notebook sprawl—one pipeline, one API, reproducible backtests and paper trading.
- **Researchers** working on regime detection, execution quality, or agent-based trading who need a real engine and data layer to plug into.
- **Builders** who prefer modular, open components and clear contracts over a single black box.

## Who this is not for

- **Passive-income or “set and forget” seekers** — This is not a turnkey bot. You bring strategy logic and oversight.
- **High-frequency or tick-level trading** — Latency and microstructure are out of scope.
- **Beginners expecting one-click deployment and guaranteed outcomes** — This is a professional framework. We are transparent about what exists and what does not.

---

## Why QuantTradingOS instead of Backtrader, Zipline, QuantConnect, or notebooks?

- **vs Backtrader / Zipline:** We focus on a full pipeline (regime → portfolio → allocation → execution) and a single orchestrator + API, not only a backtester. Same engine and contracts for backtest and live; agents (regime, sentiment, discipline) are first-class.
- **vs QuantConnect:** Self-hosted, no vendor lock-in. You own the data pipeline, the engine, and the execution layer. Live broker wiring is placeholder today; the interface is ready for you or the community to wire Alpaca, IBKR, etc.
- **vs ad-hoc notebooks:** One data layer, one backtest API, one pipeline. Run from CLI, API, or (optionally) a chatbot over MCP. Fewer “which notebook was that?” moments.

---

## How to read the architecture diagram

The diagram below shows the system in four layers, top to bottom: **intelligence** (signals and context), **orchestration** (one pipeline), **core engine** (backtest + execution), and **control + execution** (risk and broker). Arrows show data flow. The **REVIEW** box runs in parallel—trade journal and portfolio analyst support humans and don’t sit in the main order path. Use the diagram to see where your work fits: e.g. a new regime classifier plugs into Intelligence; a new broker adapter plugs into Execution.

---

## Architecture

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/3383f2a5-219d-4d16-bf86-ec546b2a7638" />

```
REVIEW (parallel): Trade journal, portfolio analyst — support humans.
```

**Orchestrator** runs the pipeline (regime → portfolio → [execution-discipline] → allocation → [guardian]) and exposes it via FastAPI and CLI; an optional scheduler runs it on a timer. **Data-ingestion-service** stores prices, news, and insider data in PostgreSQL/TimescaleDB and exposes them via API. **qtos-core** is the engine: backtesting, paper/sandbox execution, and (when wired) live execution through a broker adapter. Intelligence agents feed context; control agents gate risk and discipline; execution is sandbox-first with a safety gate for live.

---

## Use cases that map onto this architecture

- **Validate a strategy on history** — Use the data service (or CSV) and the orchestrator’s `/backtest` endpoint; qtos-core runs the backtest and returns metrics. Same engine whether you call it from the API, CLI, or the optional chatbot.
- **Run the full pipeline on a schedule** — Orchestrator pulls regime, portfolio, and (optionally) execution-discipline and guardian; you get one combined decision. Run via CLI or API; scheduler can trigger it on an interval or cron.
- **Add a new regime or sentiment signal** — Implement an agent that fits the Intelligence layer; the orchestrator already wires regime and sentiment into the pipeline. No need to rewrite the rest of the stack.
- **Paper-trade with the same logic as backtest** — qtos-core’s PaperBrokerAdapter uses the same strategy interface as backtesting. Swap to LiveBrokerAdapter (sandbox or, when wired, live) without changing the engine.
- **Query prices, news, or insider data for research** — Data service exposes `/prices/{symbol}`, `/news/{symbol}`, `/insider/{symbol}`. Use them from the orchestrator, from your own code, or via the optional MCP/chatbot tools.
- **Review trades and portfolio** — Trade journal and portfolio analyst (REVIEW in the diagram) run alongside the pipeline; call them via the orchestrator API for human-in-the-loop review.

---

## 10-minute first win

1. **Clone and run the full stack** (requires Docker):  
   From a workspace that contains `orchestrator/` and `data-ingestion-service/`, run:
   ```bash
   docker-compose -f orchestrator/docker-compose.full.yml up --build
   ```
   This starts PostgreSQL, the data service (with a one-off price ingestion so backtests have data), and the orchestrator API at http://localhost:8000 and http://localhost:8001.

2. **Run a backtest via the API**  
   ```bash
   curl -X POST http://localhost:8000/backtest \
     -H "Content-Type: application/json" \
     -d '{"symbol":"SPY","data_source":"data_service","period":"1y"}'
   ```
   You get back metrics (PnL, Sharpe, CAGR, max drawdown) from qtos-core.

3. **Run the pipeline once**  
   ```bash
   curl -X POST http://localhost:8000/decision
   ```
   The orchestrator runs regime → portfolio → allocation (and optionally discipline/guardian) and returns the combined output.

4. **Optional: try the chatbot**  
   Clone the [chatbot](https://github.com/QuantTradingOS/chatbot) repo, set `OPENAI_API_KEY` in `config/.env`, run `streamlit run app.py`. With the stack and [mcp-server](https://github.com/QuantTradingOS/mcp-server) available, you can ask in natural language for a backtest, prices, or to run the pipeline.

---

## What’s implemented and what isn’t

**Implemented**

- **Orchestration** — One pipeline (regime → portfolio → [execution-discipline] → allocation → [guardian]). FastAPI with `/decision` and agent endpoints; CLI; APScheduler for interval/cron.
- **Data pipeline** — PostgreSQL/TimescaleDB; yfinance (prices) and Finnhub (news, insider). FastAPI: `/prices/{symbol}`, `/news/{symbol}`, `/insider/{symbol}`. Full-stack Docker runs a one-off price ingestion on startup so backtests work.
- **Core engine (qtos-core)** — EventLoop, Strategy, RiskManager, Portfolio; backtesting with OHLCV and metrics; PaperBrokerAdapter and LiveBrokerAdapter (sandbox-first, safety gate). No AI inside the core; agents plug in as advisors/validators/observers.
- **Intelligence agents** — Market regime, sentiment, insider signals (context only; they don’t execute).
- **Risk & discipline** — Execution discipline evaluation; capital guardian (experimental). Interfaces for backtest and execution.
- **Post-trade** — Trade journal coach, portfolio analyst.
- **AI interface (optional)** — MCP server (tools: run_backtest, get_prices, get_news, get_insider, run_decision); Streamlit chatbot using LangGraph + MCP.

**Not implemented (intentional)**

- **Live broker API wiring** — LiveBrokerAdapter and safety gate exist; real broker SDK calls (Alpaca, IBKR, etc.) are placeholders. Paper and sandbox work today.

---

## Complete architecture & TODO

| Layer | What it is | Status | Next / TODO |
|-------|------------|--------|-------------|
| **Data** | PostgreSQL/TimescaleDB + data-ingestion-service. FastAPI. | Done | Optional: agent migration; streaming later. |
| **Orchestrator** | One pipeline, FastAPI, CLI, scheduler. | Done | Optional: event bus / conditional flows. |
| **Core & backtest** | qtos-core backtesting; orchestrator POST/GET /backtest. | Done | Optional: VectorBT/Backtrader; more strategies. |
| **Execution** | Paper + LiveBrokerAdapter (sandbox); live broker placeholder. | Sandbox done | Wire Alpaca/IBKR; order lifecycle. |
| **Deploy** | Docker Compose: postgres + data-service + orchestrator. | Done | Optional: cloud docs. |
| **AI interface** | MCP server + chatbot. | Done | Optional: add to Docker; more tools. |

Details: [ROADMAP.md](ROADMAP.md) and (in workspace) [TODO.md](../../TODO.md).

---

## Repository Map

| Repository | Category | Description |
|------------|----------|-------------|
| `orchestrator` | Core | Pipeline, FastAPI, CLI, scheduler. `/decision` + agent endpoints. |
| `qtos-core` | Core | Event-driven engine: backtesting, Paper/Live adapters, safety gate. |
| `data-ingestion-service` | Core | Prices, news, insider in TimescaleDB/PostgreSQL; FastAPI. |
| `mcp-server` | Core | MCP tools for backtest, prices, news, insider, run_decision. |
| `chatbot` | Core | Streamlit + LangGraph; uses MCP tools. |
| `trading-os-framework` | Core | Shared libs and conventions. |
| `market-regime-agent` | Intelligence | Regime detection. |
| `sentiment-shift-alert-agent` | Intelligence | News and sentiment. |
| `equity-insider-intelligence-agent` | Intelligence | Insider signals. |
| `capital-guardian-agent` | Control | Pre-trade risk (experimental). |
| `capital-allocation-agent` | Control | Position sizing, gating (experimental). |
| `execution-discipline-agent` | Control | Execution quality evaluation. |
| `trade-journal-coach-agent` | Review | Trade journal and coaching. |
| `portfolio-analyst-agent` | Review | Portfolio analytics. |

**Categories:** Intelligence = signals/context; Control = risk & discipline; Review = post-trade; Core = infra, engine, orchestrator.

---

## Data and external APIs

**Data:** Agents use the **data-ingestion-service** (set `DATA_SERVICE_URL`) or direct sources (yfinance, Finnhub, CSV). The service ingests prices (yfinance), news and insider (Finnhub), and exposes `/prices/{symbol}`, `/news/{symbol}`, `/insider/{symbol}`. See [data-ingestion-service](https://github.com/QuantTradingOS/data-ingestion-service).

**Inference:** Some agents use OpenAI (sentiment, insider reports, trade journal). Provide `OPENAI_API_KEY` or pass keys in the request; we don’t store them. Finnhub is used for news/insider and data-service ingestion; get keys at [finnhub.io](https://finnhub.io) and [OpenAI](https://platform.openai.com/api-keys).

---

## Getting Started

- **Backtest only:** [qtos-core](https://github.com/QuantTradingOS/qtos-core) — clone, `pip install -e .`, run `PYTHONPATH=. python examples/buy_and_hold_backtest.py`.
- **Full pipeline:** [orchestrator](https://github.com/QuantTradingOS/orchestrator) — clone a workspace with orchestrator + agents, `pip install -r orchestrator/requirements.txt`, then `python -m orchestrator.run` or `uvicorn orchestrator.api:app`.
- **Full stack (Docker):** `docker-compose -f orchestrator/docker-compose.full.yml up --build` from a workspace with `orchestrator/` and `data-ingestion-service/`. Orchestrator at http://localhost:8000, data service at http://localhost:8001.
- **Chatbot:** [chatbot](https://github.com/QuantTradingOS/chatbot) — requires stack + [mcp-server](https://github.com/QuantTradingOS/mcp-server); set `OPENAI_API_KEY`, then `streamlit run app.py`.

Individual repos: clone `https://github.com/QuantTradingOS/<repo-name>` and follow each repo’s README.

---

## Repo README header convention

Each repo README should include a short header:

| Field | Purpose |
|-------|---------|
| **Status** | Active, Experimental, or Planned. |
| **Layer** | Intelligence, Control, Review, or Core. |
| **Integration** | Standalone or OS-integrated. |

Example: `**Status:** Active · **Layer:** Intelligence · **Integration:** Standalone`

---

## License

MIT unless stated otherwise in a specific repo.

---

*To update the org profile on [github.com/QuantTradingOS](https://github.com/QuantTradingOS): copy this file to the root `README.md` of the [QuantTradingOS/.github](https://github.com/QuantTradingOS/.github) repository and push.*
