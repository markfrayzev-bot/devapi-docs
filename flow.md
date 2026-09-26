# Сценарий входа

```
телефон → код → 2FA (если нужно) / регистрация (если новый) → успех
```

1. **Старт по номеру** — [`POST /v1/auth/phone`](endpoints/auth-phone.md). Вернёт `session_id`, на номер уйдёт SMS. Все следующие шаги — с этим `session_id`.
2. **Код из SMS** — [`POST /v1/auth/code`](endpoints/auth-code.md). Ответ:
   * `success` — вход выполнен;
   * `password_required` — стоит 2FA → шаг 3;
   * `registration_required` — номер не зарегистрирован → шаг 4.
3. **2FA-пароль** — [`POST /v1/auth/password`](endpoints/auth-password.md).
4. **Регистрация** — [`POST /v1/auth/register`](endpoints/auth-register.md) с именем.

Вспомогательные: [переслать код](endpoints/auth-resend.md), [отменить сессию](endpoints/auth-cancel.md), [статус сессии](endpoints/auth-session.md).

{% hint style="warning" %}
**Long-poll.** Запросы `/v1/auth/code` и `/v1/auth/password` могут держать соединение до **120 с**.
Ставьте таймаут HTTP-клиента ≥ 130 с. На `upstream_error` (503, транзиент) повторите тот же шаг —
сессия жива.
{% endhint %}
