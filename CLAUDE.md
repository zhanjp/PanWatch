# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

盯盘侠 (PanWatch) is a self-hosted AI stock assistant with real-time monitoring, intelligent technical analysis, and multi-account portfolio management. It supports A-shares, HK stocks, and US stocks.

**Tech Stack:**
- Backend: FastAPI, SQLAlchemy, APScheduler, OpenAI SDK
- Frontend: React 18, TypeScript, Tailwind CSS, shadcn/ui, Vite
- Deployment: Docker

## Development Commands

```bash
# Backend (Python 3.10+)
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python server.py                 # Runs on http://localhost:8000

# Frontend (Node.js 18+ / pnpm)
cd frontend && pnpm install && pnpm dev   # Runs on http://localhost:5173

# Docker build
./build.sh <version>             # e.g., ./build.sh v1.0.0

# Run tests
pytest                           # Tests are in tests/
```

## Architecture

```
src/
├── agents/           # Agent implementations (analysis logic)
│   ├── base.py       # BaseAgent abstract class, AgentContext, AnalysisResult
│   └── *.py          # Individual agents (daily_report, intraday_monitor, etc.)
├── collectors/       # Data collectors (quotes, kline, news, capital_flow)
├── core/             # Core utilities
│   ├── ai_client.py  # OpenAI-compatible API client
│   ├── notifier.py   # Multi-channel notifications (Telegram, WeChat, etc.)
│   └── scheduler.py  # APScheduler-based agent scheduling
├── web/              # FastAPI application
│   ├── api/          # API routes (auth, stocks, agents, settings, etc.)
│   ├── models.py     # SQLAlchemy models
│   └── database.py   # DB setup (SQLite at DATA_DIR/panwatch.db)
├── models/           # Shared data models (MarketCode enum)
└── config.py         # Pydantic settings

prompts/              # AI prompt templates (one per agent)
frontend/src/         # React UI (pages/, components/, hooks/, lib/)
server.py             # Entry point, agent registry, scheduler setup
```

## Agent System

Agents are the core analysis units. Each agent:
1. Collects data via `collect()` method
2. Builds prompts via `build_prompt()` using templates from `prompts/`
3. Calls AI via `analyze()` (BaseAgent default implementation)
4. Optionally notifies users via `should_notify()`

**Execution Modes:**
- `batch`: Analyze all stocks together (e.g., daily report)
- `single`: Analyze each stock individually (e.g., intraday monitor)

**To add a new Agent:**
1. Create `src/agents/my_agent.py` extending `BaseAgent`
2. Create `prompts/my_agent.txt` with system prompt
3. Register in `server.py`: Add to `AGENT_REGISTRY` dict and `seed_agents()` list

**Built-in Agents:**
- `daily_report`: Post-market daily report
- `intraday_monitor`: Real-time monitoring during trading hours
- `news_digest`: Fetch and summarize relevant news
- `premarket_outlook`: Pre-market analysis
- `chart_analyst`: K-line screenshot + multimodal AI analysis

## Data Sources

Data sources provide external data (quotes, kline, news, etc.). Registered in `seed_data_sources()` in `server.py`.

**Types:** `quote`, `kline`, `news`, `capital_flow`, `chart`

**To add a new Data Source:**
1. Create collector in `src/collectors/my_collector.py`
2. Add entry in `seed_data_sources()` with config schema

## Key Patterns

- **Agent Context:** `AgentContext` provides `watchlist`, `portfolio`, `ai_client`, `notifier`
- **Portfolio:** Multi-account support via `PortfolioInfo`, `AccountInfo`, `PositionInfo`
- **AI Model Resolution:** stock_agent override → agent default → system default
- **Notify Channel Resolution:** Same cascade as AI model
- **Prompt Templates:** One `.txt` file per agent in `prompts/`

## Coding Conventions

**Python:**
- PEP 8, 4-space indent, type hints for new code
- Files: `snake_case.py`, classes: `PascalCase`, functions/vars: `snake_case`
- Database models in `src/web/models.py`

**TypeScript:**
- Components: `PascalCase.tsx` in `frontend/src/`
- Hooks: `use-` prefix
- Utilities: `camelCase.ts`

**Commits:**
- Format: `<type>: <subject>` where type ∈ `{feat, fix, docs, refactor, style, test}`
- Example: `feat: add intraday monitor agent`

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `AUTH_USERNAME` | Pre-set login username | Set on first visit |
| `AUTH_PASSWORD` | Pre-set login password | Set on first visit |
| `JWT_SECRET` | JWT signing key | Auto-generated |
| `DATA_DIR` | Data storage directory | `./data` |