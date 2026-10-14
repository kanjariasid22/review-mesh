# ReviewMesh — Concepts to Learn Before Coding

Everything you need to understand **conceptually** before writing a single line of code.

---

## Python Backend

### 1. FastAPI
What it is, how it handles requests, what makes it async-native, how auto-docs (Swagger UI) work.

### 2. Pydantic
What data validation means, `BaseModel`, type hints enforcing structure, JSON ↔ Python object conversion.

### 3. Async Programming
`async`/`await`, what `asyncio.gather` does (running things in parallel), why it matters for calling 3 agents at once.

### 4. Environment Variables
`.env` files, why you don't hardcode API keys, how to swap providers by changing config.

### 5. REST API Design
What POST/GET mean, request body vs path params, status codes, JSON request/response cycle.

### 6. Error Handling
`try`/`except`, timeouts, what a 429 (rate limit) is, exponential backoff for retries.

### 7. SQLite Basics *(optional)*
What it is (file-based DB, no server), when you'd use it for simple persistence.

---

## AI / LLM Concepts

### 8. LLM API Basics
What an API call to an LLM looks like (system prompt + user message → response), what tokens are, what temperature does.

### 9. Structured Output / JSON Mode
Forcing an LLM to return valid JSON instead of free-form text, why this matters for agent-to-agent communication.

### 10. System Prompts for Specialization
How the same model becomes a "Security Agent" vs a "Bug Agent" purely through different instructions.

### 11. Tool Calling / Function Calling
How LLMs can be told to output data matching a specific schema (related to Pydantic).

---

## Agent Architecture

### 12. What is an AI Agent
An LLM + instructions + tools + a defined role — not just a raw chat call.

### 13. Multi-Agent Systems
Why multiple specialized agents beat one general agent (separation of concerns, parallel work, different expertise).

### 14. Fan-out / Fan-in Pattern
One task splits into parallel sub-tasks (fan-out), results merge back (fan-in) — this is the core of ReviewMesh.

### 15. Supervisor Pattern
A coordinator decides what happens with agent outputs (the Aggregator's role).

### 16. Agent Communication via Shared State
Agents don't talk directly to each other; they read/write to a shared state object that flows through the graph.

---

## LangGraph Specific

### 17. StateGraph
What a graph is (nodes + edges), why a graph instead of just calling functions in order.

### 18. Nodes
Each agent is a node — a function that reads state, does work, writes back to state.

### 19. Edges
Connections between nodes that define execution order.

### 20. Parallel Edges
START → multiple nodes simultaneously (how fan-out works in LangGraph).

### 21. TypedDict State
The shared state object that every node can read from and write to.

---

## LLM Providers (just enough to get an API key)

### 22. Groq
What it is (fast inference host for open models), how to get a free API key.

### 23. Google AI Studio / Gemini
Free tier, how to get an API key.

### 24. Ollama
Running models locally, no API key needed, good for dev.

### 25. OpenAI-Compatible Endpoints
All 3 providers use the same API format, so one client works for all.

---

## Suggested Learning Order

| Order | What | Why First |
|-------|------|-----------|
| **1** | LLM API basics (#8) + Structured output (#9) | You can't build agents without understanding the raw LLM call |
| **2** | What is an agent (#12) + Multi-agent (#13) + Fan-out/fan-in (#14) | The architecture you're building |
| **3** | LangGraph: StateGraph, nodes, edges (#17–21) | The framework that wires it all together |
| **4** | FastAPI + Pydantic (#1, #2) | The backend that wraps the graph |
| **5** | Async (#3) + Error handling (#6) | Making it robust |
| **6** | Get API keys (#22–24) | Last step before coding |
