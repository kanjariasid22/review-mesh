# ReviewMesh — Multi-Agent Code Review System

A multi-agent code review system where specialized AI agents independently analyze code for security vulnerabilities, bugs, and quality issues — then a supervisor agent aggregates their findings into a single, deduplicated review report.

Built with FastAPI + LangGraph + Groq/Gemini.

## What It Does

You submit a code snippet (or a Git diff) to a FastAPI endpoint. A LangGraph state graph orchestrates 4 agents that work together:

1. **SecurityAgent** — scans for vulnerabilities (SQL injection, hardcoded secrets, unsafe deserialization, etc.)
2. **BugHunterAgent** — identifies logical errors, off-by-one bugs, null reference risks, unhandled exceptions
3. **QualityAgent** — flags maintainability issues (dead code, deep nesting, missing type hints, poor naming)
4. **AggregatorAgent** — receives all findings, deduplicates overlapping issues, resolves conflicts, ranks by severity, and produces the final structured review

The first 3 agents run in parallel (fan-out). The Aggregator runs after all of them finish (fan-in). This is a classic Supervisor pattern in LangGraph.

## Architecture

```
POST /reviews  (code snippet)
       │
       ▼
┌─────────────┐
│  FastAPI     │  ← validates input, triggers the graph
└─────┬───────┘
      │
      ▼
┌─────────────┐
│  LangGraph   │  ← orchestrates the review pipeline
│  StateGraph  │
└─────┬───────┘
      │
      ├──────────────┬──────────────┐
      ▼              ▼              ▼
┌───────────┐ ┌───────────┐ ┌───────────┐
│ Security  │ │ BugHunter │ │ Quality   │   ← fan-out (parallel)
│ Agent     │ │ Agent     │ │ Agent     │
└─────┬─────┘ └─────┬─────┘ └─────┬─────┘
      │              │              │
      └──────────────┼──────────────┘
                     ▼
              ┌─────────────┐
              │ Aggregator  │   ← fan-in (merge + dedupe)
              │ Agent       │
              └─────┬───────┘
                    │
                    ▼
              Structured JSON Review
              (severity, file/line, explanation, suggested fix)
```

## LangGraph State

The shared state that flows through the graph:

```python
class ReviewState(TypedDict):
    code: str                          # input code snippet
    language: str                      # detected or specified language
    security_findings: list[Finding]   # from SecurityAgent
    bug_findings: list[Finding]        # from BugHunterAgent
    quality_findings: list[Finding]    # from QualityAgent
    final_review: list[Finding]        # from AggregatorAgent (deduplicated)
    metadata: dict                     # run_id, timestamps, agent stats

class Finding(BaseModel):
    agent: str                # which agent found it
    severity: str             # critical | high | medium | low | info
    category: str             # security | bug | quality
    title: str                # short summary
    description: str          # detailed explanation
    line_range: str | None    # e.g. "12-15" or null
    suggestion: str           # how to fix it
    confidence: float         # 0.0 to 1.0
```

## Graph Structure (LangGraph)

```python
graph = StateGraph(ReviewState)

graph.add_node("security_agent", security_node)
graph.add_node("bug_agent", bug_node)
graph.add_node("quality_agent", quality_node)
graph.add_node("aggregator", aggregator_node)

# Fan-out: START → all 3 specialists in parallel
graph.add_edge(START, "security_agent")
graph.add_edge(START, "bug_agent")
graph.add_edge(START, "quality_agent")

# Fan-in: all 3 → aggregator
graph.add_edge("security_agent", "aggregator")
graph.add_edge("bug_agent", "aggregator")
graph.add_edge("quality_agent", "aggregator")

graph.add_edge("aggregator", END)
```

## API Endpoints

### `POST /reviews`

**Body:**

```json
{ "code": "...", "language": "python" }
```

**Returns:**

```json
{ "run_id": "...", "findings": [...], "summary": {...} }
```

### `GET /reviews/{run_id}`

Returns: saved review from SQLite (if persistence is added)

### `GET /health`

**Returns:**

```json
{ "status": "ok" }
```

## Tech Stack

| Component | Choice | Why |
|-----------|--------|-----|
| Backend | FastAPI | Async-native, auto Swagger docs |
| Agent Framework | LangGraph | State graph, parallel nodes, typed state |
| Validation | Pydantic | Structured agent output, request/response models |
| LLM (Primary) | Groq (free tier) | Fastest inference, ideal for parallel agents |
| LLM (Backup) | Google AI Studio | Most generous free quota |
| LLM (Dev/Test) | Ollama (local) | Zero cost while building |
| Persistence | SQLite (optional) | No setup, no containers |
| Testing | pytest | API testing with TestClient |

## What You'll Learn

| Python Backend | AI Agents / LangGraph |
|----------------|----------------------|
| FastAPI app structure | LangGraph StateGraph |
| Pydantic models & validation | Fan-out / fan-in pattern |
| Async programming (asyncio) | Agent specialization via prompts |
| REST API design | Structured LLM output (JSON mode) |
| Error handling & retries | Agent-to-agent communication via state |
| Environment-based config | LLM provider abstraction |
| pytest + TestClient | Testing with mocks |

## Scope Boundaries (what's IN vs OUT)

| ✅ IN (10-hour scope) | ❌ OUT (future additions) |
|----------------------|---------------------------|
| Paste code via API | GitHub webhook integration |
| 3 specialist + 1 aggregator | Frontend / dashboard |
| Parallel execution | User accounts / JWT auth |
| Structured JSON findings | Docker / deployment |
| LLM response caching (disk) | Redis / PostgreSQL |
| Swagger UI as the interface | Automatic code fixes |
| | CI/CD pipeline integration |
| | FakeLLM for deterministic testing |

## 5-Day Plan (2 hours/day)

### Day 1 — Foundation

- Python venv, FastAPI app, `/health` endpoint
- Pydantic models (`ReviewRequest`, `Finding`, `ReviewResponse`)
- `LLMService` class (env-based provider, disk cache)
- Test: one raw LLM call returns structured JSON

### Day 2 — Agents

- Write system prompts for Security, BugHunter, Quality agents
- Each agent: takes code → returns `list[Finding]` as JSON
- Test each agent individually with a sample snippet

### Day 3 — The Graph

- Build the LangGraph StateGraph (`ReviewState`)
- Wire fan-out (3 agents parallel) → fan-in (aggregator)
- Aggregator prompt: deduplicate, resolve conflicts, rank severity
- Connect `POST /reviews` → `graph.invoke()`

### Day 4 — Robustness

- Handle: LLM timeouts, malformed output, partial agent failures
- Add retry with backoff on 429s
- Optional: SQLite persistence for completed reviews
- Optional: `GET /reviews/{run_id}` endpoint

### Day 5 — Polish & Demo

- pytest suite with TestClient
- Test 3–4 code samples with intentional bugs
- One live demo run on Groq
- Write README with architecture diagram
- Optional: add FakeLLM mode for deterministic testing
