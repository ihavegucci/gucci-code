# Phase 3 — Приёмка

Everything until now measured the build against **your paraphrase** of the задача. This phase measures it against the задача.

## 1. Run the whole thing once

Not the chunk you just finished — the project. Its test command, its build, its linter, whatever «Границы и правила проекта» names, truncated as always: `<команда> 2>&1 | tail -20`. Then start it the way the user will and confirm it comes up.

A red test here is not a formality to note in the report. Fix it — Phase 2's rules still apply — and only then go on. The result opens a new last section of `plan.md`, `## Приёмка`, as one line: `Проверка: npm test — 14 passed`.

**The memory file is written after this, in step 3, and that order is load-bearing.** On a repeat run in the same repo it already carries an skill-written description of the project — which is your paraphrase, in the one place the blind check would happily read it. Never write it before the check.

## 2. Gate G3 — blind acceptance

**One subagent, every run, every tier, T0 included.** Not optional, not skippable by any mode.

It is given **`.gucci/brief.md` and the repository, and nothing else** — not `plan.md`, not the manifest, not the spec, not your summary. The moment it sees your paraphrase it starts checking the build against the paraphrase, which every step before this one already did.

```
В `.gucci/brief.md` — задача, как её сформулировал заказчик. Рядом лежит проект,
который должен её решать. Ничего другого о проекте не читай: ни план, ни спецификацию,
ни остальное из `.gucci/`, ни CLAUDE.md / AGENTS.md, ни README — только код.

Разбери задачу на требования сам и по каждому скажи, что реально есть в коде.
Где можешь — запусти и проверь, а не суди по именам файлов.

Верни таблицу: требование из брифа | есть / частично / нет / не смог проверить | одна строка почему.
Ниже — что в проекте есть такого, чего в брифе нет.
Ничего не чини, не предлагай улучшений, не пиши код.
```

**Read its answer against the manifest yourself.** Three kinds of disagreement, three meanings:

- **It says «нет», the manifest says `done`** — the serious one. Either something was lost, or it was built somewhere the check could not see it. Go look. Genuinely lost → build it now. There and unfindable → that is a real usability finding, and it goes in the report.
- **It says «нет», the manifest says `deferred` / `placeholder` / `dropped`** — expected, and it confirms the record is honest. Into the report as a known line.
- **It found something the brief never asked for** — either an `A##` you recorded, reported as an addition, or scope nobody ordered, reported as exactly that.
- **It says «есть», and then names a condition under which it is not** — «работает, но только на датах вида `YYYY-MM-DD`; на других тихо печатает "нет данных"». **This is the most valuable line the check produces and the easiest to file as a pass.** It is not a disagreement about presence, so it slips past the three cases above; what it describes is a requirement that works on the example and fails silently on the user's real data. Treat it as `partial`, never `ok`, fix it if it is cheap, and put it in the report either way.

**Every disagreement goes in the report, including the ones you resolved.** A blind check whose findings all quietly disappear is a check that was never run.

**A checker that could not run anything did not check anything.** If most of its rows come back «не смог проверить» — no dependencies installed, nothing runnable, it only read file names — the gate did not happen, and calling it passed is worse than admitting it: say so in the report in one line, in those words. Do not send it back in to try harder; that is the loop below.

**It runs once.** Fix what it found, prove the fix by running the code, and write the rest into the report — never by sending the checker back in. It would be looking at a repository that has changed since, so what returns is a fresh opinion, not confirmation; there is no answer it can give that ends the loop, and each lap costs more than every instruction in this skill together. A finding too big to fix that way is a line in «Что пошло не по плану», and a new run if the user wants it.

The tally goes into «Приёмка» the moment the checker returns — `Слепая приёмка: есть 10 · частично 1 · нет 1 · не смог проверить 0`, one line per disagreement under it — counted over the requirements **the checker found in the brief**, not over your manifest. Written then and not later, because after a compaction that line is the only proof the check has already run. Anything it qualified is `partial`; a run that files every caveat as `ok` puts a clean line in the report over a build that fails on the user's real data. `нет` is the one number the run cannot argue with.

**Ask for the answer short.** On a one-file project this check cost ~50k tokens — more than every instruction in this skill put together, and on a T0 run it is the single largest expense of the flight. It is still worth it, and it is still not optional; it is the reason the last line of the prompt forbids fixing, suggesting and writing code, and the reason nothing here asks it for a second opinion.

## 3. The project memory — what the next session finds

Between `<!-- gucci-code:start -->` and `<!-- gucci-code:end -->` in the memory file chosen in Phase 0. **Everything outside those markers is untouchable.** Exactly one such block per file: two whole blocks → merged into one where the first stood. A lone or broken marker → never guess where it ends — that guess deletes the user's text: write a fresh block at the end of the file, leave the fragment as it is, and say so in one line of the report.

**It is a snapshot of the project as it is now, never a log of how it got there.** The old block is the starting point, never something to add to: every line of it is brought to what the code is now — corrected, replaced or deleted — and the block describes the whole project, not just what this run built. A small fix in a big project changes a line or two; it does not shrink the block down to the fix. Nothing about runs goes in: no dates, no chunk numbers, no requirement IDs, no «добавлено», no «в прошлый раз». History already has two homes, `.gucci/archive/` and `git log`; a third copy in a file loaded into every session is what turns it into a dump.

At most twenty lines, written from the finished code, not from the spec:

```markdown
<!-- gucci-code:start -->
## Что это

Телеграм-бот для заявок на ремонт. Заявки уходят в Google-таблицу.

## Как запустить

`npm start` — нужен `.env` с TELEGRAM_BOT_TOKEN и GOOGLE_SHEET_ID. Тесты: `npm test`.
Заглушки, которые надо заполнить, ищи по `— впиши]`.

## Как устроено

- `src/bot/` — диалог с клиентом, состояние в `data/sessions.json`
- `src/sheets/` — единственное место, которое ходит в Google API

## Что решено и почему

- Таблица вместо базы: пользователю нужно видеть заявки самому.
- Статус раз в минуту — Google Sheets не отдаёт его в реальном времени.
<!-- gucci-code:end -->
```

«Как устроено» names directories, not files — a file list is stale by the next commit. **«Что решено и почему» holds only what the code still obeys, six lines at most:** a decision this run reversed is replaced, not kept beside its successor; one the code no longer reflects is dropped. It is the only part of `.gucci/` worth outliving the run — the reasoning dies with the folder unless it is carried here — but it earns its place by being true today, not by having been decided once. What was left undone belongs in the report, not here: it is stale the day the user fills the stub. The one exception is placeholders still in the code — if there are any, one line in «Как запустить» says how to find them, by the mark they actually carry; none left → no line.

## 4. The report

The last thing the user reads. Plain language, no phase names, no process. **Every line comes from a file, not from memory** — the manifest and the «Приёмка» section of `plan.md`.

**Before the report — one test over the manifest, fix what it names:**

```bash
grep -E '^\|.*\| *(open|in-chunk) *\|' .gucci/plan.md || echo ok
```

Anything printed instead of `ok` is a row stuck at `open` or `in-chunk`; set it `deferred` with its reason.

```markdown
## Готово

Телеграм-бот принимает заявки и складывает их в таблицу. Клиент получает номер
заявки, мастер видит новую строку. Запуск: `npm start`.

## Что нужно от тебя

- Вписать в `.env`: TELEGRAM_BOT_TOKEN, GOOGLE_SHEET_ID — пустые, я их не видел.
- Цвета студии: сейчас заглушки в `src/styles/brand.css`.
- Тексты приветствия — заглушки, ищи `[ТЕКСТ —`.

## Что не вошло

- Админка для мастера — в задаче её не было, заявки смотрят прямо в таблице.
- SMS-дублирование — ты снял: «SMS не надо, только телега».

## Что я добавил сверх заказанного

- Повтор заявки при обрыве связи — без него терялась каждая прерванная.

## Что пошло не по плану

- Статус обновляется раз в минуту: Google Sheets не отдаёт его в реальном времени.
- Кусок 6 не собрался — падает на импорте библиотеки платежей, оставил как есть.

## Где что лежит

- Задача и план — `.gucci/`
- Как это устроено — `CLAUDE.md`
```

**Rules for the report:**

- **Empty sections are removed, not filled with «нет».**
- **«Готово» says what does not work, in its own first paragraph, whenever anything is `blocked` or `missing`.** A headline that reads as a finished project over a run with a dead chunk is the single most expensive line this skill can produce: the user stops reading exactly there. «Заявки принимаются и попадают в таблицу. Оплата не работает — не собралась библиотека платежей, подробности ниже.»
- **«Что нужно от тебя» comes before anything the user might skip** — empty env names, placeholders, anything blocking the thing from running.
- **Every `deferred`, `placeholder`, `dropped`, every `ПРИНЯТО ЗА ТЕБЯ` from `auto`, and every disagreement the blind check found appears here.** This is the one place they all surface, and leaving one out is the failure this whole framework is built to prevent.
- **No apologising, no process, no phase names, no «как я работал».**
- Secrets are named, never shown — through the last line of the run.

## 5. Close the run

The last line of `plan.md`: `Прогон завершён: <date -Iseconds>`. It is what tells the next start in this repo that this run landed rather than stopped — without it, Phase 0 reads `.gucci/` as a run to resume.

Then one last commit, by path exactly as in Phase 2 — never `git add -A`, never `push`: `.gucci/`, the memory file, and whatever the fixes after the blind check touched.

```bash
git add .gucci/ CLAUDE.md src/bot/ && git commit -qm "приёмка: телеграм-бот для заявок"
```

**Skipped, it undoes the run's ending:** the end-of-run line, the acceptance result, the memory file and the last fixes all sit in the working tree, and one `git checkout .` or a fresh clone turns a finished run back into one to resume. A refused commit is one line to the user, as in Phase 2, never a reason to skip hooks. Nothing to stop and nothing to kill: there is no server here. Then stop.
