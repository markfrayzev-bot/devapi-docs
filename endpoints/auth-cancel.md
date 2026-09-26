# POST /v1/auth/cancel

Досрочно отменить сессию и освободить слот.

**Запрос**

```json
{ "session_id": "<32hex>" }
```

**Ответ**

```json
{ "ok": true, "status": "cancelled" }
```

Повторный запрос в отменённую сессию → `409 already_finished`.
