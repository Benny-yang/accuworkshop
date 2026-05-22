# OTP 簡訊發送 API

發送 OTP 簡訊。

此 API 會先驗證 `Bearer` token（實際作為 `AuthKey` 使用），再依活動設定選擇簡訊商送出訊息，並寫入發送紀錄。

## Endpoint

- Method: `POST`
- Path: `/api/sms/send`
- Content-Type: `application/json`
- Authorization: `Bearer {AuthKey}`

Swagger: `https://otp.litloyal.com/swagger`

## Request Headers

| Header          | Required | Description                                         |
| --------------- | -------- | --------------------------------------------------- |
| `Authorization` | Yes      | 固定使用 `Bearer` scheme，格式為 `Bearer {AuthKey}` |
| `Content-Type`  | Yes      | `application/json`                                  |

## Request Body

```json
{
  "Mobile": "0912345678",
  "Message": "您的驗證碼為 123456",
  "VerificationCode": "123456"
}
```

### 欄位說明

| 欄位               | 型別     | Required | 說明                                                      |
| ------------------ | -------- | -------- | --------------------------------------------------------- |
| `Mobile`           | `string` | Yes      | 收件手機號碼                                              |
| `Message`          | `string` | Yes      | 要發送的簡訊內容                                          |
| `VerificationCode` | `string` | 建議帶入 | 驗證碼原文，會寫入發送紀錄，供後續 `/api/sms/verify` 使用 |

## Response

此 API 的成功與失敗主要透過回傳 JSON 內的 `code` 判斷。

```json
{
  "code": 10
}
```

### Response Body

| 欄位   | 型別  | 說明         |
| ------ | ----- | ------------ |
| `code` | `int` | 業務結果代碼 |

## HTTP Status Codes

| HTTP Status        | 說明                                                 |
| ------------------ | ---------------------------------------------------- |
| `200 OK`           | API 已完成處理，實際成功或失敗請看 `code`            |
| `401 Unauthorized` | 缺少 `Authorization` header，或 scheme 不是 `Bearer` |

## Business Code 對照

| code | 說明                       |
| ---- | -------------------------- |
| `10` | 簡訊發送成功               |
| `11` | 簡訊發送失敗               |
| `15` | `AuthKey` 不存在或不合法   |
| `16` | 呼叫來源 IP 不在允許清單   |
| `17` | 系統例外錯誤               |
| `18` | 寫入發送紀錄失敗           |
| `20` | `AuthKey` 已停用           |
| `99` | 找不到對應的簡訊供應商設定 |

## 範例

### cURL

```bash
curl -X POST 'https://otp.litloyal.com/api/sms/send' \
  -H 'Authorization: Bearer YOUR_AUTH_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "Mobile": "0912345678",
    "Message": "您的驗證碼為 123456",
    "VerificationCode": "123456"
  }'
```

### 成功回應

```json
{
  "code": 10
}
```

### 失敗回應：AuthKey 無效

```json
{
  "code": 15
}
```

## 處理流程

1. 驗證 `Authorization: Bearer {AuthKey}` 是否存在。
2. 以 `AuthKey` 查詢活動設定、啟用狀態與允許來源 IP。
3. 先寫入一筆發送紀錄。
4. 依活動綁定的簡訊供應商實際送出簡訊。
5. 更新發送結果並回傳業務代碼。

## 備註

- 專案 README 明確寫有「不支援長簡訊」，但此 API 本身未在 controller/service 層做長度檢查；若訊息過長，實際行為取決於下游簡訊商。
- 目前程式碼未在 controller 層做欄位必填與格式驗證；若缺少 `Mobile` 或 `Message`，可能在下游簡訊商呼叫時失敗，並回傳 `code = 11` 或其他錯誤碼。
- `VerificationCode` 不直接影響簡訊發送結果，但會被保存到 send log，供驗證流程使用。
