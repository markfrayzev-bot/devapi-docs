# GET /v1/auth/session/{session_id}

Статус сессии без long-poll.

**Ответ**

```json
{ "ok": true, "status": "code_required" }
```

`status` ∈ `pending | code_required | password_required | registration_required | success | error`.
