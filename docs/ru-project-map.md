# Карта проекта Autoresearch

Этот файл объясняет структуру проекта по-русски, но не заменяет английские рабочие файлы. Его задача - дать карту: какие папки за что отвечают и как они работают вместе.

## Главная модель

Autoresearch - это набор инструкций, команд, хуков и shell-скриптов для автономного цикла:

```text
пользовательская цель -> выбор режима -> команда агента -> изменение -> механическая проверка -> keep/revert -> лог результата
```

В проекте есть два слоя:

| Слой | Что это | Нужно ли переводить |
|---|---|---|
| Пользовательская оболочка | Объяснения, quick start, карта команд | Да, именно для этого добавлены русские файлы |
| Рабочее ядро | Skill-файлы, command-файлы, shell-скрипты, тесты, JSON-манифесты | Нет, это протокол работы |

## Корень проекта

| Путь | Назначение |
|---|---|
| `README.md` | Основная английская витрина проекта: идея, команды, установка, FAQ |
| `README.ru.md` | Короткая русская пользовательская оболочка для запуска без чтения всего проекта |
| `AGENTS.md` | Инструкции для AI-агентов, которые работают внутри этого repo |
| `CONTRIBUTING.md` | Правила вклада в проект |
| `COMPARISON.md` | Сравнение с оригинальной идеей Karpathy autoresearch |
| `CONTEXT.md` | Дополнительный контекст проекта |
| `LICENSE` | MIT-лицензия |
| `context7.json` | Метаданные для Context7 |

## Папка `plugins/autoresearch`

Это Codex-распространение плагина.

Главные элементы:

- `plugins/autoresearch/.codex-plugin/plugin.json` - метаданные плагина для Codex;
- `plugins/autoresearch/skills/autoresearch/SKILL.md` - тонкий маршрутизатор;
- `plugins/autoresearch/skills/autoresearch/*.md` - отдельные инструкции для команд;
- `plugins/autoresearch/skills/autoresearch/references/*.md` - общие справочные блоки.

Когда в Codex вызывается `$autoresearch debug`, Codex должен загрузить не весь проект, а соответствующий command-файл. Это снижает расход контекста и делает поведение более предсказуемым.

## Папка `claude-plugin`

Это Claude Code-распространение плагина.

Главные элементы:

- `claude-plugin/.claude-plugin/plugin.json` - метаданные Claude-плагина;
- `claude-plugin/commands/autoresearch.md` - корневая команда;
- `claude-plugin/commands/autoresearch/*.md` - подкоманды;
- `claude-plugin/skills/autoresearch/SKILL.md` - skill-маршрутизатор;
- `claude-plugin/hooks/*.cjs` - хуки безопасности и контекста.

Логика похожа на Codex-версию, но синтаксис команд другой: Claude использует `/autoresearch:debug`, Codex использует `$autoresearch debug`.

## Папка `scripts`

Это исполняемые shell-скрипты. Их лучше не переводить.

| Файл | Роль |
|---|---|
| `scripts/install.sh` | Установка плагина |
| `scripts/transform.sh` | Генерация платформенных вариантов из общего набора файлов |
| `scripts/orchestrate.sh` | Детерминированный маршрутизатор для режима orchestrator |
| `scripts/score-regression.sh` | Подсчет verdict для regression-gate |
| `scripts/score-debug-fix.sh` | Оценка debug/fix workflow |
| `scripts/release.sh` | Сборка и выпуск релиза |
| `scripts/release.md` | Инструкции к release-процессу |

`orchestrate.sh` особенно важен: он классифицирует цель, выбирает следующий шаг, считает оставшиеся units, ловит plateau и решает, когда цикл считается завершенным или заблокированным.

## Папка `tests`

Это самопроверка проекта.

| Путь | Что проверяет |
|---|---|
| `tests/test-orchestrator.sh` | Классификацию целей, маршрутизацию, plateau, safety-screening |
| `tests/test-regression.sh` | Regression verdict: `STABLE`, `UNSTABLE`, `BASELINE_UNAVAILABLE` |
| `tests/test-hooks.sh` | Поведение Claude hooks |
| `tests/fixtures/**` | Входные данные для тестов |

Фикстуры и ожидаемые строки намеренно английские. Их перевод изменит смысл тестов.

## Папка `guide`

Это подробная английская документация по сценариям:

- `guide/getting-started.md` - старт;
- `guide/autoresearch.md` - основной цикл;
- `guide/autoresearch-plan.md` - подбор метрики и verify-команды;
- `guide/autoresearch-debug.md` - поиск багов;
- `guide/autoresearch-fix.md` - ремонт ошибок;
- `guide/autoresearch-security.md` - security-аудит;
- `guide/autoresearch-ship.md` - shipping workflow;
- `guide/autoresearch-scenario.md` - сценарии и edge cases;
- `guide/autoresearch-evals.md` - анализ результатов;
- `guide/scenario/*.md` - примеры сценариев по доменам.

Для твоего использования читать всю папку не нужно. Если непонятно, какую команду выбрать, смотри сначала `README.ru.md`, затем конкретный guide-файл только по нужной команде.

## Папка `docs`

Это архитектурные и проектные документы:

- `docs/system-architecture.md` - как устроены слои, команды, hooks, routing;
- `docs/codebase-summary.md` - краткое описание codebase;
- `docs/code-standards.md` - стандарты разработки;
- `docs/development-roadmap.md` - roadmap;
- `docs/changelog.md` и `docs/project-changelog.md` - история изменений;
- `docs/project-overview-pdr.md` - продуктово-технический обзор.

`docs/ru-project-map.md` - русская навигационная карта поверх них.

## Как части работают вместе

1. Пользователь вызывает команду в Claude, OpenCode или Codex.
2. Платформа загружает соответствующий plugin/skill/command файл.
3. Command-файл задает агенту протокол работы: что читать, что спросить, какие проверки запускать.
4. Если включен orchestrator, `scripts/orchestrate.sh` помогает выбрать следующий шаг.
5. Агент делает одно изменение, коммитит эксперимент, запускает `Verify` и `Guard`.
6. Если стало лучше, изменение остается. Если стало хуже, оно откатывается.
7. Результат пишется в TSV-лог, чтобы следующие итерации видели историю.

## Практическая граница для тебя

Если ты хочешь просто несколько раз воспользоваться проектом, тебе не нужно понимать каждую папку. Достаточно:

- знать синтаксис своей платформы: для Codex это `$autoresearch ...`;
- начинать с `plan` или `--dry-run`;
- ограничивать `Iterations`;
- задавать узкий `Scope`;
- читать TSV/summary после запуска;
- не трогать рабочее ядро ради перевода.
