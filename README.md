# Инструкции по подключению кодинг-агентов к Yandex AI Studio

В этой инструкции мы описываем, как поключить к [Yandex AI Studio](https://aistudio.yandex.ru) три кодинг-агента — **[OpenCode](https://opencode.ai)**, **[Codex](https://openai.com/codex)**, а также специализированную версию Codex для Yandex AI Studio **YCodex**. При этом рассматриваются 2 варианта доступа:
по подписке и по API-ключу.

| Агент | По подписке (рекомендуется) | По API-ключу |
|---|---|---|
| **OpenCode** | [клик](subscription/opencode.md) | [клик](api/opencode.md) |
| **Codex CLI** | [клик](subscription/codex.md) | [клик](api/codex.md) |
| **YCodex** | [клик](subscription/ycodex.md) | не поддерживается |

## По подписке «Алиса AI Про»

Подписка даёт доступ к моделям Yandex AI Studio за фиксированную плату по предоплате.
Авторизация проходит по IAM-токену Yandex Cloud (`yc iam create-token`), отдельный
API-ключ не нужен. Это рекомендуемый способ: предсказуемая стоимость и доступ к
актуальным моделям.

Подробности:

- [OpenCode по подписке](subscription/opencode.md)
- [Codex CLI по подписке](subscription/codex.md)
- [YCodex](subscription/ycodex.md)

Оформить подписку: <https://aistudio.yandex.ru/manage>

## По API-ключу

> **Внимание: это очень дорого, и так делать не стоит.** Оплата идёт за каждый токен по
> тарифам Yandex AI Studio, поэтому активная работа кодинг-агента быстро обнуляет бюджет.
> Используйте этот способ только для экспериментов или точечной автоматизации. Для
> постоянной работы берите подписку.

Подробности:

- [Общий порядок и предупреждение о стоимости](api/README.md)
- [OpenCode по API-ключу](api/opencode.md)
- [Codex CLI по API-ключу](api/codex.md)

## SourceCraft CLI

[SourceCraft CLI](https://sourcecraft.dev/portal/docs/ru/sourcecraft/operations/cli-quickstart)
(`src`) — это тот же **OpenCode**, встроенный в платформу SourceCraft: он подключён к
моделям Code Assistant и умеет работать с репозиториями, задачами и предложениями
изменений прямо из терминала. На старте доступны бесплатные лимиты, поэтому для знакомства с агентом это самый простой вариант.

```bash
# macOS / Linux
curl -fsSL https://s3.yandexcloud.net/sourcecraft-cli/install.sh | sh

# Windows (PowerShell)
iex (New-Object System.Net.WebClient).DownloadString('https://s3.yandexcloud.net/sourcecraft-cli/install.ps1')
```

Запуск агента в репозитории — `src code`, разовая задача без интерфейса — `src do "..."`.
Полная инструкция: <https://sourcecraft.dev/portal/docs/ru/sourcecraft/operations/cli-quickstart>.

## Официальные источники

- Актуальная инструкция по подключению кодинг-ассистентов по подписке: <https://github.com/serjnarbut/yandex-aistudio-profile>
- Про подписку в документации AI Studio: <https://aistudio.yandex.ru/ru/docs/ai-studio/manage/concepts/subscription>
- Оформление подписки: <https://aistudio.yandex.ru/manage>
