# Clear Goal Skill — постановка целей для AI-агента

**Русский** · [English](#english)

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

## Обновление

### Вариант 1 — каталог OpenCode (обновляется сам)

Требуется OpenCode V2. Один раз добавьте источник в `opencode.jsonc`:

```jsonc
{
  "skills": ["https://igorkiselev2.github.io/clear-goal-skill/catalog/"]
}
```

Дальше ничего делать не нужно: при выходе новой версии OpenCode подтянет её сам.
Если навык уже стоит локально в `~/.config/opencode/skills/clear-goal-skill/` —
удалите эту папку, иначе будут сосуществовать две копии (каталожная имеет
более высокий приоритет и победит, но путаница останется).

### Вариант 2 — одна команда вручную

Работает в любой версии OpenCode. Windows:

```powershell
$dir = "$env:USERPROFILE\.config\opencode\skills\clear-goal-skill"
New-Item -ItemType Directory -Force -Path $dir | Out-Null
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/IgorKiselev2/clear-goal-skill/master/SKILL.md" -OutFile "$dir\SKILL.md"
```

macOS / Linux:

```bash
dir=~/.config/opencode/skills/clear-goal-skill
mkdir -p "$dir"
curl -fsSL https://raw.githubusercontent.com/IgorKiselev2/clear-goal-skill/master/SKILL.md -o "$dir/SKILL.md"
```

После обновления **перезапустите OpenCode** — навыки читаются при старте.

### Вариант 3 — git clone + junction (для разработки)

Папка навыка становится ссылкой на клон, обновление сводится к `git pull`:

```powershell
git clone https://github.com/IgorKiselev2/clear-goal-skill C:\dev\clear-goal-skill
New-Item -ItemType Junction -Path "$env:USERPROFILE\.config\opencode\skills\clear-goal-skill" -Target "C:\dev\clear-goal-skill"
```

### Проверка установленной версии

Спросите у агента «какая версия навыка clear-goal-skill загружена?» — номер
указан в теле навыка. Либо посмотрите файл:

```powershell
Select-String -LiteralPath "$env:USERPROFILE\.config\opencode\skills\clear-goal-skill\SKILL.md" -Pattern "Версия навыка"
```

История изменений — в [CHANGELOG.md](./CHANGELOG.md).

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

---

<a name="english"></a>

# Clear Goal Skill — goal setting for AI agents

[Русский](#clear-goal-skill--постановка-целей-для-ai-агента) · **English**

An OpenCode skill that helps you **formulate** goals for AI agents rather than
execute them. It turns scattered thoughts into a clear, verifiable goal contract.

## What it does

- Collects scattered ideas and structures them
- Runs a GAP analysis against 7 SMART+STRONG criteria
- Splits compound goals ("do X and Y") into separate verifiable goals
- Asks questions **in waves of 2**, and only where the risk of guessing wrong
  is medium or higher
- Offers 2–3 ready answer options per question — you can reply with "1, 5"
- Marks everything it decided for you with `[от модели]`
- Outputs a ready `/goal` command with flags, or a markdown goal contract
- Pulls safety rules from your project's `AGENTS.md` automatically

What it does **not** do: execute the work, invent rules that aren't in your
project, or turn into an endless questionnaire.

## SMART+STRONG — 7 criteria

| # | Criterion | Checks |
|---|---|---|
| **S** | Specific | Concrete object, file, module? |
| **M** | Measurable | Verification command (exit 0)? |
| **A** | Achievable | Dependencies, env, realistic scope? |
| **R** | Relevant | Why does it matter? Business value? |
| **T** | Time-boxed | `--max-turns`, `--max-minutes` limits? |
| **S** | Strategy/Resources | Env, APIs, access, dependencies? |
| **T** | No-waste | What NOT to touch (files, modules, branches)? |

## Install

1. Copy the skill into your OpenCode skills directory. The folder name **must**
   match the `name` in the frontmatter:

   ```bash
   cp -r clear-goal-skill ~/.config/opencode/skills/clear-goal-skill/
   ```

2. Optionally add slash commands to `opencode.jsonc`:

   ```jsonc
   "command": {
     "цель": {
       "description": "Help me formulate a goal for an AI agent.",
       "template": "Load clear-goal-skill and follow its algorithm in full. …",
       "agent": "build"
     },
     "goalhelp": {
       "description": "Quick goal clarification — short mode.",
       "template": "Load clear-goal-skill. This is SHORT mode. …",
       "agent": "build"
     }
   }
   ```

3. Restart OpenCode.

## Commands

| Command | Language | Mode |
|---|---|---|
| `/цель` | Russian | Full, detailed interview |
| `/goalhelp` | Project language | Short, fast, 1–2 questions |

The skill also activates automatically when a request lacks a concrete object
or a definition of done.

## How it pairs with a goal plugin

This skill is a **planner**, not an executor:

```
/цель [fragments] → ready /goal command → goal plugin runs it to completion
```

| Plugin | Version (2026-10-04) | Note |
|---|---|---|
| `opencode-goal-plugin` | 0.11.0 | Main |
| `@bybrawe/opencode-goal` | 1.3.46 | Richer, host-verified |

OpenCode plugins are **not** installed with `npm install` — add the package with
a pinned version to the `plugin` array in `opencode.json` and OpenCode handles
the rest:

```jsonc
"plugin": ["opencode-goal-plugin@0.11.0"]
```

If no plugin is detected, the skill outputs a markdown goal contract instead,
which works anywhere.

## Project safety policies

The skill has no hardcoded policies. It reads your project's `AGENTS.md` and
moves those rules into the goal's `constraints` and `non-goals`. The list inside
`SKILL.md` is an **example** — replace it with your own.

## Versioning

See [CHANGELOG.md](./CHANGELOG.md). Current version: **0.2.0**.

## License

MIT
