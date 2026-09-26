# Быстрый старт

Все запросы — `POST`/`GET` с телом JSON, авторизация вашим ключом в заголовке.

|                 |                                   |
| --------------- | --------------------------------- |
| **Base URL**    | `https://api-jt.top`              |
| **Формат**      | `application/json`                |
| **Авторизация** | `Authorization: Bearer jt_live_…` |

Минимальный запрос — старт входа по номеру:

```bash
curl -X POST https://api-jt.top/v1/auth/phone \
  -H "Authorization: Bearer jt_live_ВАШ_КЛЮЧ" \
  -H "Content-Type: application/json" \
  -d '{"phone":"+79001234567"}'
```

Ответ:

```json
{
  "ok": true,
  "status": "code_required",
  "session_id": "a3f00c1d9b1c4f58a8f2c7d1e9b0a111",
  "code_length": 6,
  "resend_after_ms": 60000,
  "request_count_left": null,
  "expires_in_ms": 600000
}
```

Дальше отправляете код из SMS в [`/v1/auth/code`](endpoints/auth-code.md) с этим `session_id`.
Полный сценарий — в разделе [Сценарий входа](flow.md), готовый скрипт — в [Пример интеграции](integration.md).
