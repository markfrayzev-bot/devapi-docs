# POST /v1/auth/phone

Старт входа по номеру. Long-poll до ~40 с, пока код реально отправится.

**Запрос**

```json
{ "phone": "+79001234567", "domain": "site.com" }
```

* `phone` — номер в формате E.164 (`+7…`). Обязателен.
* `domain` — опционален, информационный.

**Ответ**

```json
{
  "ok": true,
  "status": "code_required",
  "session_id": "<32hex>",
  "code_length": 6,
  "resend_after_ms": 60000,
  "request_count_left": null,
  "expires_in_ms": 600000,
  "request_max_duration_ms": 120000,
  "alt_action_duration_ms": 60000
}
```

* `request_count_left` — остаток суточных отправок; `null` = без лимита.
* `request_max_duration_ms` / `alt_action_duration_ms` — тайминги для совместимости со сторонними интеграциями (потолок long-poll шага `/code` и кулдаун resend).

**Ошибки:** `bad_phone` (400), `rate_limited` (429), `busy` (429), `quota_exceeded` (429, если на ключ задан лимит), `send_failed` (502), `upstream_error` (503), `misconfigured` (503). См. [Частые ошибки](../errors.md).
