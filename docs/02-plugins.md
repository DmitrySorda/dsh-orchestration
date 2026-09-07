# Плагины: выборка, установка, верификация

Экосистема каталогизирована GitHub-топиком [`dsh-plugin`](https://github.com/topics/dsh-plugin)
(~14 тыс. репозиториев на момент опыта). Выборка ниже — по подборкам Composio/ZimaSpace
и звёздам топика; устанавливалось в профиль `web`.

## Что установлено и проверено

| Пакет | Версия | Назначение |
|---|---|---|
| `@linxin666/dsh-web-ui-all` | 0.3.6 | «Все inclusive»-бандл Web UI (см. ниже) |
| `@liustack/modlens` | 3.25.4 | Зрение для текстовых моделей: OCR, layout, сущности из изображений |
| `dsh-at-file` | 0.6.3 | `@`-ссылки на файлы/папки в композере Web UI |
| `dsh-find-plugin` | 0.3.7 | Поиск плагинов по топику `dsh-plugin` агентом прямо в разговоре |
| `dsh-genui` | 0.1.2 | 30+ интерактивных компонентов в ответах (графики, формы, Mermaid) |

### Бандл `dsh-web-ui-all` — что внутри

Один пакет тянет: task-board (доска задач), git-graph, **better-sidebar** (файловый
эксплорер / редактор / терминал / Git-diff рядом с чатом — workbench в стиле VS Code),
маркетплейс, менеджер плагинов, skill-explorer, центр скинов, aionui-panel,
remote-web-ui (доступ с телефона), ssh-инструменты, archive-manager, better-session
(ветвление сессий; по умолчанию выключен).

### Сознательно не установлено

- **dsh-TUI** — полноэкранный терминальный интерфейс; требует интерактивный TTY,
  в headless-оркестрации не участвует.
- **dsh-mnemon** — трёхуровневая персистентная память; требует отдельный Go-бэкенд
  (`go install github.com/mnemon-dev/mnemon@latest`).

Крупные проекты топика (OpenViking, ruflo, archify, WeKnora, EverOS, MemOS) —
самостоятельные системы, а не плагины профиля; их обзор — в README экосистемы.

## Установка

```bash
dsh plugin --profile web add @linxin666/dsh-web-ui-all@latest
dsh plugin --profile web add @liustack/modlens dsh-at-file dsh-find-plugin dsh-genui
```

`dsh plugin` пробрасывает аргументы в pnpm в каталоге профиля. Команда добавляет
пакет в `dependencies` **и** в `dsh.profile.bundles` файла
`~/.dsh/profiles/web/package.json` — обе записи обязательны для загрузки.

## Верификация

1. **Дерево конфига** — каждый бандл виден в компоновке:

   ```bash
   dsh --profile web --dump-config | grep -E "^# == "
   # ожидать слои: dsh-base → + web-ui-all → + web-app → + modlens/at-file/…
   ```

2. **Boot без ошибок**:

   ```bash
   dsh web --no-open        # затем проверить: curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:3080/
   # ожидать: HTTP 200, в логе — "dsh web: http://127.0.0.1:3080"
   ```

3. **Лог старта** — отсутствие строк `Error: failed to import loader entry …`
   (типовой симптом рассинхрона версий, см. [01-installation.md](01-installation.md)).

## Замечания совместимости

- `@liustack/modlens`: в подборках советуют пиновать версию (`@liustack/modlens@3.21.1`)
  из-за задержек свежих публикаций в pnpm 11; опыт с `latest` (3.25.4) прошёл без проблем.
- После смены версий бандла ссылки в `node_modules/@linxin666/*` могут указывать на
  старые каталоги store — лечится переустановкой зависимостей профиля.
