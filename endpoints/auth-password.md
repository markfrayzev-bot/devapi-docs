# POST /v1/auth/password

2FA-пароль (после `password_required`).

**Запрос**

```json
{ "session_id": "<32hex>", "password": "пароль-2fa" }
```

**Ответ**

```json
{ "ok": true, "status": "success", "account": { } }
{ "ok": false, "error": "wrong_password" }
```

`wrong_password` (400) — сессия жива, повтор возможен.
