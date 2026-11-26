# Tasks

> Правила agent-pack, общие для групп 3–7. Править только `canon/`, `refs/`, `skills/`, `dist/` руками не трогать.
> Тело агента не растёт (`canon/baseline.json`): детали — в `refs/<agent>/*.md` с триггером «когда читать»; в теле — одна строка-указатель.
> Проверка каждой группы с правками canon: `pwsh scripts/generate.ps1`, `sh scripts/validate.sh` и `node test-smoke.js` проходят, тело каждого затронутого агента ≤ baseline.

## 1. Внешние зависимости (запускаются параллельно, не блокируют остальное)

- [ ] 1.1 Подготовить текст запроса администратору TestRail на поля `Dev coverage` (Dropdown: Не оценено [по умолчанию] / Нет / Частично / Полностью) и `Dev autotests` (Text) по D17 и сохранить его в `openspec/changes/testrail-autotest-coverage/requests/testrail-fields.md`. Проверка: в тексте есть типы полей, значения, проект и обоснование
- [ ] 1.2 Подготовить постановку задачи SRE на Python-джобу отчёта покрытия по спецификации `qa-coverage-report` и схеме матрицы (ссылка на `refs/qa/coverage-matrix.md` из задачи 2.2). Сохранить в `openspec/changes/testrail-autotest-coverage/requests/sre-coverage-report.md`, отправлять после 2.2. Проверка: постановка ссылается на оба документа, перечисляет входы (матрица, результаты тестов, TestRail только на чтение) и артефакты `coverage-report.html` / `coverage-report.json`

## 2. Референсы QA: шкала, схема матрицы, формат кейса

- [ ] 2.1 Создать `refs/qa/coverage-scale.md`: три статуса шага, вывод статуса кейса, ограничение по уровню теста, зависимости от стенда (C4), «не оценено ≠ НЕТ», правило для кейсов не про API, примеры из design. Проверка: каждый Requirement спецификации `test-coverage-scale` отражён в разделе файла; дисклеймер refs в начале
- [ ] 2.2 Создать `refs/qa/coverage-matrix.md`: YAML-схема матрицы (поля `case`, `ac`, `planned`, `actual`, `steps.covered`, `steps.logical`, `tests[].name`, `tests[].steps`, `downgrade`, `reason`, `origin` со значениями architect | tester | stage2-added | existing), правила полных имён тестов, пример. Проверка: пример из файла разбирается `python3 -c "import yaml…"` без ошибок и содержит все поля схемы
- [ ] 2.3 Обновить `refs/qa/testrail-format.md`: Type по умолчанию берётся из существующих кейсов секции (на проекте — `API`), References при REUSE/UPDATE дописываются, State не трогается, поля `Dev coverage` / `Dev autotests` с пометкой «системные имена — после создания администратором». Проверка: в файле нет утверждения «по умолчанию 6 Functional» без оговорки
- [ ] 2.4 Обновить `refs/qa/testrail-coverage.md`: поиск в секциях затрагиваемых endpoint'ов и в `Change Request/*`, найденный кейс сразу с C-id и решением REUSE/UPDATE, поля покрытия читаются как подсказка и подтверждаются поиском трейта в коде, отметка «покрытие не оценено» для старых кейсов. Проверка: каждый сценарий требования «Поиск существующих кейсов…» спецификации `coverage-planning` отражён в файле

## 3. Скилл testrail-api: создание секции

- [ ] 3.1 Добавить `skills/testrail-api/scripts/add_section.sh` и PowerShell-компаньон (параметры: project, suite, parent section, name) по образцу существующих скриптов (`common.sh`, обработка 429). Проверка: `bash add_section.sh` без аргументов печатает usage и выходит с ненулевым кодом; на тестовом проекте TestRail секция создаётся, и `get_sections.sh` её показывает
- [ ] 3.2 Описать `add_section` в `skills/testrail-api/SKILL.md` (таблица скриптов, пример). Проверка: скрипт есть в таблице, пример вызова совпадает с usage скрипта

## 4. Стадия 1: analyst, qa (дизайн и выгрузка), planner

- [ ] 4.1 Создать `refs/qa/modes.md` с режимами «дизайн» (черновики TC-xx в `docs/tests/<EPIC>/test-cases.md`, поиск существующих кейсов, без записи в TestRail и прогона) и «выгрузка» (ждёт Jira-ключ среза, новые кейсы в `Change Request/<KEY>`, при отсутствии — создание секции под HUMAN GATE, References = ключ среза, read-back, замена TC-xx на C-id в `test-cases.md` эпика и в «Покрытии кейсов» планов). Проверка: каждое требование `coverage-planning` про qa отражено в файле
- [ ] 4.2 Обновить `canon/qa.md`: определение режима по запросу (дизайн / выгрузка / приёмка) и указатель на `refs/qa/modes.md`; `editPaths` разрешает `docs/tests/**` и раздел «Покрытие кейсов» в `docs/plans/**`. Проверка: тело ≤ baseline; генерация и смоук проходят
- [ ] 4.3 Обновить `canon/analyst.md`: `subagents` += `qa`; между Гейтом 1 и делегированием planner — вызов qa в режиме «дизайн»; planner получает путь к черновикам. Проверка: тело ≤ baseline (детали — в `refs/analyst/review-handoff.md`); генерация проходит валидацию subagents
- [ ] 4.4 Создать `refs/planner/coverage-intent.md`: раскладка TC-xx по срезам через AC, правило «кейс на стыке — в срез последнего AC», раздел «Покрытие кейсов» в `<slug>.plan.md` (формат по `refs/qa/coverage-matrix.md`, поля `planned`), поиск существующих тестов для старых кейсов, оценка автотестов от намерения. Обновить `canon/planner.md` одной строкой-указателем. Проверка: тело planner ≤ baseline; пример раздела из refs валиден по схеме 2.2
- [ ] 4.5 Обновить `canon/commands/task-start.md`, чек-лист стадии 1: «после выгрузки qa — закоммитить артефакты с C-id». Проверка: пункт есть в чек-листе; генерация проходит

## 5. Стадия 2: architect, spec-auditor, tester, reviewer, coder

- [ ] 5.1 Создать `refs/tester/coverage.md`: поиск существующих тестов до проектирования, AT-xx в матрице при отсутствии архитектурного шага (пометка «спроектировано tester'ом»), трейт `[Trait("TestRail","C<id>")]` (эквиваленты NUnit/MSTest), заполнение `actual` и полных имён тестов, понижение с причиной, `stage2-added`, `existing`. Обновить `canon/tester.md` указателем. Проверка: тело ≤ baseline; каждое требование `autotest-traceability` про tester отражено в refs
- [ ] 5.2 Обновить `canon/solution-architect.md`: в плане верификации — список AT-xx с привязкой к AC и шагам кейсов, поиск существующих тестов для REUSE-кейсов (детали — в новом `refs/solution-architect/` или в `refs/tester/coverage.md` по ссылке). Проверка: тело ≤ baseline
- [ ] 5.3 Обновить `canon/spec-auditor.md`: проверка «каждый кейс с намерением full/partial обеспечен AT-xx нужного уровня на заявленные шаги». Проверка: тело ≤ baseline; формулировка соответствует требованию «Аудит AT-xx против намерения покрытия»
- [ ] 5.4 Обновить `canon/reviewer.md` и `canon/coder.md`: метка TestRail — разрешённое исключение из запрета трассировки; reviewer сверяет ассерты помеченных тестов с шагами кейса, матрицу с фактом и валидность YAML-блока. Проверка: тела ≤ baseline; в `coder.md` правило о трассировке упоминает исключение
- [ ] 5.5 Обновить `canon/teamlead.md`: передача tester'у пути к матрице `docs/tests/<KEY>/test-cases.md`; понижения покрытия — в задании tech-writer для `changes.md`. Проверка: тело ≤ baseline

## 6. Стадия 3: qa (приёмка)

- [ ] 6.1 Дополнить `refs/qa/modes.md` режимом «приёмка»: объём ручного прогона по фактическому статусу, источник результатов CI — `coverage-report.json` или подтверждение пользователя с пометкой, красный, пропущенный или не найденный тест отменяет облегчённую приёмку, приёмка без матрицы — полный прогон, источник результата по каждому шагу в отчёте и TestRail, запись полей `Dev coverage` / `Dev autotests` после согласия (пропуск, если полей нет; намерение не пишется). Проверка: каждое требование `coverage-aware-acceptance` отражено в файле
- [ ] 6.2 Обновить в `canon/qa.md` правило «автоматизированными тестами не занимаешься»: статус берётся из `coverage-report.json` или от пользователя. Проверка: тело ≤ baseline; генерация и смоук проходят

## 7. llm-wiki

- [ ] 7.1 Обновить `wiki/topics/testing.md`: конвенция метки TestRail (ключ, значение, несколько меток на тест, эквиваленты NUnit/MSTest), явное исключение из правила о ссылках на тикеты, ссылка на шкалу покрытия. Проверка: `scripts/lint-wiki.sh` проходит; `updated` во frontmatter обновлён
- [ ] 7.2 Обновить `wiki/topics/ai-sdlc.md` и `wiki/topics/ai-sdlc-contracts.md`: порядок стадии 1 (analyst → qa → planner), выгрузка после заведения подзадач, артефакт `docs/tests/<EPIC>/test-cases.md`, отчёт покрытия в pipeline. Проверка: `scripts/lint-wiki.sh` проходит; описание стадии 1 не противоречит `canon/analyst.md`
- [ ] 7.3 Исправить подпись паттерна BaseApiTest в `wiki/topics/testing-stack.md` (не «TestRail»): это основной уровень для покрытия API-шагов. Проверка: `scripts/lint-wiki.sh` проходит
- [ ] 7.4 Записать изменения в `wiki/log.md` по правилам `wiki/contributing.md`. Проверка: запись есть и оформлена по правилам; `scripts/wiki-size.sh` в пределах лимитов

## 8. Интеграционная проверка

- [ ] 8.1 Полная регенерация и проверки agent-pack: `pwsh scripts/generate.ps1`, `sh scripts/validate.sh`, `node test-smoke.js`, `claude plugin validate .` (без `--strict`). Проверка: всё проходит, `canon/baseline.json` не вырос без обоснования
- [ ] 8.2 Сквозной прогон-сухой на учебном эпике: analyst → qa (дизайн) → planner → выгрузка (на тестовом проекте TestRail или без записи) → tester с трейтами → матрица. Проверка: матрица валидна по схеме 2.2, у каждого кейса есть план и факт, трейты в тестах соответствуют матрице, в TestRail не тронуто поле State

## Workflow follow-up

- Поставить задачу SRE (1.2) после завершения 2.2 и отправить запрос администратору TestRail (1.1).
- Когда поля созданы, вписать их системные имена в `refs/qa/testrail-format.md`.
- Пилот процесса на одном реальном эпике, затем архивировать изменение (`/openspec-archive-change`).
