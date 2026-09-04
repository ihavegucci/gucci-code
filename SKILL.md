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
- **After a compaction re-read `plan.md` and `state.js`, never the phases.** The thread was never in them.
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
| **0 Подготовка** | nothing — it is below | `state.js`, `dashboard.html`, mode announced |
| **1 План** | `phases/1-plan.md` | `brief.md`, `plan.md` — gates G1, G2 |
| **2 Сборка** | `phases/2-build.md` | code, commits |
| **3 Приёмка** | `phases/3-final.md` | blind acceptance (G3), memory file, report |

Five stages on screen: `Подготовка · План · Сборка · Приёмка · Готово`. The user sees exactly these words and no others — one vocabulary, not two. **Единица работы — «кусок»**, never «таск» or «тикет»; «задача» is what the user ordered.

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
├── plan.md         manifest + short spec + chunks as checkboxes + project rules
├── state.js        the run state — the only file the dashboard reads
├── dashboard.html  copied once from the skill, never edited
└── archive/<дата>/ the previous run's brief.md and plan.md, moved here by Phase 0
CLAUDE.md | AGENTS.md   the project as the next session finds it, between markers
```

Committed, not ignored — it is the user's record of what was promised and what was delivered. The live run always sits at those four fixed names; only the archive carries a date. No `--wip`, no per-chunk files, no `interfaces.md`, no ADRs, no HTTP server.

## Phase 0 — Подготовка

Nothing here is a question. Process decisions, one turn.

**1. Look before writing.** `git rev-parse --show-toplevel`; `CLAUDE.md` / `AGENTS.md`; `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml`; `.gucci/`. Anything readable is a fact, not a question.

**2. Is `.gucci/` already there?** Three different situations, and telling them apart is the whole of this step:

- **Unchecked boxes in `plan.md` → this is a resume.** Read `brief.md`, `plan.md`, `state.js`, say where things stand in one line («Продолжаю: 4 из 7 готово, следующий — корзина»), and continue from the first unchecked box. Do not redo finished phases or re-ask answered questions. A chunk left half-done with nothing committed behind it starts over.
- **Every box checked, or `finishedAt` is set → the previous run landed, and this is a new one.** **Archive before writing anything**, or the new brief silently destroys the record of what was promised last time:

  ```bash
  A=$(git rev-parse --show-toplevel 2>/dev/null || pwd -P)/.gucci
  P=$A/archive/$(date +%Y-%m-%d-%H%M)
  mkdir -p "$P" && mv "$A/brief.md" "$A/plan.md" "$P/" 2>/dev/null
  ```

  Then a fresh `state.js` — new `startedAt`, `finishedAt` back to `null`, all five stages back to `pending` except `setup`, `chunks` empty, counters zeroed. **Reusing the finished run's `state.js` gives the user a dashboard that is all green before the new work has started**, and `finishedAt` left in place stops the page polling, so it never turns back.
- **`state.js` written within the last few minutes and no other window of yours accounts for it** — the run is going on somewhere else. Say what you see and ask which one carries on. Do not archive, do not overwrite.

**3. Memory file.** `CLAUDE.md` → else `AGENTS.md` → else `CLAUDE.md`. Never a question. What this skill writes lives between `<!-- gucci-code:start -->` and `<!-- gucci-code:end -->`; **anything outside those markers is untouchable.**

**4. Git.** No repo → `git init`, and `.env`, `.env.*` (not `.env.example`), `node_modules/`, `__pycache__/` ignored before anything is created. Dirty tree → say so in one line and carry on; never stash, reset or clean the user's work.

**5. Raise the instruments.**

```bash
A=$(git rev-parse --show-toplevel 2>/dev/null || pwd -P)/.gucci
mkdir -p "$A"
T=$(find -L ~/.claude/skills ~/.agents/skills .claude/skills -maxdepth 4 -iname dashboard.html -ipath '*gucci*' 2>/dev/null | head -1)
if [ -n "$T" ]; then cp "$T" "$A/dashboard.html"; else echo "шаблон не найден — прогон пойдёт без дашборда"; fi
```

`if`, not `mkdir -p "$A" && [ -n "$T" ] && cp …`: that chain returns non-zero whenever the template is missing, and a Phase 0 that reports failure for an instrument it can live without is a Phase 0 you will start debugging instead of flying.

`find -L`, because skills are installed as symlinks and a plain `find` reports nothing while the file sits right there. No template found → widen the search once by hand, then carry on without a dashboard: it is an instrument, not a gate.

Then write `.gucci/state.js` — **first line exactly `window.STATE =`**, then indented JSON. That shape is not decoration: it loads through `<script src="state.js">`, which works over `file://` where `fetch` does not. That one fact is why this skill needs no server.

```js
window.STATE =
{
  "title": "Телеграм-бот для заявок на ремонт",
  "mode": "semi", "depth": "normal", "tier": null,
  "startedAt": "2026-08-29T14:02:06+03:00",
  "updatedAt": "2026-08-29T14:02:06+03:00",
  "finishedAt": null,
  "stages": [
    { "id": "setup", "status": "active", "startedAt": "2026-08-29T14:02:06+03:00" },
    { "id": "plan",  "status": "pending" },
    { "id": "build", "status": "pending" },
    { "id": "final", "status": "pending" },
    { "id": "done",  "status": "pending" }
  ],
  "requirements": { "total": 0, "done": 0, "placeholder": 0, "deferred": 0, "dropped": 0 },
  "chunks": [],
  "tests": null,
  "blind": null
}
```

**ISO 8601 with the offset**, from `date -Iseconds` at the moment the thing happens, never estimated and never copied from another row — a bare `14:50` gives an invalid date and a dead clock, and a guessed one gives a dashboard that quietly disagrees with itself. **Never put a secret value in this file.**

`title` is yours to write here, from what the user has already said — `brief.md` does not exist yet, and the dashboard opens before Phase 1 runs. Three or four words naming the thing, not a restatement of the задача.

Three fields start `null` and are filled later; each is written by exactly one place, and the dashboard shows nothing until it is:

- **`tier`** — by Phase 1 the moment the tier is decided, **before the cut**. Until it is there the header has no ярус at all.
- **`tests`** — by Phase 3, one line: `"npm test — 14 passed"`.
- **`blind`** — by Phase 3, the tally of the blind acceptance: `{ "ok": 10, "partial": 1, "missing": 1, "unchecked": 0 }`.

Then open it once, by you — the user should not be told where a file is and sent to find it:

```bash
D="$A/dashboard.html"
W=$(cygpath -w "$D" 2>/dev/null || echo "$D")
open "$D" 2>/dev/null || xdg-open "$D" 2>/dev/null \
  || powershell -NoProfile -c "Start-Process '$W'" 2>/dev/null \
  || echo "открой вручную: $W"
```

**`cygpath -w` is what makes the Windows branch work at all.** In Git Bash `$A` is a POSIX path like `/c/Users/…`, and neither PowerShell nor Explorer can open one — without the conversion the launcher fails silently and even the printed path is unusable. On macOS and Linux `cygpath` does not exist, `W` falls back to `$D`, and `open` / `xdg-open` handle it before the PowerShell branch is ever reached.

A real browser loads `state.js` from beside it, so the clocks tick with no server. **Opened exactly once per flight** — never re-opened, never refreshed; the page re-reads `state.js` every ten seconds itself. A failure is not an error: print the path and carry on. Skip opening when `$SSH_CONNECTION` or `$CI` is set.

**6. The update ritual — for the rest of the run.** Edit the affected rows of `state.js` and move `updatedAt`. That is all: nothing to mirror, nothing to restart.

| When | What |
|---|---|
| entering a phase | that stage → `active` + `startedAt`; the one you left → `done` + `finishedAt` |
| launching a chunk | → `in-progress` + `startedAt`, **before** any code is written |
| chunk committed | → `done` + `finishedAt` |
| a chunk goes back for a fix | → `repair`; two failed repairs → `blocked` |
| **any manifest row changes status** | the matching counter in `requirements` |

**The counters move with the manifest, not at the end.** `done`, `placeholder`, `deferred` and `dropped` are each written the moment a row takes that status — a row set to `placeholder` in the briefing and counted in Phase 3 is a dashboard that spent the whole run claiming the work was cleaner than it was. `total` never moves after Phase 1 except when the user adds a `G##` row.

**Prove it still parses after every edit**, in the same turn:

```bash
tail -n +2 "$A/state.js" | python -m json.tool >/dev/null && echo ok
```

A `state.js` broken by an edit takes the dashboard to a blank screen and, worse, takes the resume with it — that file is what a compacted context reads to find out where the run is. It fails silently: nothing in your session errors, and the damage surfaces an hour later as a dashboard that stopped moving. If the check does not print `ok`, re-read the file and fix it before anything else. No python on the machine → skip the check and be correspondingly careful.

Anchor every edit on the `"id"` line above the field you are changing — `"status": "pending"` appears once per stage and once per chunk, and `replace_all` rewrites every row in one stroke. A stage the run walked past gets `skipped` **with a reason**, never left `pending`: a stage that never moves reads as a stuck build. `startedAt` goes in when the thing starts, not when it ends — an interval with a start and no end is what makes the clock run.

**7. Announce, once, and do not wait for a reply.** The only place the dials are ever named.

```
Ярус T1 · режим полуавтомат · глубина обычная.
Спрошу только то, что в задаче не определено, дальше соберу сам.
Дашборд открыл — .gucci/dashboard.html, обновляется сам.
Переключить в любой момент: «полный автомат» · «погриль меня» · «строго по брифу» · «проработай глубоко».
```

Then straight into Phase 1. Waiting for an answer to this block is exactly the pause this skill exists to remove.

## Judgement

The numbers here — tiers, chunk counts, question counts — are calibration for a first guess, never targets. A plan cut to land inside a tier has optimised for the rule instead of for the person who asked. Where following a rule would make the result worse, break it deliberately, say so in one line, and carry on. What is never acceptable is breaking one quietly.
