# Подключение OpenCode v2 к подписке «Алиса AI Про»

[OpenCode](https://opencode.ai) подключается к подписке **Алиса AI Про** как обычный
OpenAI-совместимый провайдер. Авторизация — по IAM-токену Yandex Cloud, который выпускается
командой `yc iam create-token`.

Эта инструкция — для **OpenCode 2** (актуальная версия). Конфигурация для **OpenCode 1**
лежит в [соседней инструкции](opencode.md) и её файле [`opencode/config.json`](opencode/config.json).

Оформить подписку: <https://aistudio.yandex.ru/manage>

## Требования

- Установленный **OpenCode 2** (<https://opencode.ai/v2/docs>):

  ```bash
  # macOS / Linux
  curl -fsSL https://opencode.ai/v2/install | bash

  # либо через менеджер пакетов
  brew install anomalyco/tap/opencode-v2
  npm install -g @opencode/cli
  ```

  > Если раньше стоял OpenCode 1, удалите пакетную установку v1: установщик v2
  > заменяет бинарник `opencode`, но пакетные версии по умолчанию рядом не живут.

- Установленный [Yandex Cloud CLI](https://yandex.cloud/ru/docs/cli/) (`yc`).
- Аккаунт Yandex Cloud с активной подпиской «Алиса AI Про».
- Идентификатор каталога (`folder-id`), в котором оформлена подписка.

## 1. Опишите провайдера и модели

**Файл `~/.config/opencode/opencode.json`** (либо `opencode.json` в корне проекта).
Готовый пример лежит в репозитории: [`opencode-v2/config.json`](opencode-v2/config.json).

```json
{
  "$schema": "https://opencode.ai/config.json",
  "providers": {
    "yandex": {
      "name": "Yandex Cloud (Алиса AI Про)",
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://apps.ai.api.cloud.yandex.net/v1"
      },
      "headers": {
        "OpenAI-Project": "{env:folder_id}"
      },
      "models": {
        "DeepSeek-V4.1-Flash": {
          "modelID": "gpt://{env:folder_id}/deepseek-v4.1-flash/latest",
          "name": "DeepSeek V4.1 Flash",
          "capabilities": {
            "tools": true,
            "input": ["text", "image"],
            "output": ["text"],
            "reasoning": {
              "supported": true,
              "effort": ["low", "medium", "high"]
            }
          },
          "compatibility": {
            "reasoningField": "reasoning_content"
          },
          "limit": {
            "context": 1048576,
            "output": 32768
          },
          "variants": [
            { "id": "none", "settings": { "reasoningEffort": "none" } }
          ]
        },
        "DeepSeek-V4-Flash": {
          "modelID": "gpt://{env:folder_id}/deepseek-v4-flash/latest",
          "name": "DeepSeek V4 Flash",
          "capabilities": {
            "tools": true,
            "input": ["text"],
            "output": ["text"],
            "reasoning": {
              "supported": true,
              "effort": ["minimal", "low", "medium", "high", "xhigh"]
            }
          },
          "compatibility": {
            "reasoningField": "reasoning_content"
          },
          "limit": {
            "context": 1048576,
            "output": 32768
          },
          "variants": [
            { "id": "none", "settings": { "reasoningEffort": "none" } }
          ]
        }
      }
    }
  }
}
```

Каталог подставляется из переменной окружения `folder_id` (синтаксис `{env:имя}` —
подстановка переменных окружения OpenCode):

```bash
export folder_id="<folder-id>"
```

Если переменные окружения неудобны, замените `{env:folder_id}` в трёх местах — в
`headers."OpenAI-Project"` и в обоих `modelID` — на сам идентификатор каталога.

Что за что отвечает:

| Поле | Значение |
|---|---|
| `providers.yandex.package` | Рантайм OpenAI-совместимого API: `@opencode/ai/providers/openai-compatible`. |
| `settings.baseURL` | Адрес Yandex Cloud для подписок: `https://apps.ai.api.cloud.yandex.net/v1`. |
| `headers."OpenAI-Project"` | Каталог подписки. |
| `models.<ключ>.modelID` | URI модели в Yandex Cloud (`gpt://<folder-id>/<model>/latest`). |
| `models.<ключ>.capabilities.tools` | Модель умеет вызывать инструменты; для кодинг-агента нужно `true`. |
| `models.<ключ>.capabilities.input` | У `DeepSeek-V4.1-Flash` добавлен вход `image` — это вижен-модель. |
| `compatibility.reasoningField` | Модель отдаёт рассуждения в поле `reasoning_content`. |
| `variants` | Вариант `none` выключает рассуждения (выбирается как `#none`). |

Модели DeepSeek, доступные в подписке:

| Модель | modelID | Контекст | Вход |
|---|---|---|---|
| DeepSeek V4.1 Flash | `gpt://<folder-id>/deepseek-v4.1-flash/latest` | 1 048 576 | текст, изображения |
| DeepSeek V4 Flash | `gpt://<folder-id>/deepseek-v4-flash/latest` | 1 048 576 | текст |

> Ключ в конфиге не нужен: IAM-токен вводится при входе (шаг 2) и хранится в отдельном
> хранилище авторизации OpenCode. Тарифы (`cost`) тоже не указываем: подписка оплачивается
> фиксированно по предоплате, и оценка расходов по токенам только путала бы.

## 2. Войдите в аккаунт

Откройте в TUI команду `/connect` либо выполните в терминале:

```bash
opencode auth login
```

Выберите провайдера «Yandex Cloud (Алиса AI Про)» и вставьте IAM-токен, полученный командой
`yc iam create-token`.

> В OpenCode 1 секреты лежали в `~/.local/share/opencode/auth.json`. В v2 аккаунты
> хранятся в SQLite-базе сервиса; путь к ней печатает команда `opencode debug paths db`.
> Legacy-файл `auth.json` при переходе на v2 импортируется автоматически.

Если провайдер не появляется в списке для подключения, задайте токен через переменную
окружения сервиса и добавьте её имя в конфиг провайдера:

```bash
opencode service set env YANDEX_IAM_TOKEN $(yc iam create-token)
```

```json
{
  "providers": {
    "yandex": {
      "env": ["YANDEX_IAM_TOKEN"]
    }
  }
}
```

## 3. Запустите OpenCode

```bash
opencode
```

Провайдер появится в списке как «Yandex Cloud (Алиса AI Про)»; модели DeepSeek V4 Flash и
DeepSeek V4.1 Flash выбираются в `/models`.

## Обновление токена

IAM-токен действует не более суток, поэтому его нужно периодически обновлять. Перед новым
рабочим днём или когда запросы начали возвращать ошибку авторизации выпустите токен заново
и обновите его:

```bash
yc iam create-token
opencode auth login
```

## Что изменилось относительно OpenCode 1

OpenCode 2 читает конфиги v1 из тех же мест и нормализует их, поэтому
[`opencode/config.json`](opencode/config.json) в v2 тоже продолжит работать. Ключевые
переименования:

| OpenCode 1 | OpenCode 2 |
|---|---|
| `provider` | `providers` |
| `npm: "@ai-sdk/openai-compatible"` | `package: "@opencode/ai/providers/openai-compatible"` |
| `options` | `settings`, а заголовки — в `headers` провайдера |
| `models.<ключ>.id` | `models.<ключ>.modelID` |
| `models.<ключ>.modalities` | `capabilities.input` и `capabilities.output` |
| `models.<ключ>.interleaved.field` | `compatibility.reasoningField` |

Полная таблица соответствия — в [инструкции по API-ключу](../api/opencode-v2.md#что-изменилось-относительно-opencode-1).

## Проверка подключения

```bash
curl https://apps.ai.api.cloud.yandex.net/v1/chat/completions \
  -H "Authorization: Bearer $(yc iam create-token)" \
  -H "OpenAI-Project: <folder-id>" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt://<folder-id>/deepseek-v4.1-flash/latest","messages":[{"role":"user","content":"ping"}],"max_tokens":16}'
```

Успешный ответ — объект `chat.completion` с `choices[].message` и `usage`.

## См. также

- [Codex CLI по подписке](codex.md)
- [YCodex](ycodex.md)
- [OpenCode v1 по подписке](opencode.md)
- [OpenCode v2 по API-ключу](../api/opencode-v2.md)
- Профиль сообщества: <https://github.com/serjnarbut/yandex-aistudio-profile>
