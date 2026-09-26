# Пример интеграции

Полный сценарий на bash: номер → ждём код от пользователя → отправляем → обрабатываем 2FA/регистрацию.

```bash
#!/usr/bin/env bash
set -e
BASE="https://api-jt.top"
KEY="jt_live_ВАШ_КЛЮЧ"
PHONE="+79001234567"
AUTH=(-H "Authorization: Bearer $KEY" -H "Content-Type: application/json")

# 1) старт по номеру
resp=$(curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/phone" -d "{\"phone\":\"$PHONE\"}")
sid=$(echo "$resp" | jq -r .session_id)
echo "session=$sid — код отправлен, спросите его у пользователя"

# 2) код из SMS
read -p "Код из SMS: " CODE
resp=$(curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/code" \
       -d "{\"session_id\":\"$sid\",\"code\":\"$CODE\"}")
status=$(echo "$resp" | jq -r .status)

# 3) ветвление по статусу
case "$status" in
  success)               echo "Готово: вход выполнен" ;;
  password_required)     read -p "2FA пароль: " PW
                         curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/password" \
                              -d "{\"session_id\":\"$sid\",\"password\":\"$PW\"}" ;;
  registration_required) read -p "Имя: " NAME
                         curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/register" \
                              -d "{\"session_id\":\"$sid\",\"first_name\":\"$NAME\"}" ;;
  *)                     echo "Ошибка: $resp" ;;
esac
```

Рекомендации:

* Ветвитесь по полю `status`, а не по HTTP-коду.
* На `upstream_error` (503) повторяйте тот же шаг — это транзиент, сессия жива.
* Не дёргайте `/v1/auth/resend` чаще `resend_after_ms`.
* Завершайте ненужные сессии через `/v1/auth/cancel`, чтобы не упираться в лимит одновременных сессий.
