# POST /v1/auth/code

Проверка SMS-кода. Long-poll до 120 с.

**Запрос**

```json
{ "session_id": "<32hex>", "code": "123456" }
```

**Ответы**

```json
{ "ok": true, "status": "success", "account": { } }
{ "ok": true, "status": "password_required" }
{ "ok": true, "status": "registration_required" }
{ "ok": false, "error": "wrong_code" }
```

* `success` — вход выполнен (2FA не стоит), аккаунт у вас в панели, +1 к успешным авторизациям.
* `password_required` — нужен 2FA → [`/v1/auth/password`](auth-password.md).
* `registration_required` — новый аккаунт → [`/v1/auth/register`](auth-register.md).
* `wrong_code` (400) — сессия жива, можно повторить.

**Терминальные** (повтор → 409 `already_finished`): `code_limit` (429), `code_expired` (409), `account_blocked` (403). Транзиент → `upstream_error` (503), сессия жива, повторите шаг.
