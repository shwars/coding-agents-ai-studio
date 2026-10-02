# Подключение OpenCode по API-ключу

> ## ⚠️ Это очень дорого, и так делать не надо
>
> Оплата идёт за каждый токен по тарифам Yandex AI Studio. Одна активная сессия
> кодинг-агента может прожечь заметную сумму. Для постоянной работы используйте
> [подписку «Алиса AI Про»](../subscription/opencode.md).

OpenCode подключается к Yandex AI Studio как OpenAI-совместимый провайдер
(`@ai-sdk/openai-compatible`), но вместо IAM-токена используется статический
**API-ключ** сервисного аккаунта.

Готовый конфиг лежит в репозитории: [`opencode/config.json`](opencode/config.json).

## Требования

- Установленный OpenCode (<https://opencode.ai/docs>).
- Сервисный аккаунт с ролью `ai.languageModels.user` и созданный для него API-ключ.
- Идентификатор каталога (`folder-id`).

## 1. Задайте переменные окружения

Конфиг из этого репозитория не содержит секретов: и ключ, и каталог он берёт из
переменных окружения. Поэтому **обязательно** задайте обе переменные — без них OpenCode
не сможет подставить значения в `apiKey` и в URI моделей.

```bash
export folder_id="<folder-id>"
export api_key="<API-ключ>"
```

| Переменная | Где используется | Значение |
|---|---|---|
| `folder_id` | в `models.*.id` — URI `gpt://{env:folder_id}/...` | Идентификатор каталога Yandex Cloud |
| `api_key` | в `options.apiKey` | Секрет API-ключа сервисного аккаунта |

Синтаксис `{env:имя}` — это подстановка переменных окружения OpenCode; сам секрет в
файле конфигурации не хранится.

## 2. Скопируйте конфиг

Возьмите готовый файл [`opencode/config.json`](opencode/config.json) и положите его как
`~/.config/opencode/opencode.json` (либо `opencode.json` в корне проекта):

```bash
mkdir -p ~/.config/opencode
cp opencode/config.json ~/.config/opencode/opencode.json
```

Тот же конфиг целиком:

```json
{
  "provider": {
    "yandex": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "https://ai.api.cloud.yandex.net/v1",
        "apiKey": "{env:api_key}"
      },
      "models": {
        "DeepSeek-V4.1-Flash": {
          "id": "gpt://{env:folder_id}/deepseek-v4.1-flash",
          "name": "DeepSeek-V4.1-Flash",
          "reasoning": true,
          "tool_call": true,
          "interleaved": { "field": "reasoning_content" },
          "limit": { "context": 1048576, "output": 32678 },
          "cost": {
            "input": 0.000002459016,
            "output": 0.00000409836,
            "cache_read": 0.000000614754,
            "cache_write": 0.000000614754
          }
        },
        "DeepSeek-V4-Flash": {
          "id": "gpt://{env:folder_id}/deepseek-v4-flash",
          "name": "DeepSeek-V4-Flash",
          "reasoning": true,
          "tool_call": true,
          "interleaved": { "field": "reasoning_content" },
          "limit": { "context": 1048576, "output": 32678 },
          "cost": {
            "input": 0.000002459016,
            "output": 0.00000409836,
            "cache_read": 0.000000614754,
            "cache_write": 0.000000614754
          }
        },
        "Qwen3": {
          "id": "gpt://{env:folder_id}/qwen3-235b-a22b-fp8",
          "name": "Yandex Qwen3",
          "reasoning": true,
          "tool_call": true,
          "limit": { "context": 262144, "output": 32678 },
          "cost": {
            "input": 0.00000409836,
            "output": 0.00000409836,
            "cache_read": 0.00000409836,
            "cache_write": 0.00000409836
          }
        },
        "Qwen36": {
          "id": "gpt://{env:folder_id}/qwen3.6-35b-a3b",
          "name": "Yandex Qwen3.6",
          "reasoning": true,
          "tool_call": true,
          "modalities": {
            "input": ["text", "audio", "image"],
            "output": ["text"]
          },
          "limit": { "context": 262144, "output": 32678 },
          "cost": {
            "input": 0.000001639344,
            "output": 0.000002459016,
            "cache_read": 0.000000409836,
            "cache_write": 0.000000409836
          }
        }
      }
    }
  },
  "$schema": "https://opencode.ai/config.json"
}
```

В конфиге описаны четыре модели:

| Модель | id | Контекст | Вход |
|---|---|---|---|
| DeepSeek V4.1 Flash | `gpt://{env:folder_id}/deepseek-v4.1-flash` | 1 048 576 | текст |
| DeepSeek V4 Flash | `gpt://{env:folder_id}/deepseek-v4-flash` | 1 048 576 | текст |
| Yandex Qwen3 | `gpt://{env:folder_id}/qwen3-235b-a22b-fp8` | 262 144 | текст |
| Yandex Qwen3.6 | `gpt://{env:folder_id}/qwen3.6-35b-a3b` | 262 144 | текст, аудио, изображения |

Что за что отвечает:

| Поле | Значение |
|---|---|
| `options.baseURL` | Адрес Yandex AI Studio для API-ключей: `https://ai.api.cloud.yandex.net/v1`. |
| `options.apiKey` | API-ключ из переменной окружения `api_key`. |
| `models.<ключ>.id` | URI модели (`gpt://<folder-id>/<model>`), каталог берётся из `folder_id`. |
| `models.<ключ>.interleaved.field` | Модель отдаёт рассуждения в поле `reasoning_content`. |
| `models.<ключ>.modalities` | У `Qwen36` включён вход `audio` и `image`. |
| `models.<ключ>.cost` | Справочные тарифы за токен, чтобы OpenCode показывал оценку расходов. |

## 3. Запустите OpenCode

```bash
opencode
```

Провайдер `yandex` появится в списке, модели выбираются в интерфейсе.

## Проверка подключения

```bash
curl https://ai.api.cloud.yandex.net/v1/chat/completions \
  -H "Authorization: Bearer $api_key" \
  -H "OpenAI-Project: $folder_id" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt://'$folder_id'/deepseek-v4.1-flash","messages":[{"role":"user","content":"ping"}],"max_tokens":16}'
```

## См. также

- [Codex CLI по API-ключу](codex.md)
- [Общее предупреждение и настройка API-ключа](README.md)
- [OpenCode по подписке](../subscription/opencode.md)
