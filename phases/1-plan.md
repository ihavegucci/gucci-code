# Phase 1 — План

The brief is the only thing here the user actually authored; everything downstream is your paraphrase of it. This phase turns it into an artifact that survives every later rewrite, then cuts the work.

## 1. Redaction gate — before anything is written

Every piece of user text — the brief, every answer, every pasted fragment — passes this on its way to a file or a prompt.

`sk-…` `sk_live_…` `pk_live_…` · `ghp_…` `github_pat_…` · `AKIA…` `AIza…` `ya29.…` · `xoxb-…` `xoxp-…` · Telegram bot (8–10 digits, colon, 35 chars) · JWT `eyJ…` · `<scheme>://<user>:<pass>@<host>` · `-----BEGIN … PRIVATE KEY-----` · ≥32 chars of hex/base64 next to `key`, `token`, `secret`, `password`, `ключ`, `токен`, `пароль`.

On a hit: the value becomes `[REDACTED:<VAR_NAME>]` under the conventional name (`STRIPE_SECRET_KEY`, `TELEGRAM_BOT_TOKEN`, `DATABASE_URL`); that name goes into `.env.example`, empty; one line to the user — «Ты прислал ключ Stripe — я его не сохранил. Впиши сам в `.env`, а этот лучше отзови: он уже побывал в переписке.» — and the flight carries on.

**Never echo the value back**, not even to confirm what you found. Run the gate over `.gucci/` once before the first commit.

## 2. `brief.md` — their words, untouched

The redacted brief, **word for word**, into `.gucci/brief.md`.

```markdown
# Изначальная задача

> Текст задачи не редактируется — это эталон, с которым сверяется результат.
> Всё, что сказано позже, дописывается в «Дополнения» ниже.

<весь текст пользователя, дословно, после редакции секретов>

## Дополнения

- <дата> — «SMS не надо, только телега»
```

**No paraphrase, no cleanup, no reordering** — bad grammar, contradictions and half-sentences stay. A tidied brief is already a spec, and a spec is the thing you cannot check against. Everything counts: the idea, the asides, the constraints, the stack preference, the deadline mentioned in passing.

**The text above is never edited; the file goes on growing.** Everything said later is appended under «Дополнения», dated and verbatim. This is the easiest rule to skip, because updating the manifest row feels like enough — but **G3 is given this file and never `plan.md`**, so a change that reaches the manifest and not this file is invisible to the only check capable of catching a loss.

## 3. `plan.md` — the manifest

One file holds the manifest, the spec and the chunks. Write the manifest **before any question is asked** — the questions are themselves a re-encoding of the brief, so the anchor is dropped first.

Split the brief into the smallest units that can independently be true or false about the finished product. Every row carries **the exact words it came from**.

```markdown
# План

## Требования

Источник: `brief.md`. Строку из этого списка может снять **только пользователь**.

| ID | Из брифа (дословно) | Статус | Основание | Кусок |
|----|---------------------|--------|-----------|-------|
| R01 | «принимает заявки на ремонт техники» | in-chunk | — | 2 |
| R02 | «складывает их в Google-таблицу» | in-chunk | — | 3 |
| R03 | «и дублировать на SMS» | dropped | пользователь: «SMS не надо, только телега» | — |
| R04 | «фирменные цвета студии» | placeholder | цвета не переданы | отчёт |
| R05i | *(подразумевается)* кто-то должен читать заявки | deferred | админки в брифе не было | отчёт |
```

**Statuses.** `open` initial · `in-chunk` a chunk exists that delivers it · `done` built and checked · `placeholder` in the build with a visible stub where a user fact belongs · `deferred` consciously postponed, reason recorded, reported at the end · `dropped` **cancelled by the user, and only ever by them.**

- **`dropped` requires a quote** in Основание. No quote, no drop. Proposing a drop is a question, not a status change.
- **`deferred` is not `dropped`.** Postponing is yours; cancelling is not.
- **Silence never cancels anything.** A requirement the user stopped mentioning is still live — forgetting and deciding must not look alike.

**Marks.** `R##` from the brief, untouchable · `R##.n` deepening one of them, uncapped, this is the main work of the spec · `R##i` clearly implied but never said · `G##` the user's own words said later, untouchable like `R##` · `A##` a new capability you thought of, must name a parent, **forbidden at `strict`** · `D##` a constraint the build itself demonstrated, added in Phase 2 and only from a real finding.

Three of these are the same shape and mean different things, and getting them wrong is how a requirement quietly dies: **`G##` is the user's words, `A##` is your idea, `D##` is what the code proved.** A wish of theirs filed as `D##` retires a requirement nobody cancelled; an idea of yours filed as `G##` puts your taste into the one column meant to hold only theirs.

`R##i` rows are the most dangerous items in the flight — too obvious to state, too big to skip. «Принимает заявки» implies somewhere to read them; «интернет-магазин» implies a way to pay. Route each to a question; never invent it and never ignore it. In `auto` they become `ПРИНЯТО ЗА ТЕБЯ` lines in the report.

**How fine to cut.** One requirement = one thing that can independently be true or false. «Бот принимает заявки и складывает их в таблицу» → **two** rows: one can work while the other does not. «Красивый современный дизайн» → **one**, inherently untestable; either a question turns it into something checkable, or it stays a recorded matter of taste.

Then one line — «Разобрал задачу на 23 требования, держу их под контролем до конца» — and `requirements.total` into `state.js`. No table in the chat.

**Every later status change moves its counter in the same edit** — `placeholder`, `deferred` and `dropped` as they happen in the briefing below, `done` during the build. Counted at the end instead, they describe a run the user already stopped watching.

## 4. The questions

**One at a time.** Wait for the answer before the next; a wall of questions gets answered badly.

- **Every question names its requirement**, internally: «this asks about R07». A question that closes no row is one you invented for your own comfort — drop it.
- **Recommend an answer with every question**, so it can be accepted in one word: «Заявки складывать в Google-таблицу или сразу в базу? Я бы взял таблицу — тебе её видно и не нужен сервер.»
- **Blocking unknowns go first** — payment, hosting, which accounts exist, where data lives, what system this fits into. In the first three questions, never at the finish line: a payment question asked at the end costs half the project.
- **Decisions, never secrets.** *Which* provider, *whether* an account exists — yes. The key, the token, the connection string — never.
- **Never answer for the user.** Forced past an unknown → the row becomes `placeholder` and you move on.

**How many is an outcome of the brief, not a number you were handed.** Every blocking unknown is asked in every mode. A fork only the user can settle is asked: in `semi`, the ones whose branches lead to a visibly different product; in `interview`, all of them. Everything else you decide yourself — error wording, retry policy, defaults, naming, layout.

A decision goes back to the user if it **costs money or ties them to a vendor**, **changes what they see or can do**, **means rebuilding rather than editing to undo**, or **encodes a rule about their business** — prices, deadlines, who may do what, what happens to someone's data. When you cannot tell which side it falls on, that uncertainty *is* the signal: ask.

Calibration only: `semi` usually lands between two and eight, `interview` ten to twenty-five. **Nothing left open? Say so and go** — «Вопросов нет, в задаче всё однозначно».

**Untestable requirements** — «красиво», «удобно», «быстро» — have one cheap fix: «Есть сайты, на которые это должно быть похоже? Скинь два-три.» Record the answer verbatim in Основание. Do not spend three questions here.

**In `auto` there is no interview** — you run the same checklist against yourself, and the line between two kinds of answer is the whole discipline of that mode. **Decisions are yours:** stack, structure, provider, data model, layout — pick what runs on the user's own machine without a third-party account and without money, recorded as `ПРИНЯТО ЗА ТЕБЯ: …` and reported at the end. **Facts about the user are not:** prices, texts, addresses, business rules, accounts, brand colours become `placeholder` and visibly labelled filler. A plausible invented price is worse than an obvious blank — the blank gets fixed, the price gets shipped. A paid or account-bound service becomes an adapter with a local stub behind it, never a guess.

**Record after each answer, not at the end.** Resolves a requirement → the decision into Основание. **Cancels** one → `dropped` with their words quoted, the only path there. Raises something new → a `G##` row in their phrasing. «Не знаю» → `placeholder`, and the build gets a labelled stub. Anything that **cancels, adds or reverses** also goes into `brief.md` under «Дополнения» — not instead of the row, as well as it.

**Gate G1:** every row has a status; nothing `open` without a recorded reason. In `auto`, nothing `open` at all.

**A gate is redone once.** If a row still will not resolve on the second pass, it becomes `deferred` with the reason written down and appears in the final report. Going round a third time does not produce a new answer — it produces a run that never reaches the code.

## 5. The spec — short, and inside `plan.md`

**Skipped entirely at T0**: there is nothing a separate document would say that the manifest does not. At T1 and T2 it is four sections and stays under a page.

```markdown
## Решение

<2–5 предложений: что это за штука и как она устроена>

## Истории

1. Клиент пишет боту → отвечает на три вопроса → получает номер заявки.
2. Мастер открывает таблицу и видит новую строку.

## Решения по реализации

- Хранилище: Google Sheets — таблицу видно пользователю, сервер не нужен.
- Состояние диалога: в памяти, переживает рестарт через файл.

## Границы и правила проекта

- Стек и версии, команда запуска, команда тестов.
- Что трогать нельзя.
- Не хватает библиотеки или целой возможности — это BLOCKED. Не ставить самому
  и не писать замену с нуля: новая подсистема вместо пары строк склейки — это BLOCKED.
```

**Chunk 1 owes the project a check that discriminates.** On a trial run of this skill three executors in a row reported `python -m unittest -q` → «NO TESTS RAN», rc=5: the command named in «Границы» existed but could not tell «сломано» from «пусто», so the first chunks had nothing to prove themselves against and each improvised. Either the first chunk leaves one runnable test behind, or «Границы» says plainly what to do until the harness exists — and the orchestrator never reads an empty run as a pass.

**«Границы и правила проекта» is what executors read before they write anything** — the whole of what a fresh context cannot derive, and the reason this skill needs no separate `interfaces.md`. Keep it current: when a chunk builds something the next chunk must use, one line goes here.

**Depth decides how thorough the stories are, never how much there is to build.** A `deep` spec for a landing page is a long section about one page — still T0, still one pass.

## 6. The cut

Decide the tier from **what has to be built** (SKILL.md), **write it into `state.js` as `tier` before cutting anything**, then cut to it. Written first because the dashboard has no ярус until it is there, and because everything below reads the tier to decide how much to cut.

At **T0 nothing is cut apart, but the single pass is still chunk 1** — one row in `plan.md` and one in `state.js`, titled after the whole task, carrying every live requirement and the acceptance criteria. To the user you still say «Задача небольшая, собираю сразу, без разбивки»; the chunk is bookkeeping, not a plan.

**This is not ceremony — without it T0 cannot pass its own gates.** G1 in `auto` forbids a single `open` row, and G2 requires every requirement to sit in a chunk; a T0 that cuts literally nothing leaves every row `open` and outside any chunk, so the two gates fail by construction on the tier this skill calls the common one. One row costs a line and makes both checks mean something. It also gives the dashboard a progress bar instead of an empty list.

Every chunk is a **narrow but complete path through every layer it touches** — data, logic, interface, tests — not a horizontal slice of one layer. When it is done, something works end to end that did not before, and you can show it.

- Anything that must exist before the rest — the shell, the schema, the shared primitives — is chunk 1, alone.
- Number in dependency order, blockers first; give each chunk its **blockers** and its **zone** (the directories it owns).
- **Chunks closing `R` requirements come before chunks closing only `A`.** If the run is cut short, what is missing must be your additions, never the user's request.

**Two tests every chunk must pass.** *Payback:* would its subagent spend more effort flying in than building? Then merge it into the neighbour it depends on. *Neighbour:* under three acceptance criteria and touching the same files as an adjacent chunk? Then it is a checklist item inside that neighbour.

**The merge pass is mandatory.** After the draft, before writing anything: merge adjacent chunks touching the same files, any chunk under three criteria with a natural parent, and chains where B is blocked by A and A alone demos nothing. A draft that lands at 12 and merges to 6 was a T1 job pretending to be T2 — normal, and the reason this pass exists. **Cutting too fine feels careful and is the opposite:** every extra boundary is another chance for two contexts to disagree.

**Waves, at T2 only.** `wave = 1 + max(wave of its blockers)`; then split each wave by zone — two chunks in one wave that would write the same files cannot run together, so the later one moves to the next wave. Same files → serialise, always: two subagents editing one file overwrite each other and the loss is silent. A wave of one is a normal answer. Waves are *discovered* in the dependency graph, never designed into it, and assigned once — renumbering mid-run makes rows jump on the dashboard and reads as the agent losing the plan.

```markdown
## Куски

### [ ] 3 — Приём заявки от клиента
**Требования:** R01, R01.1 · **Blocked by:** 1, 2 · **Зона:** `src/bot/` · **Волна:** 2

Клиент пишет боту, отвечает на три вопроса — что сломалось, адрес, телефон — и получает
подтверждение с номером заявки. Обрыв связи не сбрасывает диалог.

Из брифа: «принимает заявки на ремонт техники»

- [ ] Диалог из трёх шагов доходит до подтверждения
- [ ] Номер заявки уникален и виден клиенту
- [ ] Прерванный диалог продолжается, а не начинается заново
- [ ] Незаполненный телефон даёт понятную ошибку, а не падение
```

The verbatim brief quote is not decoration: it is the last thing standing between a fresh context and a plausible reinterpretation of what was ordered, and it costs forty tokens.

**Every criterion is checked against the requirement's own wording, not against a convenient reading of it.** On a trial run of this skill the brief said «у заметки **должен быть** тег»; the criteria said the tag is stored and displayed, the build made it optional, and G1, G2 and two executors all passed it — the blind check at the very end was the first thing to say «частично». A criterion that is easier to satisfy than the sentence it came from is how a requirement quietly shrinks, and it shrinks in the one file that is supposed to prevent exactly that.

**Avoid file paths and code snippets** — they go stale faster than the chunk does. The exception is a structure prose states worse than code: a schema, a state machine, a type shape.

**Gate G2.** *Forward:* every live requirement appears in ≥1 chunk's Требования line — a requirement in no chunk does not get built. *Backward:* every chunk names ≥1 requirement, or a spec decision that traces to one; a chunk tracing to nothing is work nobody ordered. *Complete:* every chunk has a zone, at T2 a wave, and no two chunks in one wave share a zone.

Then set every covered row to `in-chunk` with its number, and **publish the chunks to `state.js` before a line of code is written**:

```json
{ "id": 3, "title": "Приём заявки от клиента", "requirements": ["R01","R01.1"],
  "blockedBy": [1,2], "wave": 2, "zone": ["src/bot/"], "status": "pending" }
```

A build running while the dashboard still says nothing was cut is broken instruments, and it breaks them at the moment the user is most likely to look.

## 7. Showing it

Write the files first — **a chunk that exists only in the dialogue is not a chunk.** Then one screen, plain language, no technical detail, one line per chunk saying what the user will be able to do when it lands. Parallelism gets one line and only if it is true: «6 кусков в 4 волны, часть пойдёт параллельно».

Then «Показываю план и начинаю. Скажи "стоп", если что-то не так» — and start. **Do not wait for approval**; waiting is the failure mode this skill exists to remove. Never promise a countdown: you cannot hold a pause, so a stated delay is a promise you will break. `interview` gets the same screen — the questions were the point of that mode, the chunk list was not; `auto` gets it as a notification.
