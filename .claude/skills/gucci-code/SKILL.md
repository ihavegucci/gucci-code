---
name: gucci-code
description: Use when the user dictates an app, site, bot, feature or fix to build end-to-end and expects a finished result without reviewing specs, tickets or code — vibecoding, non-technical users, "собери под ключ", "build it for me", "не задавай лишних вопросов". Also on /gucci-code, «гуччи», «собери гуччи», or a named mode — «полный автомат», «погриль меня», «строго по брифу», «проработай глубоко».
argument-hint: "[auto|semi|interview] [strict|deep] что построить"
---

# Gucci Code

Dictated idea → working project, in one dialogue, no stage-by-stage approvals.

**The brief is the contract.** The user's words become a numbered manifest before anything else exists, and nothing leaves that list except by their say-so.

**Cheap by construction.** Everything below that reads like a shortcut is one, taken deliberately. What is *not* cut is *The invariants* and *The gates*.

## The invariants — never cut, in any mode, at any tier

1. **A requirement is removed only by the user**, in their own words, quoted into the manifest. You may defer; you may never drop.
2. **A secret is never requested, echoed or written** — not into a file, prompt, commit or report. *Which* provider is a question; the key is not.
3. **A fact about the user is never invented** — prices, texts, addresses, accounts stay visible placeholders (`[ЦЕНА — впиши]`, never `4990 ₽`).
4. **Irreversible or outward-facing actions are a question** — deploy, publish, pay, message a third party, delete data, rewrite history. `auto` included.
5. **A chunk's diff never enters your context.** `git diff --stat` at most.

## Token discipline — why this skill stays light

Rules, not advice. Breaking one costs money on every remaining turn.

- **One phase file, when that phase starts.** Never ahead, never twice.
- **After a compaction re-read `plan.md`, never the phases.** The thread was never in them.
- **Test output is always truncated:** `<команда> 2>&1 | tail -20`.
- **Subagents get paths, never pasted files.** They have a filesystem.
- **Never re-read a file you wrote this session.**
- **One plain line to the user per closed chunk** — no tables, no progress reports, no restating the plan, no summarising your own work back at them.
- **Look facts up, ask only decisions.** What stack the repo uses is a fact.

## The dials

Everything typed after the invocation splits into **mode**, **depth** and **brief**. Bare words, no dashes; anything unrecognised is brief. Decided once, announced once, never revisited. Ambiguity → `semi` / `normal`. A dial may be switched mid-run; it applies from the next phase, and passed phases are not replayed.

| Mode | Triggers | What the user is asked |
|---|---|---|
| **auto** | «полный автомат», «ничего не спрашивай», `auto` | nothing. Forks become `ПРИНЯТО ЗА ТЕБЯ` rows in the report |
| **semi** *(default)* | — | only forks whose two branches give a visibly different product |
| **interview** | «погриль меня», «допроси», `interview` | every genuine fork, one at a time |

| Depth | Triggers | Elaborating a requirement | New capabilities (`A##`) |
|---|---|---|---|
| **strict** | «строго по брифу» | only what it cannot work without | **forbidden** |
| **normal** *(default)* | — | where it plainly helps | allowed, with a parent |
| **deep** | «проработай глубоко» | every dimension of every requirement | encouraged, with a parent |

**There is no approval mode and no polish loop here** — both are pauses, and pauses are what this skill exists to remove. On «согласовывай каждый шаг» the honest answer is that this is not that tool; on «вылижи до эталона», that there is no polishing round, and the way to get one is to say what specifically is wrong and let it be a new run.

## Tiers — read from the product, never from the length of the brief

| Tier | The product looks like | Chunks | Who writes the code |
|---|---|---|---|
| **T0** | one page, script, form, endpoint | 1 — the whole task, for the gates and the screen | you, one pass |
| **T1** | one coherent feature over one data shape | 2–4 | you, chunk by chunk |
| **T2** | several features, or one across store / logic / interface / integration | 5–10 | **a subagent per large chunk** |

**T0 is common and correct** — «Задача небольшая, собираю сразу, без разбивки». Above 10 chunks: say so in one line and **build anyway**. Stopping to ask the user to do something is the pause this skill exists to remove.

## The flight

| Phase | Read | Produces |
|---|---|---|
| **0 Подготовка** | nothing — it is below | memory file chosen, mode announced |
| **1 План** | `phases/1-plan.md` | `brief.md`, `plan.md` — gates G1, G2 |
| **2 Сборка** | `phases/2-build.md` | code, commits |
| **3 Приёмка** | `phases/3-final.md` | blind acceptance (G3), memory file, report |

Whenever the user sees a stage named, it is one of these words: `Подготовка · План · Сборка · Приёмка · Готово`, and no others — one vocabulary, not two. **Единица работы — «кусок»**, never «таск» or «тикет»; «задача» is what the user ordered.

## The gates

A failed gate sends the phase back **once**. They are checks against the user's own words, not requests for their time — **no mode and no tier skips one.**

| Gate | Where | Passes when |
|---|---|---|
| **G1** | phase 1, after the questions | every manifest row has a status; nothing `open` without a recorded reason |
| **G2** | phase 1, after the cut | **forward:** every live requirement is in ≥1 chunk. **backward:** every chunk names ≥1 requirement — a chunk tracing to nothing is work nobody ordered |
| **G3** | phase 3 | **blind acceptance:** a subagent given **only `brief.md` and the repo** reports what is actually built. Every disagreement goes in the report |

**G3 is the gate that earns the whole framework.** Everything before it measures the build against *your paraphrase* of the задача; G3 measures it against the задача. Every tier, T0 included. One subagent, never optional.

## The run always ends

Every loop in this skill has a floor, and they are collected here because the orchestrator's context is the one place that always holds them. **A run that cannot finish is worse than a run that finishes with three honest lines in «Что пошло не по плану».**

| Loop | Floor | What happens at the floor |
|---|---|---|
| a chunk fails its check | **2 repair attempts**, and a `BLOCKED` return counts as one | chunk → `blocked`, carry on with what does not depend on it, one line in the report |
| a chunk outgrows its context | **split once** | the remainder that outgrows a second time → `blocked`, not a third chunk |
| a gate fails (G1, G2) | **one redo of that phase** | what still will not resolve becomes `deferred` with the reason, and goes in the report |
| **G3 — the blind acceptance** | **runs exactly once per flight** | its findings are fixed and the fix is proved by running the code, never by a second blind check |
| the user adds requirements mid-flight | **no floor — it is their run** | but every addition is priced out loud, and past ~3 you say plainly that this has become a second project |

**G3 is the one to be careful about, because re-running it feels like diligence.** It is not: the checker gets a repository that has changed since it last looked, so what comes back is a *fresh opinion*, not a confirmation of your fix — there is no state in which it says «да, теперь всё». Each lap costs more than every instruction in this skill put together, and two agents handing work back and forth with no counter between them is how a run burns an afternoon and ships nothing. **Fix what it found, prove the fix by running the thing, write the rest into the report.** A finding too big to fix that way is not a second lap; it is a line in «Что пошло не по плану» and, if the user wants it, a new run.

The same logic is why there is no per-chunk reviewer here at all: a reviewer that can send work back is a loop, and a loop needs a counter more than it needs an opinion.

## Subagents — two cases, and no others

1. **An executor per chunk — at T2, or any chunk clearly over ~4 files** that is independent of what you are holding. The reason is invariant 5 and nothing else. Below that, inline: a cold start costs 20–40k tokens of re-orientation, more than the chunk itself. At most two in parallel, only with disjoint zones.
2. **The blind acceptance** — always, once, at the end.

No per-chunk reviewer, no craft reviewer, no memory or ADR subagent. Their job is done by running the code after every chunk and by G3.

## Files this skill owns

```
.gucci/
├── brief.md        the user's words verbatim after redaction; changes appended.
│                   The only file the blind acceptance is ever given
├── plan.md         manifest + short spec + chunks as checkboxes + project rules;
│                   Phase 3 adds the acceptance result and the end-of-run line
└── archive/<дата>/ the previous run's brief.md and plan.md, moved here by Phase 0
CLAUDE.md | AGENTS.md   the project as the next session finds it, between markers
```

Committed, not ignored — it is the user's record of what was promised and what was delivered. The live run always sits at those two fixed names; only the archive carries a date. No state file, no dashboard, no `--wip`, no per-chunk files, no `interfaces.md`, no ADRs, no HTTP server. **`plan.md` is the whole run state:** its boxes say where the build stands, its `Слепая приёмка:` line says the blind check has run, and its last line says whether the run has ended.

## Phase 0 — Подготовка

Nothing here is a question. Process decisions, one turn.

**1. Look before writing.** `git rev-parse --show-toplevel`; `CLAUDE.md` / `AGENTS.md`; `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml`; `.gucci/`. Anything readable is a fact, not a question.

**2. Is `.gucci/` already there?** Three different situations, and telling them apart is the whole of this step:

- **`plan.md` without a `Прогон завершён` line → this is a resume.** Read `brief.md` and `plan.md`, say where things stand in one line («Продолжаю: 4 из 7 готово, следующий — корзина»), and continue from the first chunk still marked `[ ]`. Every chunk closed → Phase 3; a `Слепая приёмка:` line already in `plan.md` means the blind check has run and does not run again. Do not redo finished phases or re-ask answered questions. A chunk left half-done with nothing committed behind it starts over.
- **`plan.md` ends with `Прогон завершён: …` → the previous run landed, and this is a new one.** **Archive before writing anything**, or the new brief silently destroys the record of what was promised last time:

  ```bash
  A=$(git rev-parse --show-toplevel 2>/dev/null || pwd -P)/.gucci
  P=$A/archive/$(date +%Y-%m-%d-%H%M)
  mkdir -p "$P" && mv "$A/brief.md" "$A/plan.md" "$P/" 2>/dev/null
  ```
- **`plan.md` changed within the last few minutes and no other window of yours accounts for it** — the run is going on somewhere else. Say what you see and ask which one carries on. Do not archive, do not overwrite.

**3. Memory file.** `CLAUDE.md` → else `AGENTS.md` → else `CLAUDE.md`. Never a question. What this skill writes lives between `<!-- gucci-code:start -->` and `<!-- gucci-code:end -->`; **anything outside those markers is untouchable.**

**4. Git.** No repo → `git init`, and `.env`, `.env.*` (not `.env.example`), `node_modules/`, `__pycache__/` ignored before anything is created. Dirty tree → say so in one line and carry on; never stash, reset or clean the user's work.

**5. Announce, once, and do not wait for a reply.** The only place the dials are ever named.

```
Ярус T1 · режим полуавтомат · глубина обычная.
Спрошу только то, что в задаче не определено, дальше соберу сам.
Переключить в любой момент: «полный автомат» · «погриль меня» · «строго по брифу» · «проработай глубоко».
```

Then straight into Phase 1. Waiting for an answer to this block is exactly the pause this skill exists to remove.

## Judgement

The numbers here — tiers, chunk counts, question counts — are calibration for a first guess, never targets. A plan cut to land inside a tier has optimised for the rule instead of for the person who asked. Where following a rule would make the result worse, break it deliberately, say so in one line, and carry on. What is never acceptable is breaking one quietly.
