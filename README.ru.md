# Autoresearch: русская пользовательская оболочка

Этот файл не переводит рабочее ядро проекта. Он объясняет, как пользоваться Autoresearch несколько раз без необходимости читать все английские инструкции, скрипты и тестовые контракты.

Autoresearch превращает Claude Code, OpenCode или Codex в цикл автономного улучшения:

```text
цель -> измеримая проверка -> одно изменение -> проверка -> оставить или откатить -> повторить
```

Главная идея: агент не просто "думает над задачей", а делает маленькие изменения, механически проверяет результат и использует git как память экспериментов.

## Что тебе реально нужно понимать

Для нескольких запусков проекта достаточно понимать пять вещей:

1. **Goal** - что ты хочешь получить обычным языком.
2. **Scope** - какие файлы агенту разрешено менять.
3. **Metric / Verify** - какая команда доказывает, что стало лучше.
4. **Guard** - какая команда не дает сломать остальное.
5. **Iterations** - сколько попыток агенту дать за один запуск.

Если метрики пока нет, начинай с `plan`: он должен помочь превратить цель в `Scope`, `Metric`, `Verify` и `Guard`.

## Как запускать в Codex

В Codex команды вызываются через `$autoresearch`:

```text
$autoresearch plan
Goal: Make API responses faster
```

Для обычного автономного цикла:

```text
$autoresearch
Iterations: 5
Goal: Increase test coverage for auth code
Scope: src/auth/**/*.ts, src/auth/**/*.test.ts
Metric: coverage % (higher is better)
Verify: npm test -- --coverage
Guard: npm test
```

Для свободной цели без ручной метрики:

```text
$autoresearch help me fix the login bug --dry-run
```

`--dry-run` полезен для первого запуска: агент покажет, как он понял цель и какой pipeline собирается применить, но не начнет менять проект.

## Какая команда для чего

| Команда | Когда использовать | Что важно понимать |
|---|---|---|
| `$autoresearch plan` | Цель есть, метрики нет | Помогает сформулировать рабочую конфигурацию |
| `$autoresearch` | Есть метрика и команда проверки | Основной цикл улучшения |
| `$autoresearch debug` | Нужно найти причину бага | Идет через гипотезы и проверки |
| `$autoresearch fix` | Тесты, типы, lint или build уже падают | Чинит ошибки по одной до нуля или лимита |
| `$autoresearch security` | Нужен аудит безопасности | По умолчанию аудит; автоисправления только если явно просить |
| `$autoresearch scenario` | Нужны edge cases или тест-сценарии | Генерирует сценарии, не обязательно меняет код |
| `$autoresearch predict` | Нужно мнение нескольких экспертных ролей перед действием | Хорошо для дорогих или сомнительных изменений |
| `$autoresearch reason` | Нужно обдумать архитектурное или продуктово-техническое решение | Спор и сходимость через "судей" |
| `$autoresearch learn` | Нужно обновить или создать документацию | Читает код и генерирует docs/wiki |
| `$autoresearch evals` | Нужно понять, как прошел прошлый цикл | Анализирует TSV-логи итераций |
| `$autoresearch regression` | Нужно проверить, не стало ли хуже | Сравнивает baseline и candidate |
| `$autoresearch ship` | Нужно провести релиз/PR/deploy | Не используй без явного намерения ship |

Для твоего режима "воспользоваться несколько раз" обычно достаточно `plan`, `$autoresearch` с малым `Iterations`, `debug`, `fix`, `security --diff`, `scenario` и `evals`.

## Безопасный первый запуск

1. Проверь, что проект под git и что ты понимаешь текущий `git status`.
2. Начни с `Iterations: 3` или `Iterations: 5`, а не с большого числа.
3. Всегда задавай узкий `Scope`, если разрешаешь изменения.
4. Добавляй `Guard`, если метрика не покрывает безопасность всего проекта.
5. Для свободной цели сначала используй `--dry-run`.
6. Перед `ship`, `deploy`, `push` и публикацией требуй отдельное явное подтверждение.

Пример аккуратного запроса:

```text
$autoresearch
Iterations: 5
Goal: Reduce failing type errors in the billing module
Scope: src/billing/**/*.ts
Metric: number of TypeScript errors (lower is better)
Verify: npm run typecheck
Guard: npm test
```

## Что не надо переводить

Рабочее ядро проекта оставлено на английском осознанно. Не стоит переводить:

- имена команд: `$autoresearch`, `debug`, `fix`, `ship`;
- флаги: `--dry-run`, `--chain`, `--diff`, `--fail-on`;
- статусы: `STABLE`, `UNSTABLE`, `CONVERGED`, `BLOCKED`, `PLATEAU`;
- JSON-ключи, TSV-заголовки, shell-команды;
- тестовые фикстуры в `tests/fixtures`;
- внутренние command-файлы в `claude-plugin/commands` и `plugins/autoresearch/skills`.

Эти слова не просто текст. Они являются протоколом между агентом, shell-скриптами, тестами и плагинами.

## Где читать дальше

- [docs/ru-project-map.md](docs/ru-project-map.md) - русская карта структуры проекта.
- [README.md](README.md) - полный английский README.
- [guide/getting-started.md](guide/getting-started.md) - подробный английский quick start.
- [docs/system-architecture.md](docs/system-architecture.md) - английская архитектура для глубокой проверки.
