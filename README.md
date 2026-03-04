# Notion ↔ Langflow Agents Pipeline

A multi-agent pipeline built in [DataStax Langflow](https://langflow.datastax.com) that reads decision briefs from Notion, runs them through a 3-agent analysis chain, and writes structured recommendations back to Notion.

## Architecture

```
Notion Page (Decision Brief)
  │
  ▼
┌──────────────────────────────────────────────────────┐
│                  Langflow Canvas                      │
│                                                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐       │
│  │ Planner  │───▶│ Analyzer │───▶│  Critic  │       │
│  │ (gpt-4o) │    │ (gpt-4o) │    │ (gpt-4o) │       │
│  └──────────┘    └──────────┘    └──────────┘       │
│                                                      │
│  Breaks down     Makes a          Red-teams the      │
│  the decision    recommendation   recommendation     │
│  into tasks      with tradeoffs   for weaknesses     │
│                                                      │
└──────────────────────┬───────────────────────────────┘
                       │
                       ▼
              Notion Page (Analysis appended)
```

## Agent Roles

### 1. Planner
Receives the raw decision brief and produces:
- **Task decomposition** — 5-8 specific evaluation tasks
- **Missing information** — what data would improve the decision
- **Decision criteria** — factors to optimize for

### 2. Analyzer
Receives the brief + Planner's output and produces a **Decision Memo**:
- **Recommendation** — which options to prioritize and why
- **Options considered** — combinations evaluated
- **Tradeoffs** — "we gain X; we risk Y"
- **Risks & mitigations** — what could go wrong + countermeasures
- **Next steps** — who does what

### 3. Critic
Red-teams the Analyzer's recommendation:
- **5 critiques** — what's wrong or weak
- **Missing considerations** — blind spots
- **Risk assessment** — worst case + likelihood
- **Alternative view** — argues against the recommendation
- **Verdict** — APPROVE / REVISE / ESCALATE with reasoning

## Key Design Patterns

- **Sequential chain** — each agent builds on the previous agent's output, adding depth at each stage
- **Role separation** — Planner structures, Analyzer decides, Critic challenges
- **Structured output** — every agent returns labeled sections for consistent parsing
- **Notion as I/O** — reads from and writes back to Notion, keeping analysis alongside the original brief

## Setup

### Prerequisites
- [DataStax Langflow](https://langflow.datastax.com) account (free tier works)
- Notion integration with read + write access to your target page
- OpenAI API key

### 1. Create a Notion Integration

1. Go to [Notion Integrations](https://www.notion.so/profile/integrations)
2. Click **+ New integration**, name it, and copy the **Integration Secret**
3. Open the target Notion page → **...** menu → **Add connections** → select your integration

### 2. Get Your Notion Page ID

From the Notion URL:
```
https://www.notion.so/Your-Page-Title-<PAGE_ID>
```
Copy the 32-character ID after the last `-`.

### 3. Import the Flow

1. In DataStax Langflow, click **+ New Flow** → **Import**
2. Upload `notion_agents_flow.json`
3. Fill in your credentials:
   - Notion component → set `token` to your Integration Secret
   - Notion component → set `page_id` to your target page
   - OpenAI components → set `api_key` to your OpenAI API key

### 4. Run

Click **Run** (▶) in the Langflow canvas. The analysis will be appended to your Notion page.

## Repo Structure

```
├── notion_agents_flow.json   ← Langflow flow export (secrets redacted)
├── README.md
└── .gitignore
```

## Built With

- [DataStax Langflow](https://langflow.datastax.com) — visual agent orchestration
- OpenAI GPT-4o — LLM inference for all 3 agents
- [Notion API](https://developers.notion.com/) — read/write pages
