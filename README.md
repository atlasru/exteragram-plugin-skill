# Плагины exteraGram

Навык для ИИ-агентов по созданию, изменению и отладке плагинов exteraGram. Охватывает Python SDK, Elyx, сборку в `.pyc` и упаковку Java/Kotlin в DEX с R8.

## Codex

Вставьте в чат Codex:

> $skill-installer Установи навык из `atlasru/exteragram-plugin-skill`. Используй `--repo atlasru/exteragram-plugin-skill --path . --name exteragram-plugin-skill --method download`.

Если навык не появился в меню `$`, перезапустите Codex. Для вызова используйте `$exteragram-plugin-skill`. [Документация Codex](https://learn.chatgpt.com/docs/build-skills#install-curated-skills-for-local-use)

## Claude Code

Скачайте репозиторий через `Code → Download ZIP` и распакуйте его. Переименуйте извлечённую папку в `exteragram-plugin-skill` и поместите её в `~/.claude/skills/` для всех проектов или в `.claude/skills/` внутри нужного проекта. Файл `SKILL.md` должен лежать сразу внутри папки навыка. Для вызова введите `/exteragram-plugin-skill`. [Документация Claude Code](https://code.claude.com/docs/en/skills)

## Claude.ai

В `Customize → Skills` загрузите ZIP с папкой навыка и включите его. Если загрузка недоступна, проверьте настройки рабочего пространства. [Справка Claude](https://support.claude.com/en/articles/12512180-use-skills-in-claude)
