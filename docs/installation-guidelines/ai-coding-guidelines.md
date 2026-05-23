# AI-Assisted Development Guidelines (Karpathy)

Behavioral guidelines for AI coding assistants working in this repository, derived from
[Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on
common LLM coding pitfalls. These apply to Claude Code and any AI tool used on OpenAlgo.

---

## 1. Think Before Coding

**State assumptions. Surface confusion. Present tradeoffs.**

- Before writing code, state what you're assuming and why.
- If a request has multiple valid interpretations, list them — don't silently pick one.
- If a simpler approach exists, say so and push back.
- If something is unclear (e.g. which broker, which database, which mode — live vs sandbox), stop and ask.

OpenAlgo-specific: many concepts share similar names (`sandbox` mode vs `sandbox.db` vs
sandbox UI route `/analyzer`). Name what you mean precisely before touching code.

---

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No new abstractions for single-use code.
- No "future-proof flexibility" that wasn't requested.
- No error handling for scenarios that cannot occur given the single-user deployment model.

OpenAlgo-specific: the codebase already has 30+ broker integrations, 6 databases, and a
multi-layer WSGI stack. Every new abstraction has a real maintenance cost. If you write
200 lines and it could be 50, rewrite it.

---

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Do not "improve" adjacent formatting, comments, or style.
- Do not refactor things that aren't broken.
- Match existing style — even if you'd do it differently.
- If you spot unrelated dead code, mention it as a side note. Do not delete it.

When your changes orphan something:
- Remove imports / variables / functions that **your** change made unused.
- Do not remove pre-existing dead code unless explicitly asked.

OpenAlgo-specific: broker integrations follow a strict pattern. Changing a shared utility
(e.g. `utils/httpx_client.py`, `database/auth_db.py`) ripples across all 30+ brokers.
Confirm scope before touching shared code.

---

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform vague tasks into verifiable goals before starting:

| Vague | Verifiable |
|---|---|
| "Fix the WebSocket bug" | "Reproduce the disconnect, confirm it doesn't recur after the fix" |
| "Add a new broker" | "All 8 required modules exist; `uv run app.py` loads the plugin; a test order round-trips" |
| "Refactor the order flow" | "All existing `pytest test/` tests pass before and after" |

For multi-step tasks, state a plan first:

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

OpenAlgo-specific verification commands:

```bash
# Python lint + format check
uv run ruff check .
uv run ruff format --check .

# Run test suite
uv run pytest test/ -v

# Start app and confirm it loads cleanly (watch for import errors)
uv run app.py
```

---

## Quick Reference

| Guideline | One-line rule |
|---|---|
| Think first | Assumptions explicit, confusion named, tradeoffs surfaced |
| Simplicity | Minimum code; no speculative features |
| Surgical | Only change what the request requires |
| Goal-driven | Define "done" before writing line one |
