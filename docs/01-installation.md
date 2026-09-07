# Установка и совместимость версий

## Исходное состояние

Окружение: Node.js 22, npm 9, pnpm 11. Каталог харнесса `~/.dsh` частично существовал
(профиль `web` с развёрнутым деревом `node_modules` — след запуска через `npx`).

Ключ API хранится локально в `.env` рядом с установкой Python-оркестратора и
подхватывается переменной окружения:

```bash
export DEEPSEEK_API_KEY=$(grep -E "^DEEPSEEK_API_KEY=" <путь>/.env | head -1 | cut -d= -f2- | tr -d "\"'")
```

Адаптер `dsh-llm-deepseek` читает ключ именно из `DEEPSEEK_API_KEY` (резолвится на каждый
запрос через credentials seam → окружение), базовый URL — `https://api.deepseek.com`
(переопределяется `DEEPSEEK_BASE_URL`).

## Установка

```bash
npm install -g @deepseek-ai/dsh
dsh --version
```

## Ловушка версий (важно!)

`npm install -g @deepseek-ai/dsh` ставит новейшую версию (на момент опыта —
**0.1.2-rc.1**), но популярные сторонние плагины собираются под предыдущую линейку.
Симптом: `dsh web` падает на старте с

```
Error: failed to import loader entry web-ui-settings (…):
The requested module '@deepseek-ai/dsh-settings' does not provide an export
named 'settingsNamespace'
```

Причина: developer preview с ломающими изменениями между 0.1.1 и 0.1.2 — из
`dsh-settings` исчезли экспорты `settingsNamespace` / `installSettingsSection`,
на которых построены UI-плагины.

Диагностика по датам публикаций npm:

| Пакет | Версия | Дата |
|---|---|---|
| `@deepseek-ai/dsh` | 0.1.1-rc.2 | 2026-08-21 |
| `@linxin666/dsh-web-ui-all` | 0.3.6 | 2026-08-27 |
| `@deepseek-ai/dsh` | 0.1.2-rc.1 | 2026-09-03 |

Бандл 0.3.6 вышел между линейками — собран под 0.1.1. Решение:

```bash
npm install -g @deepseek-ai/dsh@0.1.1-rc.2
```

после чего Web UI поднялся без ошибок. **Перед обновлением dsh проверяйте, под какую
линейку собраны установленные плагины**, и наоборот.

## Сборки нативных модулей (pnpm 11)

При установке плагинов pnpm блокирует build-скрипты нативных зависимостей
(`node-pty`, `ssh2`, `cloudflared`, `cpu-features`) — без них не работает
терминал в Web UI. pnpm 11 **игнорирует** поле `pnpm.onlyBuiltDependencies` в
`package.json`; правильное место — `pnpm-workspace.yaml` профиля
(`~/.dsh/profiles/web/pnpm-workspace.yaml`):

```yaml
packages:
  - .

nodeLinker: hoisted
autoInstallPeers: false
allowBuilds:
  cloudflared: true
  cpu-features: true
  node-pty: true
  ssh2: true
```

После правки — `pnpm install --dir ~/.dsh/profiles/web`.

## Смоук-тест

```bash
dsh --profile headless "Reply with the single word: OK"
# → OK, exit code 0
```

## Полезные команды

| Команда | Назначение |
|---|---|
| `dsh --version` | версия CLI |
| `dsh web --no-open` | поднять Web UI без открытия браузера (порт 3080) |
| `dsh --profile web --dump-config` | скомпонованное дерево плагинов профиля |
| `dsh plugin --profile web add <пакет>` | установка плагина в профиль |
| `dsh --profile headless "<задача>"` | one-shot прогон с ответом в stdout |

Сессии-траектории (append-only JSONL, zstd) пишутся в `~/.dsh/sessions/<литерал-workdir>/`
— по одному каталогу на рабочий каталог, с cwd и полной цепочкой событий.
