# Assignment — Build Your Own MCP Reviewer 🛰️

> You have the full **RepoRadar** codebase in front of you: a LangGraph multi-agent system that reviews a GitHub repo through the **GitHub MCP server**. Your job is to **fork the idea** and build your *own* multi-agent reviewer powered by a **different MCP server**.
>
> Same skeleton. New data source. New agents. Your own repo.

---

## 1. The goal

Each of you gets a **unique MCP server** and a **unique "X-Radar" project**. You will reuse RepoRadar's architecture and swap in your MCP server so three specialist agents inspect *your* subject, return three verdicts with evidence, a summary, and a 1–5 quality score — streamed live to the UI.

You learn the thing that actually matters: **an MCP client is just a tool layer. Swap the server, keep the brain.**

---

## 2. What you KEEP (reuse RepoRadar as-is)

You do **not** rebuild the plumbing. Keep all of this:

- **LangGraph multi-graph** — one main graph ([`agent/graph.py`](agent/graph.py)) + one **subgraph per agent** ([`agent/agents/`](agent/agents/)).
- **The agent pattern** — every agent is `execution_guardrail → tool_selection → tool_execution → analyze` ([`agent/agents/_common.py`](agent/agents/_common.py)).
- **MCP client layer** — [`agent/mcp_client.py`](agent/mcp_client.py) via `langchain-mcp-adapters`. **MCP is the only way your agents touch the outside world.** No raw SDKs/REST in agent logic.
- **Three guardrails** — input / execution / output ([`agent/guardrails.py`](agent/guardrails.py)).
- **The reasoning nodes** — orchestrator, summarizer, **answer_evaluation (LLM-as-judge)**, tone ([`agent/nodes.py`](agent/nodes.py)).
- **FastAPI SSE backend** ([`api/main.py`](api/main.py)) + **React live-progress UI** ([`frontend/`](frontend/)).
- **Docker + Railway** deploy config and the [`docs/`](docs/) guides.

---

## 3. What you CHANGE (the 7 steps)

| # | Change | File(s) |
|---|--------|---------|
| 1 | Point the MCP client at **your** server (transport, url/command, headers/env) | [`agent/mcp_client.py`](agent/mcp_client.py) + [`.env.example`](.env.example) |
| 2 | Confirm your tools load: print `await get_github_tools()` (rename it) | [`agent/mcp_client.py`](agent/mcp_client.py) |
| 3 | Rewrite the **system prompts** for your domain + your 3 agents | [`agent/prompts.py`](agent/prompts.py) |
| 4 | Redefine each agent's **verdict model** (the `Literal` scale) | [`agent/agents/*.py`](agent/agents/) |
| 5 | Update the orchestrator's agent list + the **UI verdict→badge mapping** | [`agent/nodes.py`](agent/nodes.py) `ALLOWED_AGENTS`, [`frontend/src/lib/verdicts.js`](frontend/src/lib/verdicts.js) |
| 6 | Re-theme the UI labels/title (optional but nice) | [`frontend/src/App.jsx`](frontend/src/App.jsx), [`api/main.py`](api/main.py) `MAIN_LABELS` |
| 7 | Add an **offline test** with fake LLMs/tools to prove the graph runs without keys | new `tests/` or a script |

> 💡 The single most important lesson: in step 1 you only change the **connection block**. Everything downstream (tool calling, guardrails, evaluation, streaming) keeps working because it never knew it was talking to GitHub.

---

## 4. Evaluation — REQUIRED ✅

Your project is not done until it can **judge its own answers** and you can show it works.

1. **Keep `answer_evaluation`** (the 1–5 LLM-as-judge in [`agent/nodes.py`](agent/nodes.py)). Tune its prompt for your domain.
2. **Write an `EVALUATION.md`** in your repo:
   - Run your reviewer on **3 real targets** (e.g. 3 repos / 3 pages / 3 databases).
   - Record each target's **three verdicts + the score + the judge's justification**.
   - Add a paragraph: *was the verdict fair? where did it get fooled?* (grounding, hallucination, missing tools).
3. **Offline pipeline test (bonus, recommended):** monkeypatch `get_fast_llm` / `get_smart_llm` / your `get_*_tools` with fakes and run `graph.astream_events(...)` so the whole graph executes **with no API keys** — exactly how RepoRadar was verified. Prove all three guardrails fire.

---

## 5. Deliverables

- ✅ Your **own public GitHub repo** (do not push to a classmate's).
- ✅ Working app: 3 agents, 3 guardrails, evaluation, SSE UI.
- ✅ Updated `README.md` (what it reviews, how to run, your MCP server + how to get its credentials).
- ✅ `EVALUATION.md` (section 4).
- ✅ A **3-minute demo** of a live scan + the report card.
- ⭐ Stretch: deploy to Railway or Docker using the existing [`docs/`](docs/).

---

## 6. Grading rubric (100 pts)

| Area | Pts | What we look for |
|------|-----|------------------|
| MCP integration | 25 | Tools load from **your** server; MCP is the only external access; read-only & guarded |
| Multi-agent design | 20 | 3 real subgraphs; sensible verdict scales; orchestrator selects agents |
| Guardrails | 15 | input + execution (first node) + output all fire and show in the stream |
| Evaluation | 20 | Judge works; `EVALUATION.md` with 3 targets + honest reflection |
| UX & streaming | 10 | Live progress reveals step-by-step; clean report card |
| Polish | 10 | README, runs with one command, deploy config |

---

## 7. Your assignments

Overview (full detail in the cards below):

| # | Student | Project | MCP server | Needs a key/token? |
|---|---------|---------|-----------|--------------------|
| 1 | Chan Wei Khjan | **FolderRadar** | Filesystem | No |
| 2 | Gurleen Kaur | **PageRadar** | Fetch (web) | No |
| 3 | Komal Patil | **TopicRadar** | Brave Search | Yes (Brave API) |
| 4 | Anived Mishra | **SchemaRadar** | SQLite | No |
| 5 | Lalit Jain | **TableRadar** | PostgreSQL | A reachable Postgres DB |
| 6 | Gurkamal Singh | **ChannelRadar** | Slack | Yes (Slack bot token) |
| 7 | Joseph | **WikiRadar** | Notion | Yes (Notion token) |
| 8 | Siddhesh Sawant | **DriveRadar** | Google Drive | Yes (Google OAuth) |
| 9 | Karthik Balaje R | **UXRadar** | Playwright (browser) | No |
| 10 | Sai Sankar | **PaperRadar** | arXiv | No |
| 11 | Bala Krishna Yenumula | **MarketRadar** | Tavily Search | Yes (Tavily API) |
| 12 | Beadon Roy | **CommitRadar** | Git (local repo) | No |
| 13 | Sagar Sable | **VideoRadar** | YouTube transcript | No (or YT data key) |
| 14 | Ankith Dasu | **TrendRadar** | Hacker News | No |
| 15 | Tilottama Pawar | **FactRadar** | Wikipedia | No |
| 16 | Mini Yadav | **PlaceRadar** | Google Maps | Yes (Maps API) |
| 17 | Jocelyn Jose | **SprintRadar** | Linear | Yes (Linear API) |

> The three agents below are a **starting point** — make them your own. Each `verdict` is a `Literal[...]` you set in your agent's Pydantic model, and each maps to a pass/warn/fail badge in [`frontend/src/lib/verdicts.js`](frontend/src/lib/verdicts.js).

---

### 1 · Chan Wei Khjan — **FolderRadar**
**MCP:** Filesystem (`@modelcontextprotocol/server-filesystem`, run via `npx`, stdio, no key).
**Reviews:** a local project folder you point it at.
**Agents:** `structure` (`good | needs_work`) · `docs` (`clear | unclear | missing`) · `hygiene` (`clean | cluttered` — stray files, leftover TODOs, accidental secrets).
**Score:** maintainability 1–5. **Stretch:** flag files over N lines.

### 2 · Gurleen Kaur — **PageRadar**
**MCP:** Fetch (`@modelcontextprotocol/server-fetch`, `uvx`/`npx`, stdio, no key).
**Reviews:** any public web page (URL input).
**Agents:** `copy` (`clear | unclear`) · `seo` (`good | needs_work` — title/meta/headings) · `structure` (`semantic | flat`).
**Score:** page quality 1–5. **Stretch:** word-count & reading-level note.

### 3 · Komal Patil — **TopicRadar**
**MCP:** Brave Search (`@modelcontextprotocol/server-brave-search`, needs a free **Brave Search API key**).
**Reviews:** the web landscape for a topic/question.
**Agents:** `coverage` (`broad | narrow`) · `credibility` (`strong | weak`) · `recency` (`fresh | stale`).
**Score:** research completeness 1–5. **Stretch:** list top 3 sources as evidence.

### 4 · Anived Mishra — **SchemaRadar**
**MCP:** SQLite (`@modelcontextprotocol/server-sqlite`, points at a `.db` file, no key).
**Reviews:** a SQLite database.
**Agents:** `schema` (`good | needs_work` — normalization) · `indexing` (`present | partial | missing`) · `naming` (`consistent | inconsistent`).
**Score:** DB health 1–5. **Stretch:** suggest one index.

### 5 · Lalit Jain — **TableRadar**
**MCP:** PostgreSQL (`@modelcontextprotocol/server-postgres`, needs a **connection string** to any Postgres — a free Neon/Supabase DB works).
**Reviews:** a Postgres schema.
**Agents:** `keys` (`solid | weak` — PKs/FKs/constraints) · `design` (`good | needs_work`) · `queryability` (`present | missing` — views/indexes).
**Score:** schema quality 1–5. **Stretch:** spot a missing foreign key.

### 6 · Gurkamal Singh — **ChannelRadar**
**MCP:** Slack (`@modelcontextprotocol/server-slack`, needs a **Slack bot token** for a workspace you admin).
**Reviews:** a Slack channel's recent activity.
**Agents:** `activity` (`active | quiet`) · `responsiveness` (`fast | slow`) · `knowledge` (`rich | thin`).
**Score:** team-comms health 1–5. **Read-only only.** **Stretch:** busiest hour as evidence.

### 7 · Joseph — **WikiRadar**
**MCP:** Notion (`@notionhq/notion-mcp-server`, needs a **Notion integration token** + shared pages).
**Reviews:** a Notion workspace / page tree.
**Agents:** `structure` (`organized | messy`) · `completeness` (`complete | partial | empty`) · `freshness` (`current | stale`).
**Score:** knowledge-base quality 1–5. **Stretch:** count empty pages.

### 8 · Siddhesh Sawant — **DriveRadar**
**MCP:** Google Drive (`@modelcontextprotocol/server-gdrive`, **Google OAuth**).
**Reviews:** a Google Drive folder.
**Agents:** `organization` (`tidy | cluttered`) · `naming` (`consistent | inconsistent`) · `sharing` (`safe | risky` — public/over-shared files).
**Score:** file-org 1–5. **Read-only.** **Stretch:** flag world-readable files.

### 9 · Karthik Balaje R — **UXRadar**
**MCP:** Playwright (`@playwright/mcp`, browser automation, no key).
**Reviews:** a live website by actually browsing it.
**Agents:** `navigation` (`clear | confusing`) · `content` (`substantive | thin`) · `links` (`clean | broken`).
**Score:** UX 1–5. **Stretch:** screenshot the landing page as evidence.

### 10 · Sai Sankar — **PaperRadar**
**MCP:** arXiv (community `arxiv-mcp-server`, `uvx`, no key).
**Reviews:** an academic paper (by arXiv id).
**Agents:** `structure` (`well_formed | weak` — abstract/methods/results) · `claims` (`supported | overstated`) · `references` (`solid | thin`).
**Score:** paper quality 1–5. **Stretch:** count citations.

### 11 · Bala Krishna Yenumula — **MarketRadar**
**MCP:** Tavily Search (`tavily-mcp`, needs a free **Tavily API key**).
**Reviews:** a market/competitor landscape for a product idea.
**Agents:** `market` (`broad | narrow`) · `competitors` (`deep | shallow`) · `trends` (`clear | unclear`).
**Score:** insight 1–5. **Stretch:** name 3 competitors as evidence.

### 12 · Beadon Roy — **CommitRadar**
**MCP:** Git (`@modelcontextprotocol/server-git`, points at a local repo, no key).
**Reviews:** a git repo's **history** (not the files — the commits).
**Agents:** `messages` (`clean | messy`) · `branching` (`healthy | tangled`) · `activity` (`active | stale`).
**Score:** repo-health 1–5. **Stretch:** flag commits over N files changed.

### 13 · Sagar Sable — **VideoRadar**
**MCP:** YouTube transcript (community server, e.g. `@anaisbetts/mcp-youtube` or similar; no key for transcripts).
**Reviews:** a YouTube video via its transcript.
**Agents:** `structure` (`organized | rambling`) · `clarity` (`clear | confusing`) · `hooks` (`strong | weak` — intro/engagement).
**Score:** video quality 1–5. **Stretch:** suggest a better title.

### 14 · Ankith Dasu — **TrendRadar**
**MCP:** Hacker News (community HN MCP server, no key).
**Reviews:** a HN story + its comments (or the front page).
**Agents:** `relevance` (`on_trend | niche`) · `sentiment` (`positive | mixed | negative`) · `discussion` (`substantive | shallow`).
**Score:** signal 1–5. **Stretch:** top comment as evidence.

### 15 · Tilottama Pawar — **FactRadar**
**MCP:** Wikipedia (community `wikipedia-mcp`, no key).
**Reviews:** a Wikipedia article.
**Agents:** `completeness` (`complete | partial | stub`) · `neutrality` (`neutral | biased`) · `citations` (`well_cited | under_cited`).
**Score:** article quality 1–5. **Stretch:** count `[citation needed]`-style gaps.

### 16 · Mini Yadav — **PlaceRadar**
**MCP:** Google Maps (`@modelcontextprotocol/server-google-maps`, needs a **Maps API key**).
**Reviews:** a location/neighborhood for a trip.
**Agents:** `amenities` (`rich | sparse`) · `accessibility` (`good | poor` — transit/walkability) · `value` (`strong | weak` — ratings/price).
**Score:** trip/livability 1–5. **Stretch:** nearest 3 highlights as evidence.

### 17 · Jocelyn Jose — **SprintRadar**
**MCP:** Linear (`@tacticlaunch/mcp-linear` or Linear's official server, needs a **Linear API key**).
**Reviews:** a Linear project/board.
**Agents:** `backlog` (`groomed | messy`) · `clarity` (`clear | vague` — ticket descriptions) · `flow` (`balanced | overloaded` — WIP).
**Score:** project-management 1–5. **Read-only.** **Stretch:** flag tickets with no assignee.

---

## 8. Where to find MCP servers & their docs

- **Official reference servers:** <https://github.com/modelcontextprotocol/servers>
- **Directories:** <https://mcp.so> · <https://glama.ai/mcp/servers> · <https://smithery.ai>
- Each server's README tells you the **transport** (`stdio` via `npx`/`uvx`/`docker`, or `streamable_http`) and what **token/key/env** it needs — that's exactly what goes into your [`agent/mcp_client.py`](agent/mcp_client.py) connection block and [`.env.example`](.env.example).

**Two transport shapes you'll use (both already shown in `mcp_client.py`):**

```python
# Remote HTTP server
"yourserver": {
    "transport": "streamable_http",
    "url": "https://...",
    "headers": {"Authorization": f"Bearer {token}"},
}

# Local server over stdio (npx / uvx / docker)
"yourserver": {
    "transport": "stdio",
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/folder"],
    "env": {},
}
```

---

## 9. Rules of the road

- **MCP only.** Agents reach the outside world *only* through MCP tools. No `requests`/SDK calls in agent logic (we grep for this).
- **Read-only & safe.** Prefer read-only tools and keep the execution guardrail honest. Never commit your `.env`.
- **Keep it teachable.** Small files, clear names, short prompts — like RepoRadar.
- **Your own repo.** `git init`, push to a fresh GitHub repo (see [`docs/01_push_to_github.md`](docs/01_push_to_github.md)), share the link.

---

Have fun — and make the live scan feel *alive*. 🛰️

*Agentic AI Builder Expert Bootcamp — Batch 4.0*
