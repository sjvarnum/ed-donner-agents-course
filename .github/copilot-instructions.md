# Copilot Instructions for ed-donner-agents-course

Purpose: equip AI coding agents with repo-specific context to run/edit examples fast.

## Big picture
- Multi-project course repo; each top folder is a self-contained week/framework:
  - `1_foundations` (OpenAI tool calling + Gradio), `2_openai` (OpenAI Agents SDK), `3_crew` (CrewAI),
    `4_langgraph` (LangGraph Sidekick), `5_autogen` (AutoGen), `6_mcp` (MCP servers + trading UI).
- No single app entrypoint; treat each folder as its own mini-app with its own run command.

## Environment
- Python `>=3.12`, deps via uv.
- From repo root:
  ```bash
  uv sync
  uv run playwright install   # once, for browser tools
  ```
- Root `.env` commonly uses: `OPENAI_API_KEY`, `GOOGLE_API_KEY`, `GEMINI_API_KEY`,
  `PUSHOVER_TOKEN`, `PUSHOVER_USER`, `POLYGON_API_KEY`, `POLYGON_PLAN=paid|realtime`.

## How to run
- Foundations chat bot (tool calling + pdf context):
  ```bash
  uv run python 1_foundations/app.py
  ```
- LangGraph Sidekick (tools + evaluator + Gradio UI):
  ```bash
  uv run python 4_langgraph/app.py
  ```
- CrewAI examples (run inside a crew folder):
  ```bash
  uv tool install crewai
  cd 3_crew/financial_researcher
  crewai run
  ```
- MCP servers (stdio; attach with an MCP client):
  ```bash
  uv run python 6_mcp/push_server.py
  uv run python 6_mcp/accounts_server.py
  uv run python 6_mcp/market_server.py
  ```
- Trading UI (portfolio/transactions/price charts):
  ```bash
  uv run python 6_mcp/trading_floor.py
  ```

## Patterns you’ll reuse
- LangGraph Sidekick (`4_langgraph/sidekick.py`):
  - State: messages + `success_criteria` + evaluator flags; worker uses `ChatOpenAI(...).bind_tools(self.tools)`.
  - Tools come from `4_langgraph/sidekick_tools.py` (Playwright browser toolkit, file mgmt sandbox, Serper search, Wikipedia, Python REPL, Pushover).
  - Evaluator uses `with_structured_output(EvaluatorOutput)` to decide continue vs END.
  - Add tools by returning new `Tool` objects from `other_tools()` or extending existing toolkits.
- Foundations (`1_foundations/app.py`): JSON function tools (`record_user_details`, `record_unknown_question`), PDF + text context, Gradio chat loop.
- MCP (`6_mcp/*.py`): `@mcp.tool()` and `@mcp.resource()` with `FastMCP("name").run(transport='stdio')`; accounts data persists via `database.py`; market via `market.py` (Polygon or random if no key).
- CrewAI (`3_crew/*`): template layout with `src/<crew>/config/{agents,tasks}.yaml`, `crew.py`, `main.py`; run with `crewai run`.

## Integration notes
- Playwright opens Chromium non-headless; ensure install step above.
- Polygon EOD/real-time selection via `POLYGON_PLAN`; falls back to random price and logs a note on errors.
- Pushover is best-effort; missing tokens just mean push calls no-op/fail gracefully.

## Debugging quick hits
- Browser missing? Run `uv run playwright install`.
- CrewAI on Windows may require Microsoft Build Tools and `uv tool upgrade crewai` (see root README).
- Prefer `uv run <cmd>` to avoid venv path issues; re-run `uv sync` if deps drift.

## Where to look first
- LangGraph: `4_langgraph/sidekick.py`, `4_langgraph/sidekick_tools.py`.
- Foundations: `1_foundations/app.py`, `1_foundations/me/*`.
- MCP trading: `6_mcp/accounts.py`, `6_mcp/market.py`, `6_mcp/trading_floor.py`, `6_mcp/*_server.py`.
- CrewAI templates: `3_crew/*` and `3_crew/community_contributions/*`.
