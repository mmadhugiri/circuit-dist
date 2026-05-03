# How Circuit works

Circuit is built around one idea: instead of you summoning AI on demand, the AI works on your thinking continuously — and gets better at it the longer you use it.

Here's how that actually happens under the hood.

---

## The heartbeat

Everything in Circuit is organized around a **beat** — a periodic pulse that does all the work. When you run `circuit daemon` or `circuit serve`, a beat fires immediately and then repeats on a configurable interval (default: 10 minutes).

Each beat has five phases, in order:

1. **Trigger any scheduled notes** — if you've set up recurring prompts, they plant notes before the scan
2. **Scan for new notes** — reads `context/`, compares against the trace log, finds anything unprocessed
3. **Seed new topics** — for each new note classified as a topic, generates a full response
4. **Attend to prior topics** — revisits older notes that haven't been touched recently, looks for what's changed
5. **Run async agents** — fires connectors (market data, filings, news, etc.) within a per-beat budget cap

When you run `circuit -n "..."`, you skip the beat entirely and get an immediate one-shot response. Same pipeline, no waiting for the next tick.

---

## From note to response

When you write a note, Circuit classifies it first:

- **Topic** → generates a full response
- **Principle** → absorbed silently into your context (shapes all future responses, generates nothing)
- **List / Crystal / Reference / Schedule** → saved to the appropriate directory, no response

For topics, the generation pipeline is:

1. **Build context** — pulls your principles (everything in `context/`), recent prior intent summaries (from the trace log), and any relevant references from connectors
2. **Generate** — one LLM call with all of that context loaded in
3. **Judge** — a second LLM call grades the response (0–100) across six dimensions: specific, grounded, non-obvious, followed the spec, not generic, and a critique
4. **Voice** — a third LLM call checks whether the response sounds like you, based on your principles
5. **Output** — rendered to `outputs/`, emailed if configured, written to the trace log

Judge and voice run in parallel. Neither blocks the output — they annotate it.

---

## Three independent LLMs

Generator, judge, and voice are independently configurable. You can generate with Claude Sonnet, grade with Claude Haiku, and check voice with the same Sonnet — or use entirely different providers for each.

```
CIRCUIT_PROVIDER=claude-cli          # generator
CIRCUIT_JUDGE_PROVIDER=claude-cli    # grader
CIRCUIT_VOICE_PROVIDER=claude-cli    # voice reviewer
```

The default is `claude-cli` for all three — uses your Claude subscription, no API key needed.

---

## Memory

Circuit has three memory layers, each operating at a different timescale:

**Principles (immediate)**
Every file in `context/` is loaded as a principle on every call. If you write "I think in systems, not steps" or "I only want to hear about trades I'd hold for 6+ months" — that shapes the very next response. No delay, no summarization.

**Prior context (rolling)**
The trace log tracks every intent you've ever planted and every response generated. Before each new seed, Circuit pulls recent prior intent summaries and loads them in. This is why saying "next verse" works — the Gita conversation is in prior context.

**Consolidation (long-term)**
A background agent sweeps your notes, outputs, and references daily, weekly, and monthly — distilling patterns the seed loads into prior context. The consolidations get sharper over time. Routine days (no notable thinking) cost nothing — they get a silent marker, no LLM call.

---

## Connectors

Connectors are async agents that run in Phase 5 of the beat. They produce **references** — structured data in `references/` that becomes available to the seed on future beats.

Two kinds:

- **Annotators** (sync) — run alongside the seed call, within the same beat. Judge and voice are annotators.
- **Content producers** (async) — run on their own cadence, append data over time. Market data, SEC filings, earnings calendars, macro indicators are all content producers.

A beat-level budget cap (default: 10 tasks/beat) bounds how many agents can fire per tick. Each agent implements `Plan()` (cheap, no LLM) and `Run()` (may call LLM or external API). The registry is process-scoped, so throttle state survives across beats.

---

## Routes

Routes let the seed fetch data mid-generation. Instead of pre-loading everything, the seed can invoke a route during its response — asking for current positions, a price snapshot, or a filing — and resume with the result.

Two tiers:

- **Tier 2 (auto)** — read-only data fetches. Execute immediately; seed sees the result.
- **Tier 1 (propose)** — mutations (placing a trade, sending a message). Queued as proposals; require your approval in the UI before executing.

This is how the seed can say "your NVDA position is up 8.3% today" rather than hedging with "I don't have access to current prices."

---

## The trace log

Everything writes to `logs/runs.jsonl` — an append-only JSONL file. Every seed, every beat, every agent run, every classification, every route call.

This log is the foundation for:
- Knowing which notes have already been processed (no double-seeding)
- Building prior context (what have you been thinking about lately?)
- Attention sweep eligibility (which topics are due for a revisit?)
- Consolidation source material (what happened last Tuesday?)
- Judge scores over time (is the output quality improving?)

Nothing is ever rewritten. Corrections create new records. This is what makes the memory trustworthy — the raw history is always recoverable.

---

## The shape of a response

A generated response has up to five sections:

| Section | What it is |
|---------|-----------|
| **Context** | Current state — specific, with names and numbers |
| **Questions** | Things you haven't answered that probably matter |
| **Related** | Threads that connect to this topic |
| **Worth knowing** | Concerns worth your attention |
| **Going deeper** | Sub-questions that earn their own response on the next beat |

Short notes get short responses. A casual observation might return a single paragraph with no heading. A complex question gets the full structure. The shape scales to the signal.

---

## Directory layout

```
context/        ← your notes and principles (the input)
outputs/        ← generated responses
references/     ← async connector data (market data, filings, news)
crystals/       ← distilled permanent knowledge
lists/          ← structured lists (watchlists, contacts, etc.)
schedules/      ← recurring note triggers
consolidations/ ← daily/weekly/monthly memory rollups
logs/           ← trace log (runs.jsonl)
pending/        ← queued Tier-1 proposals awaiting approval
```

Circuit doesn't need a database. The filesystem is the state.
