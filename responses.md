# Формат ответов

Все ответы — JSON. Успех — с полем `"ok": true`, ошибка — с `"ok": false`.

```json
// успех
{ "ok": true, "status": "code_required", "session_id": "…" }

// ошибка
{ "ok": false, "error": "bad_phone", "message": "phone must be E.164, e.g. +79001234567" }
```

Ветвитесь по полю `status` (успех) и `error` (ошибка), а не по HTTP-коду.

## Объект `account`

Возвращается при `status: "success"` (в `/v1/auth/code`, `/v1/auth/password`, `/v1/auth/register`) — это данные авторизованного аккаунта MAX.

```json
{
  "ok": true,
  "status": "success",
  "account": {
    "token": "<сессионный токен MAX>",     // храните в секрете
    "user_id": 100000001,                   // числовой ID аккаунта в MAX
    "max_name": "Имя",                      // имя профиля
    "mt_instance_id": "<uuid>",             // идентификатор сессии-инстанса
    "device_type": "ANDROID",               // тип устройства токена
    "connection_params": { … }              // профиль устройства для восстановления сессии
  }
}
```

| Поле | Тип | Описание |
| ---- | --- | -------- |
| `token` | string | Сессионный токен MAX. Секрет — не логируйте, не отдавайте в браузер. |
| `user_id` | number | Числовой ID аккаунта. |
| `max_name` | string | Имя профиля. |
| `mt_instance_id` | string | ID инстанса сессии. |
| `device_type` | string | `ANDROID` / `WEB`. |
| `connection_params` | object | Профиль устройства (deviceId, версии, локаль). |

## HTTP-коды

| Код | Значение |
| --- | -------- |
| `200` | Успех либо не-терминальная ошибка ввода (`wrong_code`/`wrong_password` тоже приходят телом). |
| `400` | Неверный ввод (`bad_phone`, `wrong_code`, `wrong_password`, `bad_request`). |
| `401` | Не авторизован (`unauthorized`) — проверьте ключ. |
| `403` | Доступ запрещён (`account_blocked`). |
| `404` | Сессия не найдена/истекла (`session_expired`). |
| `409` | Конфликт состояния (`already_finished`, `code_expired`). |
| `429` | Лимиты (`rate_limited`, `quota_exceeded`, `busy`, `code_limit`). |
| `502` | `send_failed` — код не отправился. |
| `503` | `upstream_error` (транзиент, повторить) / `misconfigured`. |
