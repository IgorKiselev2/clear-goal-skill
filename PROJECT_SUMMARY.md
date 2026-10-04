# PROJECT_SUMMARY.md

## Цель
Создать и опубликовать глобальный навык OpenCode для формулирования целей AI-агентов с проверкой по SMART+STRONG.

## Результат
- Навык создан: `~/.config/opencode/skills/clear-goal-skill/SKILL.md`
- Команды добавлены в конфиг: `/цель` (RU), `/goalhelp` (EN/short)
- Репозиторий создан: `IgorKiselev2/clear-goal-skill`
- Все обязательные файлы проекта готовы
- Первый коммит запушен

## Что пробовали
1. **Копирование файлов через Copy-Item** — ПРОВАЛ (кириллица в пути ломается)
   - Решение: писать файлы напрямую через write tool в целевую папку
2. **Git SSH с кириллическим путём** — ПРОВАЛ
   - Решение: использовать Short path (`960A~1`) и переменную `GIT_SSH_COMMAND`
3. **Host key verification** — ПРОВАЛ (старый KEX метод)
   - Решение: добавить `KexAlgorithms -sntrup761x25519-sha512@openssh.com` в ssh config

## Ключевые находки
- Deploy key работает только если Allow write access включён
- Git на Windows с SSH требует отдельного config для обхода KEX-ограничений
- Кириллица в путях PowerShell → только через короткую нотацию или Write-Tool

## Рекомендации
- Для человека: при работе с SSH на Windows использовать `GIT_SSH_COMMAND` вместо изменения global config
- Для ИИ: не использовать Copy-Item с кириллическими путями, писать файлы напрямую

## Файлы проекта
- `SKILL.md` — основной навык
- `README.md` — документация
- `AGENTS.md` — правила проекта
- `TASKS.md` — трекер задач
- `LICENSE` — MIT
- `temp/` — промежуточные файлы