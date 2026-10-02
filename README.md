# Agentic Design Patterns — LangGraph + LangChain + Ollama

Small, readable notebooks (one pattern each, heavily commented) that run **100% locally**.

* **Models:** `llama3.1` (main brain, tool calling, structured output) and `qwen3:0.6b` (tiny/fast helper for classify / summarise / route)
* **Libraries (latest at time of writing):** `langgraph 1.2.x`, `langchain 1.4.x`, `langchain-ollama 1.1.x`

## Quick start

```bash
# 1) Models (Ollama must be installed: https://ollama.com)
ollama pull llama3.1
ollama pull qwen3:0.6b

# 2) Python env - with uv (recommended)
uv sync
uv run jupyter lab

#    ...or with plain pip
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -e . 2>/dev/null || pip install "langgraph>=1.2" "langchain>=1.4" "langchain-ollama>=1.1" jupyterlab ipykernel
jupyter lab
```
Open `notebooks/00_start_here.ipynb` first.

## What's inside

| # | Notebook | Pattern | Key API |
|---|---|---|---|
| 00 | `00_start_here` | LangGraph basics (state, node, edge, reducer) | `StateGraph` |
| 01 | `01_planning` | Plan → execute steps → synthesize | structured output, conditional loop |
| 02 | `02_reflection` | Generate → critique → improve | conditional edge + loop cap |
| 03 | `03_map_reduce` | Parallel fan-out / fan-in | `Send`, reducers |
| 04 | `04_multi_agent` | Supervisor + specialist agents | `create_agent`, `Command(goto=…)` |
| 05 | `05_hitl_approval` | Human approves / rejects with feedback | `interrupt`, `Command(resume=…)` |
| 06 | `06_hitl_wait_for_input` | Agent pauses to ask the human | `interrupt` in a node *and* in a tool |
| 07 | `07_hitl_time_travel` | Inspect, replay, fork past checkpoints | `get_state_history`, `update_state` |
| 08 | `08_hitl_review_tool_calls` | Approve / edit / reject tool calls | `HumanInTheLoopMiddleware` |
| 09 | `09_tool_use_react`  | ReAct tool loop (by hand + one-liner) | `bind_tools`, `ToolNode`, `create_agent` |
| 10 | `10_prompt_chaining`  | Pipeline of prompts with a gate | conditional edges |
| 11 | `11_routing`  | Cheap classifier → specialist | small model as router |
| 12 | `12_parallelization`  | Sectioning + voting | parallel edges, threads |
| 13 | `13_memory`  | Short-term (thread) + long-term (store) memory | `InMemorySaver`, `InMemoryStore` |
| 14 | `14_guardrails`  | Input / output checks | regex + tiny LLM judge |
| 15 | `14_subgraphs`  | Graph and subgraphs |

 

### Other patterns worth knowing (not coded here)
* **Agentic RAG** – agent decides *when/what* to retrieve, grades the documents, rewrites the query if they're poor.
* **Swarm / handoffs** – agents transfer control to each other without a supervisor (`langgraph-swarm`).
* **Hierarchical teams** – a supervisor whose workers are themselves supervisors (compiled graphs used as nodes).
* **Self-healing code agent** – generate code → run → feed the error back → fix (reflection with a real tool).
* **Evaluation / LLM-as-judge** – automated scoring of agent runs (LangSmith / `openevals`).

## Human-in-the-loop cheat sheet
1. Compile with a **checkpointer** (`InMemorySaver()` for demos; SQLite/Postgres in production).
2. Always pass a **`thread_id`**: `{"configurable": {"thread_id": "abc"}}`.
3. `interrupt(payload)` pauses → result contains `"__interrupt__"`.
4. Resume with `graph.invoke(Command(resume=value), same_config)`.
5. The interrupted node **re-runs from the top** on resume → keep side-effects *after* `interrupt()`.

## Troubleshooting
* **`ConnectError` / connection refused** – start Ollama (`ollama serve`).
* **`model not found`** – run the two `ollama pull` commands above.
* **Tool calls not happening / malformed structured output** – use `llama3.1` for those; `qwen3:0.6b` is only used for simple text jobs.
* **Answers cut off or forget context** – raise the context window: `ChatOllama(model="llama3.1", num_ctx=8192)`.
* **Slow on CPU** – swap `llm` for `qwen3:0.6b` while experimenting (quality drops, wiring is identical).
* Small local models are non-deterministic in quality; if a demo gives an odd answer, simply re-run the cell.

## Verification note
The graph wiring of every notebook (interrupts/resume, time-travel, middleware, `Send`, memory store) was executed against the versions above using a scripted stand-in for the chat model. Actual answer quality depends on your local Ollama models.
