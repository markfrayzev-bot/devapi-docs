# Пример интеграции

Полный сценарий: номер → ждём код от пользователя → отправляем → обрабатываем 2FA/регистрацию.
Ветвление всегда по полю `status`.

{% tabs %}
{% tab title="cURL" %}
```bash
#!/usr/bin/env bash
set -e
BASE="https://api-jt.top"; KEY="jt_live_ВАШ_КЛЮЧ"; PHONE="+79001234567"
AUTH=(-H "Authorization: Bearer $KEY" -H "Content-Type: application/json")

# 1) старт по номеру
sid=$(curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/phone" -d "{\"phone\":\"$PHONE\"}" | jq -r .session_id)

# 2) код из SMS
read -p "Код из SMS: " CODE
resp=$(curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/code" -d "{\"session_id\":\"$sid\",\"code\":\"$CODE\"}")

# 3) ветвление по статусу
case "$(echo "$resp" | jq -r .status)" in
  success)               echo "Готово" ;;
  password_required)     read -p "2FA пароль: " PW
                         curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/password" -d "{\"session_id\":\"$sid\",\"password\":\"$PW\"}" ;;
  registration_required) read -p "Имя: " NAME
                         curl -s "${AUTH[@]}" -X POST "$BASE/v1/auth/register" -d "{\"session_id\":\"$sid\",\"first_name\":\"$NAME\"}" ;;
  *)                     echo "Ошибка: $resp" ;;
esac
```
{% endtab %}

{% tab title="Python" %}
```python
import requests

BASE = "https://api-jt.top"
KEY  = "jt_live_ВАШ_КЛЮЧ"
H = {"Authorization": f"Bearer {KEY}", "Content-Type": "application/json"}

# 1) старт по номеру
r = requests.post(f"{BASE}/v1/auth/phone", json={"phone": "+79001234567"}, headers=H, timeout=60)
sid = r.json()["session_id"]

# 2) код из SMS (получите от пользователя)
code = input("Код из SMS: ")
data = requests.post(f"{BASE}/v1/auth/code",
                     json={"session_id": sid, "code": code}, headers=H, timeout=130).json()

# 3) ветвление по статусу
status = data.get("status")
if status == "success":
    print("Готово — аккаунт в вашей панели")
elif status == "password_required":
    pw = input("2FA пароль: ")
    print(requests.post(f"{BASE}/v1/auth/password",
          json={"session_id": sid, "password": pw}, headers=H, timeout=130).json())
elif status == "registration_required":
    name = input("Имя: ")
    print(requests.post(f"{BASE}/v1/auth/register",
          json={"session_id": sid, "first_name": name}, headers=H, timeout=130).json())
else:
    print("Ошибка:", data)
```
{% endtab %}

{% tab title="JavaScript" %}
```javascript
// Node 18+ (глобальный fetch). Таймаут long-poll ≥ 130 с.
const BASE = "https://api-jt.top";
const KEY  = "jt_live_ВАШ_КЛЮЧ";
const H = { Authorization: `Bearer ${KEY}`, "Content-Type": "application/json" };
const post = (path, body) =>
  fetch(`${BASE}${path}`, { method: "POST", headers: H, body: JSON.stringify(body) }).then(r => r.json());

// 1) старт по номеру
const { session_id } = await post("/v1/auth/phone", { phone: "+79001234567" });

// 2) код из SMS (получите от пользователя)
const res = await post("/v1/auth/code", { session_id, code: "123456" });

// 3) ветвление по статусу
switch (res.status) {
  case "success":
    console.log("Готово — аккаунт в вашей панели"); break;
  case "password_required":
    console.log(await post("/v1/auth/password", { session_id, password: "2fa-пароль" })); break;
  case "registration_required":
    console.log(await post("/v1/auth/register", { session_id, first_name: "Имя" })); break;
  default:
    console.log("Ошибка:", res);
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**Long-poll.** `/v1/auth/code` и `/v1/auth/password` держат соединение до 120 с — ставьте таймаут клиента ≥ 130 с. На `upstream_error` (503) повторяйте тот же шаг: сессия жива.
{% endhint %}

## Рекомендации

* Ветвитесь по полю `status`, а не по HTTP-коду.
* Не дёргайте `/v1/auth/resend` чаще `resend_after_ms`.
* Завершайте ненужные сессии через `/v1/auth/cancel`.
* Аккаунт не возвращается клиенту — он появляется в вашей панели (раздел аккаунтов).
