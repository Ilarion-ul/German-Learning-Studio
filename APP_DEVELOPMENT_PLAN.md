# German Learning Studio — технічне завдання і план розробки з Codex

Версія 1.0 · 30 жовтня 2026

## 1. Бачення продукту

Windows-first, offline-first персональний навчальний застосунок для 12-тижневої інтенсивної програми німецької A0→A2 (30 год/тиждень). Застосунок пов'язує **навчальний план, фактичний час, словник/SRS, граматику, вправи, перевірки й аналітику прогресу**. Пізніше — зовнішній навчальний чат/AI-викладач, аудіотренування, синхронізація і підтримка B1.

### Пріоритетні користувацькі сценарії

1. Відкрити «Сьогодні»: тема, план 300 хв, незавершені задачі й належні картки.
2. Запустити таймер певного виду активності, зупинити/продовжити, зберегти реальний час.
3. Виконати повторення карток із самоперевіркою й автоматичним плануванням наступної дати.
4. Прочитати граматичне правило, виконати вправи, отримати пояснення помилок.
5. Виконати рольову/письмову задачу, вручну або за рубрикою зареєструвати результат.
6. Переглянути статистику: години за тиждень, активність, retention, теми помилок, поступ A1/A2.
7. Експортувати машинозчитуваний звіт для ChatGPT-викладача; імпортувати структурований план/словник, який схвалив користувач.

## 2. Обраний технічний стек

- **UI:** Qt 6 Quick / QML + Qt Quick Controls, desktop (Windows 11 спочатку; Linux пізніше).
- **Core:** C++20 з чіткими інтерфейсами для SRS, grading, timers, progress.
- **DB:** SQLite (локальна база, транзакції, міграції версій, зовнішні ключі).
- **Build:** CMake + Ninja; перевірити наявну Qt-конфігурацію в першому завданні.
- **Tests:** Qt Test або Catch2 (обрати один), CTest, інтеграційні тести SQLite; GitHub Actions Windows за можливості.
- **Content:** версійовані JSON-файли або YAML для навчального курсу, граматичних правил, вправ; імпорт у SQLite.
- **Audio:** локальні файли/ресурси з перевіреною ліцензією; TTS лише як необов'язкова майбутня функція.
- **AI:** тільки опційний адаптер у подальшій фазі; не блокувати MVP API-ключами чи мережею.

## 3. Важливе рішення щодо Anki

«Вбудований Anki» може означати три різні речі:

A. **Власний SRS-модуль із картками та повторенням в інтерфейсі застосунку** — **MVP, рекомендовано**.
B. **Сумісність із Anki** через підтримувані текстові імпорт/експорт і за потреби `.apkg` — фаза 2.
C. **Робота безпосередньо з установленим Anki** (наприклад, через сторонній AnkiConnect) — експериментальна опція, вимагає Anki desktop, перевірки API, версій, прав і конфліктів даних.

Не називати власний scheduler «Anki FSRS», якщо не реалізовано й не перевірено справжній алгоритм FSRS. Для першої версії — простий, тестований алгоритм зберігання карток/оцінки та повторення; пізніше — коректна інтеграція перевіреної реалізації FSRS із відповідною ліцензією і regression tests. Не редагувати Anki collection DB безпосередньо.

## 4. MVP: обов'язкові функції

### Dashboard
- Поточний тиждень/день, 30 год на тиждень, 5 год на день, шість днів.
- Скільки реально виконано; progress by activity; due cards; weakest grammar topics.
- «Start study block» і «Continue last session».

### Time Tracking
- Start/pause/resume/finish із видом заняття: SRS, grammar, reading, listening, speaking, writing, assessment.
- Сесія переживає перезапуск; зберігати timestamps і монотонний час там, де доступно; не зараховувати час автоматично після аварійного закриття без підтвердження.
- Редагування часу вручну з позначкою `manual`; захист від подвійного зарахування паралельних таймерів.
- Підсумки по днях/тижнях, експорт CSV.

### Flashcards + SRS
- Нотатка (лексична одиниця) і окрема картка (напрям запитання) із стабільними UUID.
- Front/back, артикль, множина, переклад, приклад, тег CEFR/week/topic, аудіо URL/relative path (опційно).
- Review log: evaluation (Again/Hard/Good/Easy), reviewed_at, due_before/after, interval_before/after, algorithm_version.
- Алгоритм із детермінованими тестами; без втрати історії при імпорті.
- Пошук/фільтри, захист від дублікатів, імпорт/експорт UTF-8 TSV/CSV.

### Grammar and Exercises
- Правило: title, level, prerequisites, explanation_uk, examples_de, common_errors, references.
- Вправи: multiple_choice, gap_fill, ordering, short_answer (manual/controlled normalization); опціонально conjugation table.
- Кожна вправа має `exercise_id`, перевірювану правильну відповідь, пояснення та версію.
- Зберігати усі спроби, а не тільки підсумковий бал.
- Для відкритих відповідей не видавати просту перевірку рядків за повноцінну граматичну оцінку.

### Course and Assessment
- 12 тижнів × 6 днів, за замовчуванням 300 хв/день, перевірки на 4/7/10/11/12 тижнях.
- Зв'язок між уроками, правилами, картками, вправами та фактичними сесіями.
- Оцінювання за навичками: Reading, Listening, Writing, Speaking; офіційні іспити НЕ включати в код без ліцензій.
- Backup / restore; JSON/CSV експорт progress snapshot для викладача.

## 5. Архітектура та схеми даних

Пропонована структура репозиторію:

```text
german-learning-studio/
  AGENTS.md
  README.md
  docs/PRD.md
  docs/ROADMAP.md
  docs/ARCHITECTURE.md
  docs/CONTENT_SPEC.md
  docs/TEST_PLAN.md
  content/course_12_weeks.json
  content/grammar/*.json
  content/cards/*.json
  content/exercises/*.json
  src/core/{timer,srs,exercise,course,analytics}/
  src/data/{sqlite,migrations,repositories}/
  src/app/{main.cpp,qml/}
  tests/{unit,integration,fixtures}/
  tools/{content_validator,exporter}/
```

Мінімальні сутності SQLite:

- `course_weeks(id, sequence, target_level, objective)`
- `course_days(id, week_id, day_number, planned_minutes)`
- `study_tasks(id, day_id, category, title, planned_minutes, status)`
- `study_sessions(id, task_id_nullable, category, started_at, ended_at, active_seconds, source, status)`
- `grammar_topics(id, level, title, content_version, content_path)`
- `exercises(id, topic_id, type, prompt, answer_spec_json, version)`
- `exercise_attempts(id, exercise_id, attempted_at, raw_answer, score, feedback_json)`
- `notes(id, lemma, article, plural, translation_uk, example_de, example_uk, level, source_id)`
- `cards(id, note_id, direction, deck_id, created_at, state, due_at)`
- `reviews(id, card_id, reviewed_at, rating, due_before, due_after, algorithm_version)`
- `assessments(id, date, exam_type, skill, score, maximum, feedback)`
- `app_settings(key, value)` and `schema_migrations(version, applied_at)`.

Обов'язково: стабільні первинні ключі, foreign keys, індекси за due_at і датами, перевірка UTC/timezone, міграції без втрати записів, резервні копії перед міграціями.

## 6. Поетапна дорожня карта

| Фаза | Функції | Критерій готовності |
|---|---|---|
| 0 — Foundation | PRD, макет екранів, CMake/Qt, SQLite migrations, CI, мінімальний набір контенту | застосунок запускається; тести та збірка відтворювані |
| 1 — Timer + Dashboard | графік на 12 тижнів, сесії, пауза, журнал, CSV | немає подвійного часу; дані зберігаються після перезапуску |
| 2 — SRS MVP | картки, due queue, review log, scheduler, CSV import/export | календар повторів проходить фіксовані тести |
| 3 — Grammar Trainer | картки правил, типи вправ, grading, спроби, помилки | 5+ тем, 30+ перевірених вправ; історія помилок |
| 4 — Course Integration | план на 12 тижнів, рекомендовані задачі, тижневі тести | Day 1→Day 6 працює наскрізно, прогрес коректно обчислюється |
| 5 — Anki Compatibility | імпорт/експорт TSV/CSV, дослідження `.apkg`, опційний AnkiConnect | idempotent імпорт, дублікатів немає, записані межі сумісності |
| 6 — Analytics & Export | дашборди, слабкі теми, звіт для ChatGPT | звіт не вигадує години/оцінки, повторно відтворюється з БД |
| 7 — Audio / AI / B1 | вимова, TTS, розмовний AI, контент B1, синхронізація | кожен модуль окремий opt-in з privacy/cost контролями |

Фази 0–4 — перший корисний реліз. Фаза 5 — інтеграційна, не блокує початок занять.

## 7. UX: основні екрани

1. **Today:** timeline навчального дня, Start/Pause, поточний прогрес, наступні вправи.
2. **Cards:** повторення карток, перегляд колод, редактор, статистика.
3. **Grammar:** дерево A1/A2 тем, правило → вправа → аналіз помилок.
4. **Course:** тижні, дні, контрольні точки, unlocked/completed стани.
5. **Statistics:** фактичні хвилини, баланс навичок, retention, помилки, weekly trend.
6. **Settings:** мова інтерфейсу UA, шлях до БД, резервні копії, контент, імпорт/експорт.

Робити UI придатним для масштабування та клавіатури; не блокувати навігацію під час таймера; темна і світла теми як опція.

## 8. Контент та якість вправ

Створити machine-readable schema (`course`, `grammar`, `exercise`, `note`, `dialogue`) і content validator. Усі навчальні дані мають містити `id`, `version`, `level`, `topic`, а вправи — допустимі варіанти відповіді й пояснення. Контрольні перевірки не повинні мати хибних відповідей або декількох неописаних правильних варіантів. Усі правила та приклади перевіряти лінгвістично, не генерувати тисячі карток без QA.

Контент Nicos Weg і платних підручників не копіювати до застосунку без відповідних прав. Зберігати посилання/метадані, створювати власні вправи та картки.

## 9. Тестування та нефункціональні вимоги

- Unit: відмінювання/нормалізація відповідей, SRS edge cases, дата/timezone, scheduler determinism.
- Integration: міграції, transactions, import idempotency, backup/restore, перезапуск таймера.
- UI smoke: запуск, перехід між модулями, доступні due cards, збереження сесій.
- Data quality: усі уроки посилаються на чинні правила й вправи; немає orphan references.
- Приватність: offline-first; зовнішні API тільки з явною згодою. Не надсилати повну навчальну історію без потреби.
- Надійність: атомарне збереження review/сесії; експорт на випадок переходу до іншої системи.
- Доступність: гарячі клавіші для оцінки карток, масштабування тексту, достатній контраст.

## 10. Послідовність задач для Codex

**Codex Task 01 — repository scaffold.** Прочитай `AGENTS.md`, створити CMake Qt 6 Quick app + SQLite data layer, базові тести, README з точними build/run/test командами. Не реалізовувати бізнес-функції до проходження smoke test.

**Task 02 — schemas/migrations.** Створити схему, repositories, міграції, тест backup/restore.

**Task 03 — timer.** State machine idle/running/paused/completed, persisted study sessions, edge-case tests (midnight, restart, double start).

**Task 04 — flashcards.** Notes/Cards/Reviews, due queue, простий scheduler, імпорт/експорт, тести.

**Task 05 — grammar.** JSON schema, validator, 5 стартових тем, 30 вправ, grading + explanations.

**Task 06 — plan integration.** Seed 12×6 календар, week/day objective, dashboard, completion criteria.

**Task 07 — analytics/export.** Aggregate time, review counts and per-skill scores; експорт JSON `LEARNING_STATE`.

**Task 08 — Anki interoperability spike.** Спочатку дослідити API/формати й протестувати на копії профілю, потім реалізувати reversible adapter. Не чіпати реальні колекції користувача без резервної копії.

Кожну задачу Codex має завершувати: зміни файлів, команди збірки/тестів, їхній фактичний результат, ризики, наступний маленький крок. Не заявляти «тести пройдені», якщо їх не запускали.

## 11. Що НЕ входить до MVP

- Повна реалізація інтерфейсу Anki чи читання його внутрішньої БД.
- Обов'язкова хмарна синхронізація, облікові записи та сервер.
- Автоматичне надійне оцінювання довільного усного мовлення без окремої speech-системи.
- Ліцензійно закриті матеріали Goethe, ÖSD, DW.
- Перевірка офіційної сертифікації всередині застосунку.

## 12. Джерела документації

- Qt Quick: https://doc.qt.io/qt-6/qtquick-index.html
- Anki manual: https://docs.ankiweb.net/
- Anki text import: https://docs.ankiweb.net/importing/text-files.html
- Anki packaged decks: https://docs.ankiweb.net/importing/packaged-decks.html
- Codex AGENTS.md: https://learn.chatgpt.com/docs/agent-configuration/agents-md
