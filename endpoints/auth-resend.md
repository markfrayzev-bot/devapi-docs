# POST /v1/auth/resend

Переслать код. Алиас (MLB-совместимый): `POST /v1/auth/code/resend`. Не чаще, чем раз в `resend_after_ms`.

**Запрос**

```json
{ "session_id": "<32hex>" }
```

**Ответ**

```json
{ "ok": true, "status": "code_resent", "resend_after_ms": 60000 }
```

Если первичная отправка ещё идёт, вернётся `{ …, "already_sending": true }` без списания квоты.
