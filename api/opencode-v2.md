# Подключение OpenCode v2 по API-ключу

> ## ⚠️ Это очень дорого, и так делать не надо
>
> Оплата идёт за каждый токен по тарифам Yandex AI Studio. Одна активная сессия
> кодинг-агента может прожечь заметную сумму. Для постоянной работы используйте
> [подписку «Алиса AI Про»](../subscription/opencode-v2.md).

Эта инструкция — для **OpenCode 2** (актуальная версия). Конфигурация для **OpenCode 1**
лежит в [соседней инструкции](opencode.md) и её файле [`opencode/config.json`](opencode/config.json).

В v2 провайдеры описываются в объекте `providers`, а OpenAI-совместимый рантайм задаётся
полем `package`. В остальном схема прежняя: статический **API-ключ** сервисного аккаунта,
каталог в URI моделей, модели DeepSeek и Qwen.

Готовый конфиг лежит в репозитории: [`opencode-v2/config.json`](opencode-v2/config.json).

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
| `folder_id` | в `models.*.modelID` — URI `gpt://{env:folder_id}/...` | Идентификатор каталога Yandex Cloud |
| `api_key` | в `providers.yandex.settings.apiKey` | Секрет API-ключа сервисного аккаунта |

Синтаксис `{env:имя}` — это подстановка переменных окружения OpenCode; сам секрет в
файле конфигурации не хранится. Кроме `{env:...}` поддерживается подстановка из файла:
`{file:~/.secrets/yandex-key}`.

## 2. Скопируйте конфиг

Возьмите готовый файл [`opencode-v2/config.json`](opencode-v2/config.json) и положите его как
`~/.config/opencode/opencode.json` (либо `opencode.json` в корне проекта):

```bash
mkdir -p ~/.config/opencode
cp opencode-v2/config.json ~/.config/opencode/opencode.json
```

Поддерживаются и JSON, и JSONC (с комментариями). Глобальный файл действует для всех
проектов; файл в корне проекта или в `.opencode/` — только для него и имеет приоритет.

Тот же конфиг целиком:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "websearch": {
    "provider": "parallel"
  },
  "providers": {
    "yandex": {
      "name": "Yandex",
      "package": "@opencode/ai/providers/openai-compatible",
      "settings": {
        "baseURL": "https://ai.api.cloud.yandex.net/v1",
        "apiKey": "{env:api_key}"
      },
      "models": {
        "DeepSeek-V4.1-Flash": {
          "modelID": "gpt://{env:folder_id}/deepseek-v4.1-flash",
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
            "output": 32678
          },
          "cost": {
            "input": 2.459016,
            "output": 4.09836,
            "cache": {
              "read": 0.614754,
              "write": 0.614754
            }
          },
          "variants": [
            { "id": "none", "settings": { "reasoningEffort": "none" } }
          ]
        },
        "DeepSeek-V4-Flash": {
          "modelID": "gpt://{env:folder_id}/deepseek-v4-flash",
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
            "output": 32678
          },
          "cost": {
            "input": 2.459016,
            "output": 4.09836,
            "cache": {
              "read": 0.614754,
              "write": 0.614754
            }
          },
          "variants": [
            { "id": "none", "settings": { "reasoningEffort": "none" } }
          ]
        },
        "Qwen36": {
          "modelID": "gpt://{env:folder_id}/qwen3.6-35b-a3b",
          "name": "Qwen3 35B (V4)",
          "capabilities": {
            "tools": true,
            "input": ["text", "audio", "image"],
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
            "context": 262144,
            "output": 32678
          },
          "cost": {
            "input": 1.639344,
            "output": 2.459016,
            "cache": {
              "read": 0.409836,
              "write": 0.409836
            }
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

В конфиге описаны три модели:

| Модель | modelID | Контекст | Вход |
|---|---|---|---|
| DeepSeek V4.1 Flash | `gpt://{env:folder_id}/deepseek-v4.1-flash` | 1 048 576 | текст, изображения |
| DeepSeek V4 Flash | `gpt://{env:folder_id}/deepseek-v4-flash` | 1 048 576 | текст |
| Qwen3 35B (V4) | `gpt://{env:folder_id}/qwen3.6-35b-a3b` | 262 144 | текст, аудио, изображения |

Что за что отвечает:

| Поле | Значение |
|---|---|
| `providers.yandex.name` | Отображаемое имя провайдера. |
| `providers.yandex.package` | Рантайм OpenAI-совместимого API: `@opencode/ai/providers/openai-compatible`. |
| `settings.baseURL` | Адрес Yandex AI Studio для API-ключей: `https://ai.api.cloud.yandex.net/v1`. |
| `settings.apiKey` | API-ключ из переменной окружения `api_key`. |
| `models.<ключ>.modelID` | ID модели в терминах провайдера — URI `gpt://<folder-id>/<model>`. |
| `capabilities.tools` | Модель умеет вызывать инструменты (для кодинг-агента обязательно `true`). |
| `capabilities.input` / `output` | Принимаемые типы: у `Qwen36` включены `audio` и `image`, у DeepSeek V4.1 Flash — `image`. |
| `capabilities.reasoning` | Модель отдаёт рассуждения; `effort` — доступные уровни. |
| `compatibility.reasoningField` | Рассуждения приходят в поле `reasoning_content`. |
| `variants` | Именованные варианты модели. `none` выключает рассуждения (`#none` в выборе модели). |
| `cost` | Справочные тарифы **за 1 млн токенов**, чтобы OpenCode показывал оценку расходов. |

> Блок `websearch` к Yandex отношения не имеет: он задаёт провайдера веб-поиска
> (`parallel`). Можно удалить его вместе с запятой, если поиск не нужен.

## 3. Войдите в аккаунт (необязательно)

В примере ключ берётся из переменной окружения `api_key`, поэтому этот шаг не нужен. Если
хочется хранить ключ не в окружении, а в хранилище OpenCode, откройте `/connect` в TUI
(или выполните `opencode auth login`), выберите провайдера **Yandex** и вставьте API-ключ —
тогда строку `apiKey` из конфига можно убрать.

> В OpenCode 1 секреты лежали в `~/.local/share/opencode/auth.json`. В v2 аккаунты
> хранятся в SQLite-базе сервиса; путь к ней печатает команда `opencode debug paths db`.
> Legacy-файл `auth.json` при переходе на v2 импортируется автоматически.

Чтобы задать переменную окружения самому сервису (а не оболочке, из которой вы
запускаете `opencode`), используйте:

```bash
opencode service set env api_key <API-ключ>
opencode service set env folder_id <folder-id>
```

Это останавливает работающий фоновый сервис; следующая команда `opencode` поднимет его с
новым окружением.

## 4. Запустите OpenCode

```bash
opencode
```

Провайдер `yandex` появится в списке; модели DeepSeek и Qwen выбираются в `/models`.
Варианты рассуждений подключаются суффиксом `#`, например `yandex/DeepSeek-V4.1-Flash#none`.

## Что изменилось относительно OpenCode 1

OpenCode 2 читает конфиги v1 из тех же мест и нормализует их, поэтому
[`opencode/config.json`](opencode/config.json) в v2 тоже продолжит работать. Конвертация в
нативный формат v2 необязательна, но делает конфиг явнее:

| OpenCode 1 | OpenCode 2 |
|---|---|
| `provider` | `providers` |
| `npm: "@ai-sdk/openai-compatible"` | `package: "@opencode/ai/providers/openai-compatible"` (либо `"aisdk:@ai-sdk/openai-compatible"`) |
| `options.baseURL` | `settings.baseURL` |
| `options.apiKey` | `settings.apiKey` |
| `options.headers` | `headers` на уровне провайдера |
| `models.<ключ>.id` | `models.<ключ>.modelID` |
| `models.<ключ>.tool_call: true` | `models.<ключ>.capabilities.tools: true` |
| `models.<ключ>.modalities` | `capabilities.input` и `capabilities.output` |
| `models.<ключ>.reasoning: true` | `capabilities.reasoning.supported: true` (плюс `toggle` и `effort`) |
| `models.<ключ>.interleaved.field` | `compatibility.reasoningField` |
| `models.<ключ>.cost.cache_read` / `cache_write` | `cost.cache.read` / `cost.cache.write` |
| Тарифы за токен | Тарифы за 1 млн токенов (числа в 10⁶ раз больше) |
| `variants: { "high": { ... } }` | `variants: [ { "id": "high", "settings": { ... } } ]` |
| `status: "deprecated"` | `disabled: true` |

Проще всего поручить конвертацию самому OpenCode — попросите его:

```text
Migrate my OpenCode configuration from the V1 format to the native V2 format.
Preserve its behavior and all unrelated settings.
```

## Проверка подключения

```bash
curl https://ai.api.cloud.yandex.net/v1/chat/completions \
  -H "Authorization: Bearer $api_key" \
  -H "OpenAI-Project: $folder_id" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt://'$folder_id'/deepseek-v4.1-flash","messages":[{"role":"user","content":"ping"}],"max_tokens":16}'
```

Успешный ответ — объект `chat.completion` с `choices[].message` и `usage`.

Если модель не выбирается, проверьте:

- обе переменные окружения заданы в процессе сервиса OpenCode (см. `opencode service set env`);
- ключ виден в `opencode auth list`;
- `modelID` собран как `gpt://<folder-id>/<model>` без опечаток.

## См. также

- [Codex CLI по API-ключу](codex.md)
- [Общее предупреждение и настройка API-ключа](README.md)
- [OpenCode v1 по API-ключу](opencode.md)
- [OpenCode v2 по подписке](../subscription/opencode-v2.md)
