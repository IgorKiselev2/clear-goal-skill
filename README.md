# Goal Setting Skill — постановка целей для AI-агента

Навык OpenCode, который помогает формулировать (а не выполнять) цели для AI-агентов. Превращает хаотичные мысли в чёткую проверяемую цель.

## Что делает

- Собирает разрозненные идеи → структурирует
- GAP-анализ по 7 критериям SMART+STRONG
- Расщепляет составные цели («сделай X и Y») на отдельные проверяемые цели
- Задаёт вопросы волнами по 2 и только там, где риск ошибки средний или выше
- К каждому вопросу подкидывает 2–3 готовых варианта по цифрам — можно ответить «1, 5»
- Выводит готовую команду `/goal` с флагами ИЛИ markdown-контракт
- Автоматически подтягивает safety-правила из AGENTS.md проекта

## SMART+STRONG — 7 критериев

| # | Criterion | Checks |
|---|---|---|
| **S** | Specific | Concrete object, file, module? |
| **M** | Measurable | Verification command (exit 0)? |
| **A** | Achievable | Dependencies, env, realistic scope? |
| **R** | Relevant | Why does it matter? Business value? |
| **T** | Time-boxed | `--max-turns`, `--max-minutes` limits? |
| **S** | Strategy/Resources | Env, APIs, access, dependencies? |
| **T** | No-waste | What NOT to touch (files, modules, branches)? |

## Установка

1. Скопируйте навык:
   ```bash
   cp -r clear-goal-skill ~/.config/opencode/skills/clear-goal-skill/
   ```
2. Добавьте команды в `opencode.jsonc`:
   ```jsonc
   "command": {
     "цель": {
       "description": "Помоги сформулировать цель для AI-агента.",
       "template": "Загрузи навык clear-goal-skill и следуй его алгоритму полностью. …",
       "agent": "build"
     },
     "goalhelp": {
       "description": "Quick goal clarification — short mode.",
       "template": "Load clear-goal-skill. This is SHORT mode. …",
       "agent": "build"
     }
   }
   ```
3. Перезапустите OpenCode.

## Команды

| Команда | Язык | Режим |
|---|---|---|
| `/цель` | Русский | Развёрнутый, полное интервью |
| `/goalhelp` | Язык проекта | Короткий, 1–2 вопроса, быстро |

## Pairing: как работает с goal-плагином

Этот навык — **планёр**, а не исполнитель. Идеальный flow:

```
/цель [фрагменты] 
    ↓
Агент даёт готовую команду /goal c флагами
    ↓
Вы запускаете её → goal-плагин берёт и доводит до конца
```

### Что автоопределяется

Проверка выполняется **один раз за сессию** и кэшируется.

| Проверка | Если есть → | Если нет → |
|---|---|---|
| `goal` в секции `plugin` конфига (проектный и глобальный, `.json` и `.jsonc`) | **Вариант A** — команда `/goal` с флагами | **Вариант B** — markdown-контракт |
| Конфиг недоступен для чтения | Один вопрос «плагин установлен?», ответ кэшируется на сессию | По умолчанию Вариант B |
| Файл `TASKS.md` | Авто-вставляет `--check "grep -q '✅' TASKS.md"` | Не добавляет проверку через файл |

### Пример полного цикла

```bash
# 1. Формулировка (через навык)
/цель «починить SkillCard билд, синхронизировать wiki»
```

```
# 2. Навык видит две независимые цели и предлагает порядок (Шаг 2.5):
Похоже, это две цели: 1) билд SkillCard, 2) синхронизация wiki.
  1. Последовательно, сначала билд   ← по умолчанию
  2. Только билд
  3. Одной целью
  4. свой вариант
```

```bash
# 3. После выбора «1» выдаётся ПЕРВАЯ цель — со своей проверкой:
/goal fix render error in components/SkillCard.tsx \
  --check "npm run build" \
  --constraint "only edit components/SkillCard.tsx" \
  --non-goal "test files, other components, wiki/" \
  --max-turns 10 \
  --max-minutes 30

# 4. Вы запускаете → плагин идёт сам
# 5. После её завершения — вторая цель:
/goal sync wiki --check "npm run gen:wiki-llms" --non-goal "components/" --max-turns 8
```

Цели не склеиваются в `--check "a && b"`: иначе падение второй половины
неотличимо от несработавшей первой, и агент не знает, что переделывать.

### Совместимость

| Плагин | Поддерживаемые флаги | Примечание |
|---|---|---|
| `opencode-goal-plugin` (акт. 0.11.0) | `--check`, `--constraint`, `--non-goal`, `--max-turns` | Основной |
| `@bybrawe/opencode-goal` (акт. 1.3.46) | `--success`, `--accept`, `--constraint`, `--non-goal`, `--check` | Richer, host-verified |

## Отличия от аналогов

| Плагин | Тип | Что делает |
|---|---|---|
| `opencode-goal-plugin` | Execution engine | Выполняет цели, проверяет completion |
| `@bybrawe/opencode-goal` | Execution engine | Те же возможности, V2-native |
| **Clear Goal** | **Skill (planning)** | **Формулирует цель, готовит контракт** |

Этот навык — не конкурент, а дополнение: он подготавливает цель, которую потом выполняет goal-плагин.

> Версии указаны по состоянию на 2026-10-04, проверьте актуальность перед установкой.
> Плагины OpenCode не ставятся через `npm install` — добавьте пакет с закреплённой
> версией в массив `plugin` файла `opencode.json`, остальное OpenCode сделает сам.

## License

MIT