# Goal Setting Skill — постановка целей для AI-агента

Навык OpenCode, который помогает формулировать (а не выполнять) цели для AI-агентов. Превращает хаотичные мысли в чёткую проверяемую цель.

## Что делает

- Собирает разрозненные идеи → структурирует
- GAP-анализ по 7 критериям SMART+STRONG
- Задаёт максимум 2 вопроса только по пропускам
- Выводит готовую команду `/goal` с флагами ИЛИ markdown-промпт
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
       "template": "Загрузи навык goal-setting и следуй его алгоритму полностью. …",
       "agent": "build"
     },
     "goalhelp": {
       "description": "Quick goal clarification — short mode.",
       "template": "Load goal-setting skill. This is SHORT mode. …",
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

### Что автоопределяется на каждом запуске

| Проверка | Если есть → | Если нет → |
|---|---|---|
| `plugin` в `opencode.jsonc` содержит `*goal*` | Выдаёт **Вариант A** (`/goal` c флагами) + предупреждение если плагин старый | Выдаёт **Вариант B** (markdown-промпт) и предлагает установить плагин |
| Плагин установлен | Команда готова к запуску | Даёт markdown, который можно вставить в промпт вручную |
| Файл `TASKS.md` | Авто-вставляет `--check "grep -q '✅' TASKS.md"` | Не добавляет проверку через файл |

### Пример полного цикла

```bash
# 1. Формулировка (через навык)
/цель «починить SkillCard билд, синхронизировать wiki»

# 2. Агент выдаёт (с автоопределением):
/goal fix SkillCard build and sync wiki \
  --check "npm run build && npm run gen:wiki-llms" \
  --constraint "only edit components/SkillCard.tsx and wiki/" \
  --non-goal "test files, other components" \
  --max-turns 15 \
  --max-minutes 45

# 3. Вы запускаете → плагин идёт сам
# 4. Если плагина нет → агент даёт markdown:
## Цель: Fix SkillCard build
### Done:
- `npm run build` exit 0
### Not touching:
- test files
...
```

### Совместимость

| Плагин | Поддерживаемые флаги | Примечание |
|---|---|---|
| `opencode-goal-plugin@0.10+` | `--check`, `--constraint`, `--non-goal`, `--max-turns` | Основной |
| `@bybrawe/opencode-goal@1.x` | `--success`, `--accept`, `--constraint`, `--non-goal`, `--check` | Richer, host-verified |

## Отличия от аналогов

| Плагин | Тип | Что делает |
|---|---|---|
| `opencode-goal-plugin` | Execution engine | Выполняет цели, проверяет completion |
| `@bybrawe/opencode-goal` | Execution engine | Те же возможности, V2-native |
| **Clear Goal** | **Skill (planning)** | **Формулирует цель, готовит контракт** |

Этот навык — не конкурент, а дополнение: он подготавливает цель, которую потом выполняет goal-плагин.

## License

MIT