# POST /v1/auth/register

Регистрация нового аккаунта (после `registration_required`).

**Запрос**

```json
{ "session_id": "<32hex>", "first_name": "Имя" }
```

**Ответ**

```json
{ "ok": true, "status": "success", "balance": 42 }   // аккаунт — в вашей панели; balance = ваши успешные авторизации
```
