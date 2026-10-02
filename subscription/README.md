# Алиса AI Про: подключение ассистентов по подписке

Материалы для пользователей подписки **Алиса AI Про**. Авторизация выполняется по
IAM-токену Yandex Cloud, отдельный API-ключ не требуется.

| Документ | О чём | Готовый конфиг |
|---|---|---|
| [YCodex](ycodex.md) | Консольный ассистент YCodex: установка, вход, запуск | [`ycodex/config.toml`](ycodex/config.toml) |
| [Codex CLI](codex.md) | Подключение стандартного Codex CLI к подписке по IAM-токену | [`codex/alice-ai-pro.config.toml`](codex/alice-ai-pro.config.toml), [`codex/alice-ai-pro.models_catalog.json`](codex/alice-ai-pro.models_catalog.json) |
| [OpenCode](opencode.md) | Подключение OpenCode к подписке по IAM-токену | [`opencode/config.json`](opencode/config.json) |

Оформить подписку: <https://aistudio.yandex.ru/manage>

Официальные источники:

- Профиль сообщества: <https://github.com/serjnarbut/yandex-aistudio-profile>
- Подписка в документации AI Studio: <https://aistudio.yandex.ru/ru/docs/ai-studio/manage/concepts/subscription>

> Если вы ищете вариант с API-ключом, он описан в [../api/](../api/). Там же сказано,
> почему так делать не стоит: оплата по токенам выходит очень дорого.
