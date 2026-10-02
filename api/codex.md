# Подключение Codex CLI по API-ключу

> ## ⚠️ Это очень дорого, и так делать не надо
>
> Оплата идёт за каждый токен по тарифам Yandex AI Studio. Для постоянной работы
> используйте [подписку «Алиса AI Про»](../subscription/codex.md).

Codex CLI подключается к Yandex AI Studio через отдельный профиль, но вместо IAM-токена
использует статический **API-ключ** сервисного аккаунта. Ключ и каталог передаются
через переменные окружения.

Готовые файлы лежат в репозитории:

- [`codex/yandex-ai-studio.config.toml`](codex/yandex-ai-studio.config.toml) — профиль;
- [`codex/yandex-ai-studio-catalog.json`](codex/yandex-ai-studio-catalog.json) — описание моделей.

## Требования

- Установленный Codex CLI (`npm install -g @openai/codex` или `brew install --cask codex`).
- Сервисный аккаунт с ролью `ai.languageModels.user` и созданный для него API-ключ.
- Идентификатор каталога (`folder-id`).

## 1. Задайте переменные окружения

Секретов в конфиге нет: Codex берёт ключ из переменной `api_key`, а каталог — из
переменной `folder_id` (она подставляется в заголовки `x-folder-id` и `OpenAI-Project`).

```bash
export api_key="<API-ключ>"
export folder_id="<folder-id>"
```

| Переменная | Где используется | Значение |
|---|---|---|
| `api_key` | `[model_providers.yc].env_key` | Секрет API-ключа сервисного аккаунта |
| `folder_id` | `[model_providers.yc].env_http_headers` | Идентификатор каталога Yandex Cloud |

## 2. Скопируйте профиль и каталог

```bash
mkdir -p ~/.codex
cp codex/yandex-ai-studio.config.toml ~/.codex/yandex-ai-studio.config.toml
cp codex/yandex-ai-studio-catalog.json ~/.codex/yandex-ai-studio-catalog.json
```

Полное содержимое профиля:

```toml
model = "gpt://<folder-id>/deepseek-v4.1-flash/latest"
model_provider = "yc"
model_catalog_json = "~/.codex/yandex-ai-studio-catalog.json"

# Reasoning для DeepSeek — обязательно должно быть включено!
model_supports_reasoning_summaries = true
model_reasoning_effort = "medium"
model_reasoning_summary = "auto"

web_search = "live"

[tools.web_search]
context_size = "medium"
# allowed_domains = ["aistudio.yandex.ru", "https://github.com/yandex-cloud/mcp", "https://yandex.cloud/ru/docs"]

[model_providers.yc]
name = "Yandex Cloud"
base_url = "https://ai.api.cloud.yandex.net/v1"
wire_api = "responses"
env_key = "api_key"
env_key_instructions = """
Get your API key from https://aistudio.yandex.ru/
"""
request_max_retries = 8
stream_max_retries = 20
stream_idle_timeout_ms = 900000
requires_openai_auth = false
supports_websockets = false
env_http_headers = { "x-folder-id" = "folder_id", "OpenAI-Project" = "folder_id" }

[tui]
status_line = ["model-name", "context-used"]
status_line_use_colors = true
screen_reader_detection_done = true
```

В файле `~/.codex/yandex-ai-studio.config.toml` замените `<folder-id>` в строке `model`
на идентификатор вашего каталога:

```toml
model = "gpt://<folder-id>/deepseek-v4.1-flash/latest"
```

Остальные настройки провайдера — в том же файле:

| Параметр | Значение | Описание |
|---|---|---|
| `name` | `"Yandex Cloud"` | Отображаемое имя |
| `base_url` | `"https://ai.api.cloud.yandex.net/v1"` | Базовый URL Responses API для API-ключей |
| `wire_api` | `"responses"` | DeepSeek работает через Responses API |
| `env_key` | `"api_key"` | Переменная окружения с API-ключом |
| `requires_openai_auth` | `false` | Yandex Cloud не использует OpenAI-логин |
| `supports_websockets` | `false` | WebSocket не поддерживается |
| `env_http_headers` | `{ "x-folder-id" = "folder_id", "OpenAI-Project" = "folder_id" }` | Каталог из переменной окружения |

## 3. Опишите доступные модели

Без каталога `~/.codex/yandex-ai-studio-catalog.json` Codex не знает параметры моделей:
контекстное окно, поддержку инструментов, тип shell и так далее. В нём описаны
`deepseek-v4-flash` и `deepseek-v4.1-flash`; замените `<folder-id>` в полях `slug` на
идентификатор вашего каталога.

Полное содержимое каталога:

```json
{
  "models": [
    {
      "slug": "gpt://<folder-id>/deepseek-v4-flash/latest",
      "display_name": "DeepSeek V4 Flash",
      "shell_type": "default",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 30,
      "base_instructions": "You are Codex, a coding agent.",
      "supports_reasoning_summaries": true,
      "default_reasoning_summary": "auto",
      "support_verbosity": false,
      "supported_reasoning_levels": [],
      "apply_patch_tool_type": "freeform",
      "web_search_tool_type": "text",
      "truncation_policy": { "mode": "bytes", "limit": 10000 },
      "supports_parallel_tool_calls": true,
      "experimental_supported_tools": [],
      "context_window": 1048576,
      "max_context_window": 1048576,
      "effective_context_window_percent": 90,
      "input_modalities": ["text"],
      "supports_search_tool": false,
      "use_responses_lite": false,
      "tool_mode": null,
      "multi_agent_version": null
    },
    {
      "slug": "gpt://<folder-id>/deepseek-v4.1-flash/latest",
      "display_name": "DeepSeek V4.1 Flash",
      "shell_type": "default",
      "visibility": "list",
      "supported_in_api": true,
      "priority": 30,
      "base_instructions": "You are Codex, a coding agent.",
      "supports_reasoning_summaries": true,
      "default_reasoning_summary": "auto",
      "support_verbosity": false,
      "supported_reasoning_levels": [],
      "apply_patch_tool_type": "freeform",
      "web_search_tool_type": "text",
      "truncation_policy": { "mode": "bytes", "limit": 10000 },
      "supports_parallel_tool_calls": true,
      "experimental_supported_tools": [],
      "context_window": 1048576,
      "max_context_window": 1048576,
      "effective_context_window_percent": 90,
      "input_modalities": ["text"],
      "supports_search_tool": false,
      "use_responses_lite": false,
      "tool_mode": null,
      "multi_agent_version": null
    }
  ]
}
```

### Reasoning (обязательно для DeepSeek)

DeepSeek — reasoning-модель. Если не включить параметры ниже, она не будет выдавать
цепочку рассуждений.

| Параметр | Обязательный | Описание |
|---|---|---|
| `model_supports_reasoning_summaries` | **да** | Включает блок `reasoning` в запросе |
| `model_reasoning_effort` | **да** | Уровень: `"medium"` или `"high"`. **`"none"` отключает reasoning** |
| `model_reasoning_summary` | нет | `"auto"` |

## 4. Запустите Codex

```bash
codex --profile yandex-ai-studio
```

## См. также

- [OpenCode по API-ключу](opencode.md)
- [Общее предупреждение и настройка API-ключа](README.md)
- [Codex CLI по подписке](../subscription/codex.md)
