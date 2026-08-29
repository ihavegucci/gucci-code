# Phase 2 — Сборка

The plan is cut. Now it gets built, chunk by chunk, and the one thing that must not happen is your context filling up with code.

## Inline or a subagent — decided once per chunk

**Default is inline: you write it yourself.** A subagent costs 20–40k tokens of re-orientation before its first line — reading the plan, exploring the code, finding the test command. Most chunks are smaller than their own boundary.

**Send it down when, and only when:** the tier is **T2**, or the chunk plainly touches **more than ~4 files**, or it is genuinely independent of what you are already holding. At most **two in parallel**, and only with disjoint zones — two subagents writing one file overwrite each other and the loss is silent, which is the whole reason zones exist.

At **T0** there is one chunk and it is the whole task: build it in one pass, run it, tick it, commit, go to Phase 3. The five moves below still apply — the chunk exists so that the gates and the dashboard have something to hold, not so that it gets treated as a plan.

## What a subagent gets — paths, never pasted files

It has a filesystem. Pasting a file costs the tokens twice and goes stale the moment anything moves.

```
Ты пишешь один кусок проекта. Прочитай `.gucci/plan.md`:
раздел «Границы и правила проекта» — правила, которые нельзя вывести из кода,
и кусок 3 — твоя работа.

Сделай так, чтобы все критерии приёмки этого куска выполнялись, и прогони то, чем
проверяется проект (команда — в «Границах»). Если проверять ещё нечем — тестов в
проекте пока нет — прогони критерии вручную и напиши в «Проверено», чем именно.

Пиши только в своей зоне: src/bot/. За её пределы не выходи.
`.gucci/` только читай — ничего там не меняй. Не коммить, ничего не устанавливай сам.

Если для куска не хватает библиотеки или целой возможности — верни BLOCKED и назови,
чего нет. Не ставь зависимость сам и **не пиши её замену с нуля**: если ответ на
нехватку — это новая подсистема, а не пара строк склейки, это BLOCKED, а не работа.

Верни ровно это, без диффа и без пересказа кода:
DONE | BLOCKED
Что заработало: <2–4 строки, что теперь можно сделать в продукте>
Проверено: <команда и результат, одной строкой>
Швы: <что создал и чем будут пользоваться следующие куски — имена, не пути>
Заметил: <не больше трёх строк того, что мешает дальше; пусто — нормально>
```

**`BLOCKED` is a first-class answer, not a failure.** A missing dependency, a credential that does not exist, a contradiction in the plan — it comes back as `BLOCKED` with one line saying what is missing.

**Missing capability has three exits and only one is allowed.** Installing it is forbidden, and so is **writing your own** — the second is the one that gets taken, because it looks like resourcefulness rather than a violation. Measured on a trial run of this skill: told the PDF library was absent, an executor spent 90k tokens and 32 tool calls hand-rolling a PDF writer with TrueType font embedding — 389 lines of unrequested subsystem, working, and nobody had been asked whether they wanted it. The rule that stopped nothing said «не ставь зависимость сам».

So the bar is written in the prompt below and belongs in «Границы» too: **if the answer to something missing is a new subsystem rather than a few lines of glue, that is `BLOCKED`.** The user gets asked which library to add, or told plainly that this part needs one — both are cheap. A subsystem nobody ordered is not.

**Never ask for a diff and never read one.** `git diff --stat` is the most you ever look at. The return contract above is the whole of what you need to know — invariant 5 exists because a single pasted diff can cost more than the rest of the chunk.

## After each chunk — five moves, in this order

1. **Run it.** The project's own check — `npm test`, `pytest`, `go build`, whatever «Границы» names — **always truncated**: `<команда> 2>&1 | tail -20`. Nothing is `done` on a subagent's word alone. There is no reviewer here; this is what replaces one.

   **An empty run is not a pass.** `NO TESTS RAN`, `0 passed`, a build that compiled nothing — that is the harness reporting it had nothing to say, and reading it as green is how a chunk gets marked `done` on the strength of a command that never looked at it. Until the project has a real check, the acceptance criteria are what you run, by hand, one at a time.

   **Then check where it wrote**, one command, no diff: `git diff --stat` (or `--stat HEAD`). Files outside the chunk's zone mean the boundary did not hold — and in a parallel wave that is the one failure that destroys the other chunk's work silently, because nothing errors. Say what it touched, fix the ownership, and if two chunks in one wave overlapped, serialise the rest of that wave.
2. **Tick the acceptance criteria** in `plan.md`, against what actually runs. A criterion you cannot check is a criterion that did not pass.
3. **Fold the seams forward.** Anything the next chunk must use — a function name, a table, an event, a config key — one line into «Границы и правила проекта». This is what keeps two fresh contexts agreeing about one project.
4. **Commit — and stage by path, never `git add -A`.** One chunk, one commit, plain language: `кусок 3: приём заявки от клиента`. Run the redaction gate over anything new in `.gucci/` before the first one.

   **`git add -A` on a tree that was already dirty sweeps the user's own uncommitted work into your commit**, and Phase 0 explicitly allows starting on a dirty tree. Stage the chunk's zone and `.gucci/`, nothing else:

   ```bash
   git add src/bot/ .gucci/ && git commit -qm "кусок 3: приём заявки от клиента"
   ```

   Never `push`, never `reset`, never `checkout --`, never rewrite history: a commit can be undone by the user, the others cannot. If the commit refuses — hooks, signing, an unconfigured identity — say so in one line and carry on building. **A failed commit is not a failed chunk**, and it is never a reason to start passing flags that skip hooks.
5. **`state.js`:** that chunk → `done` + `finishedAt`, `requirements.done` up, `updatedAt` moved; then the manifest rows it closed → `done`.

Then **one plain line to the user** and straight into the next chunk: «Заявки принимаются — бот доводит клиента до номера заявки.» No table, no summary of your own work, no restating what is left.

## When it comes back anything other than DONE

**A failing check is repaired where it broke.** Inline chunk → you fix it. Subagent chunk → the same subagent gets one more turn with what failed, in one line. **Never repair a subagent's chunk by reading its code yourself** — that is the diff entering your context by the back door.

**Two failed repairs on one chunk is a stop**, and a `BLOCKED` return counts as one of the two. Do not try a third with different phrasing, and do not restart it as a "fresh" chunk to reset the count — that is the same loop wearing a new number. Say in one line what does not work, mark the chunk `blocked` in `state.js`, move to the chunks that do not depend on it, and put it in the final report. A chunk quietly retried five times is how a run burns an afternoon with nothing to show.

**Chunks that depend on a `blocked` one are `blocked` too** — mark them, do not send them out to fail on a foundation that was never built, and count the whole group as one line in the report.

**A chunk that outgrew its context** — the subagent ran long and returned partial work — is not resumed inside that context. Its finished part is committed, the rest becomes a new chunk in `plan.md` with the remaining criteria, and it goes out fresh. There are no handoff files here: `plan.md` and the commits are the handoff. **This split happens once.** If the remainder outgrows a context too, the chunk was mis-cut and no further slicing will fix it: mark it `blocked`, say so, and let the report carry it.

## When the build proves the plan wrong

The code may correct the plan. It may never correct the brief.

A data model that does not hold, an interface that cannot exist, a library that does not do what it claims — amend the spec section of `plan.md`, add a manifest row marked `D##` with **what the code demonstrated**, and say it in one line: «Google Sheets не отдаёт статус в реальном времени — статус будет обновляться раз в минуту, иначе нужен свой сервер.»

`D##` comes only from a real finding, never from an idea you had, and **it never retires a requirement.** If the finding means something the user asked for cannot be built as described, that is a question to them, not a `dropped` row — only they may cancel it.

## When the задача changes mid-build

«Убери SMS» at chunk four is ordinary. One procedure, wherever it arrives:

1. **`brief.md` first** — their words, verbatim and dated, under «Дополнения». First because it is the step that gets skipped, and the only file G3 will see.
2. **The manifest** — `dropped` with the quote, or a new `G##` row.
3. **The chunks** — the `G##` becomes a new chunk, a `deferred` row, or a line in the report. Say which in one line, **and say what it costs**: «Беру, но лендинг тогда сдвигается.» A requirement accepted silently mid-flight is a schedule the user never agreed to.

Work already built for a now-`dropped` requirement stays unless removing it is trivial. Say so in one line and move on; unbuilding is rarely what they wanted.

## What never happens during the build

- **You do not read diffs**, and you do not ask for them.
- **You do not re-read `plan.md` between chunks** — you wrote it this session. After a compaction, re-read `plan.md` and `state.js`, never the phase files.
- **You do not open the next phase file.** Phase 3 is read when the last box is ticked.
- **You do not invent a fact about the user** to unblock yourself. A missing price is a visible placeholder and a line in the report, never a plausible number.
- **You do not deploy, publish, pay or message anyone** because it looked like the natural next step. That holds in `auto` too.
