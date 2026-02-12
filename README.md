### Orion Local LLM Loop v1

A bounded local LLM loop engine for Windows (Python): one scheduler + shared runtime contract with hard cutoffs, tool routing, and stable state/journaling. Supports two agent profiles (Orion + Elysia) via a router, with clean prompt separation and namespaced memory-built to add new tools/actions over time without breaking safety.

---

### Goals

* Build a **stable, testable loop** (no runaway recursion)
* Enforce **hard time cutoffs** (e.g., 3:45pm stop + 5:30pm lock)
* Maintain **durable state** and an **append-only journal**
* Provide **tool routing** with strict allowlists and guardrails
* Support **multiple agent profiles** (Orion + Elysia) under one runtime
* Make it easy to extend with **new tools/actions** over time

---

### Non-Goals (v1)

* Fully autonomous operation without user control
* Unbounded background execution
* Network-wide “agent swarm” or self-replication nonsense
* Storing secrets in repo (tokens, keys, personal journal content)

---

### Architecture (Option A)

* **One Loop Engine**
  * Executes ticks on a schedule (or manual run)
  * Applies runtime policy (cutoffs, iteration limits, safety checks)
  * Loads/saves state
  * Writes journal entries
* **Two Agent Profiles**
  * `orion` profile: structured, disciplined, long-arc planning
  * `elysia` profile: sharp, motivational, creative pressure + critique
* **Router**
  * Selects which profile handles a task
  * Can route by explicit tag (`@orion`, `@elysia`) or heuristics
* **Tools Layer**
  * Tools are explicit functions/actions
  * Each tool has a schema, permissions, and logging
  * Router (or policy) decides which tools are available per profile

---

### Repo Layout

* `src/`
  * `loop.py` (main loop)
  * `router.py` (agent selection)
  * `runtime_policy.py` (cutoffs, iteration caps, lock rules)
  * `tools/`
    * `registry.py` (tool definitions + allowlists)
    * `*.py` (individual tools)
  * `storage/`
    * `state_store.py` (load/save state)
    * `journal_store.py` (append-only journal writer)
* `profiles/`
  * `orion.yaml` (model, params, system prompt path, allowed tools)
  * `elysia.yaml`
  * `prompts/`
    * `orion.system.md`
    * `elysia.system.md`
* `config/`
  * `config.example.yaml`
  * `state.example.json`
* `docs/`
  * `SCHEDULER.md` (Windows Task Scheduler setup)
  * `TOOLS.md` (how to add tools safely)
  * `POLICY.md` (runtime safety contract)
* `.gitignore`
* `README.md`
* `LICENSE`

---

### Requirements

* Windows 11
* Python 3.11+ (3.12 is fine)
* A local model runtime (Ollama, llama.cpp server, OpenAI-compatible local server, etc.)
* Optional: Windows Task Scheduler for timed execution

---

### Quick Start

### 1) Clone + create venv

* `git clone <repo-url>`
* `cd <repo-folder>`
* `python -m venv .venv`
* `.\.venv\Scripts\activate`

### 2) Install deps

* `pip install -r requirements.txt`

### 3) Create local config

* Copy `config/config.example.yaml` → `config/config.yaml`
* Copy `config/state.example.json` → `state/state.json` (or your chosen state path)

### 4) Run manually (dev mode)

* `python -m src.loop --once`
* `python -m src.loop --ticks 10`

### 5) Run scheduled (production-ish)

* See `docs/SCHEDULER.md` for Task Scheduler triggers

---

### Configuration

### config.yaml (concept)

* `model.endpoint`: base URL for your local model server
* `model.api_key`: optional (prefer env var)
* `runtime.max_ticks`: hard upper bound per run
* `runtime.cutoffs`:
  * `hard_stop_time`: e.g., `15:45`
  * `lock_time`: e.g., `17:30`
* `paths`:
  * `state_file`
  * `journal_file`
* `profiles`:
  * `default`: `orion`
  * `available`: `orion`, `elysia`

---

### State + Journal

* **State (`state.json`)**
  * Shared runtime facts + namespaced per-agent sub-state
  * Example structure:
    * `agents.orion.*`
    * `agents.elysia.*`
    * `runtime.*`
* **Journal (`journal.md`)**
  * Append-only
  * Each loop tick writes:
    * timestamp
    * agent used
    * decision summary
    * tools invoked (if any)
    * stop reason (if any)

---

### Router Behavior

Supported routing strategies (choose one):

* **Explicit**
  * If input contains `@orion`, route to Orion
  * If input contains `@elysia`, route to Elysia
* **Heuristic (simple v1)**
  * “planning / scheduling / discipline / architecture” → Orion
  * “copywriting / critique / motivation / creative synthesis” → Elysia
* **Hybrid**
  * Explicit tag wins
  * Otherwise heuristic

---

### Runtime Policy (Boundaries)

This loop enforces a bounded execution contract:

* **Hard stop time**
  * After this time, the loop must stop initiating any new work
* **Lock time**
  * After this time, the loop cannot run except for explicitly allowed housekeeping tasks (or it exits immediately)
* **Max ticks**
  * Every run has an upper bound (prevents runaway recursion)
* **Tool allowlist**
  * Tools must be explicitly registered + permitted for the chosen profile
* **No silent escalation**
  * The loop cannot add tools, change cutoffs, or alter permissions without a human edit to config

(Details live in `docs/POLICY.md`.)

---

### Tools System

Tools are treated as **capabilities** with strict boundaries:

* Each tool:
  * has a name + description
  * defines inputs/outputs (schema)
  * logs its invocation and result
* Tools can be:
  * allowed globally
  * allowed per profile
  * disabled during lock hours
* Future direction:
  * add “Actions” like:
    * file edits (restricted)
    * task list updates
    * knowledge base writes
    * network calls (only through a safe wrapper)

See `docs/TOOLS.md`.

---

### Security Notes

* Do **not** commit:
  * `state.json`
  * `journal.md`
  * `.env`
  * logs
  * any tokens or keys
* Prefer:
  * `.env.example`
  * `state.example.json`
  * `journal.example.md`

---

### Roadmap

* v1
  * stable loop + bounded policy
  * router + profiles
  * state + journal
  * basic tool registry
* v1.1
  * better observability (structured logs)
  * crash recovery + last-known-good state
* v1.2
  * tool permission matrix + per-tool rate limits
  * test suite for policy enforcement
* v2
  * pluggable tool adapters
  * UI hooks (optional)
  * richer memory primitives (still bounded)

---

### Contributing

* This repo is currently optimized for a single-owner workflow.
* If contributions are enabled later, we’ll add:
  * issue templates
  * PR guidelines
  * security policy

---

### License

Do not use/copy/or distribute
