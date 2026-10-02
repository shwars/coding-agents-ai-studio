# Подключение Codex CLI к подписке «Алиса AI Про»

Стандартный Codex CLI подключается к подписке **Алиса AI Про** через отдельный профиль конфигурации. Авторизация — по IAM-токену Yandex Cloud, который выпускается командой
`yc iam create-token`.

Оформить подписку: <https://aistudio.yandex.ru/manage>

## Требования

- Установленный Codex CLI (`npm install -g @openai/codex` или `brew install --cask codex`).
- Установленный [Yandex Cloud CLI](https://yandex.cloud/ru/docs/cli/) (`yc`).
- Аккаунт Yandex Cloud с активной подпиской «Алиса AI Про».
- Идентификатор каталога (`folder-id`), в котором оформлена подписка.

## 1. Выпустите IAM-токен

```bash
yc iam create-token
```

Команда вернёт IAM-токен — короткоживущий ключ доступа. Он действует **не более суток**,
после чего запросы к сервису перестают проходить. По истечении срока выпустите новый
токен и снова подставьте его на шаге 4.

## 2. Создайте профиль

Профиль — это TOML-файл в каталоге `~/.codex`. Флаг `--profile <имя>` подключает файл
`~/.codex/<имя>.config.toml`.

**Файл `~/.codex/alice-ai-pro.config.toml`.** Готовый пример:
[`codex/alice-ai-pro.config.toml`](codex/alice-ai-pro.config.toml).

```toml
model = "gpt://<folder-id>/deepseek-v4.1-flash/latest"
model_provider = "yc"
model_catalog_json = "~/.codex/alice-ai-pro.models_catalog.json"

# Reasoning включён — модель отдаёт цепочку рассуждений
model_supports_reasoning_summaries = true
model_reasoning_effort = "medium"
model_reasoning_summary = "auto"

[model_providers.yc]
name = "Yandex Cloud"
base_url = "https://apps.ai.api.cloud.yandex.net/v1"
wire_api = "responses"
env_key = "YC_IAM_TOKEN"
requires_openai_auth = false
supports_websockets = false
request_max_retries = 8
stream_max_retries = 20
stream_idle_timeout_ms = 900000
```

Замените `<folder-id>` на идентификатор вашего каталога в Yandex Cloud. Чтобы работать
с моделью `deepseek-v4-flash`, укажите в строке `model` значение
`gpt://<folder-id>/deepseek-v4-flash/latest`.

## 3. Опишите доступные модели

Codex не знает параметры моделей Yandex Cloud, пока вы не опишете их в каталоге:
контекстное окно, поддержку инструментов, тип shell и так далее.

**Файл `~/.codex/alice-ai-pro.models_catalog.json`.** Готовый пример:
[`codex/alice-ai-pro.models_catalog.json`](codex/alice-ai-pro.models_catalog.json).

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
      "supported_reasoning_levels": [
        { "effort": "medium", "description": "Balanced reasoning depth" },
        { "effort": "high", "description": "Greater reasoning depth" }
      ],
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
      "supported_reasoning_levels": [
        { "effort": "low", "description": "Faster responses" },
        { "effort": "medium", "description": "Balanced reasoning depth" },
        { "effort": "high", "description": "Greater reasoning depth" }
      ],
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

Каталог описывает две модели: `deepseek-v4-flash` и `deepseek-v4.1-flash` — у обеих
окно контекста 1 048 576 токенов. У `deepseek-v4.1-flash` дополнительно доступен
уровень reasoning `low`.

## 4. Запустите Codex

```bash
export YC_IAM_TOKEN="$(yc iam create-token)"
codex --profile alice-ai-pro
```

## Обновление токена

IAM-токен действует не более суток, поэтому его нужно периодически обновлять. Перед
новым рабочим днём или когда запросы начали возвращать ошибку авторизации выпустите
токен заново:

```bash
export YC_IAM_TOKEN="$(yc iam create-token)"
codex --profile alice-ai-pro
```

При необходимости вынесите эти две команды в алиас или короткий скрипт.

## См. также

- [OpenCode по подписке](opencode.md)
- [YCodex](ycodex.md)
- Профиль сообщества: <https://github.com/serjnarbut/yandex-aistudio-profile>
