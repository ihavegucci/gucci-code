# Phase 3 — Приёмка

Everything until now measured the build against **your paraphrase** of the задача. This phase measures it against the задача.

## 1. Run the whole thing once

Not the chunk you just finished — the project. Its test command, its build, its linter, whatever «Границы и правила проекта» names, truncated as always: `<команда> 2>&1 | tail -20`. Then start it the way the user will and confirm it comes up.

A red test here is not a formality to note in the report. Fix it — Phase 2's rules still apply — and only then go on. The result goes into `state.js` as `tests`, one line: `"npm test — 14 passed"`.

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

The tally goes into `state.js` as `blind` — `{ "ok": 10, "partial": 1, "missing": 1, "unchecked": 0 }`, counted over the requirements **the checker found in the brief**, not over your manifest. Anything it qualified is `partial`; a run that files every caveat as `ok` puts a clean green line on the screen over a build that fails on the user's real data. The dashboard shows `missing` in red, which is the point: it is the one number the run cannot argue with.

**Ask for the answer short.** On a one-file project this check cost ~50k tokens — more than every instruction in this skill put together, and on a T0 run it is the single largest expense of the flight. It is still worth it, and it is still not optional; it is the reason the last line of the prompt forbids fixing, suggesting and writing code, and the reason nothing here asks it for a second opinion.

## 3. The project memory — what the next session finds

Between `<!-- gucci-code:start -->` and `<!-- gucci-code:end -->` in the memory file chosen in Phase 0. **Everything outside those markers is untouchable**, and if the markers are already there from an earlier run you replace what is between them, never append beside them.

Ten to twenty lines, written from the finished code, not from the spec:

```markdown
<!-- gucci-code:start -->
## Что это

Телеграм-бот для заявок на ремонт. Заявки уходят в Google-таблицу.

## Как запустить

`npm start` — нужен `.env` с TELEGRAM_BOT_TOKEN и GOOGLE_SHEET_ID. Тесты: `npm test`.

## Как устроено

- `src/bot/` — диалог с клиентом, состояние в `data/sessions.json`
- `src/sheets/` — единственное место, которое ходит в Google API

## Что решено и почему

- Таблица вместо базы: пользователю нужно видеть заявки самому.
- Статус раз в минуту — Google Sheets не отдаёт его в реальном времени.

## Что не сделано

- Админки нет: в задаче её не было, заявки смотрят в таблице.
- Цвета студии — заглушки в `src/styles/brand.css`.
<!-- gucci-code:end -->
```

«Что решено и почему» is the only part of `.gucci/` worth outliving the run. The spec is worthless the day the work ships; the reasoning inside it is worth something for years and dies with the folder unless this section carries it out. This skill has no ADRs — this is where they went.

## 4. The report

The last thing the user reads. Plain language, no phase names, no process. **Every line comes from a file, not from memory** — the manifest, the blind check, `state.js`.

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

## 5. Close the instruments

In `state.js`: the `final` stage → `done`, **the `done` stage → `done` as well** (never `active` — a finished run showing its last stage as still running is the one thing the dashboard exists to make impossible), `finishedAt` set, `updatedAt` moved. `finishedAt` is what freezes the clocks and stops the page polling; a finished run whose timer keeps counting reads as a build that never ended.

Nothing to stop and nothing to kill: there is no server here. Say the path once if the user may want it later, and stop.
