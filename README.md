# Partner Integration Guide
## `POST /request/member/update` — Update Member by Card Number

**Version:** 1.0.0 | **Base URL:** `https://<host>/api/v2` | **Updated:** April 2026

> **Confidential** — For authorised JDB partners only.

---

## Overview

This guide covers **all APIs** a partner must call when integrating with `POST /request/member/update`.

### Integration Flow

```
Step 1  POST  /login                    → Get JWT token
Step 2  GET   /prefix                   → Get valid prefix codes
Step 3  GET   /gender                   → Get valid gender codes
Step 4  GET   /ISDCode                  → Get valid ISD dialling codes
Step 5  GET   /country                  → Get valid country codes
Step 6  GET   /document                 → Get valid document types
Step 7  GET   /getAddressE              → Get province/district/village (English)
Step 8  POST  /request/member/update    → Submit the member update request
```

> **Note:** Steps 2–7 are reference data lookups. Their results are **cached** server-side (Redis). You can call them once and cache the results on your side, refreshing periodically.

---

## Required Headers (All APIs)

| Header | Required | Description |
|---|---|---|
| `Content-Type` | ✅ | `application/json` |
| `api-key` | ✅ | Encrypted API key issued by JDB |
| `Authorization` | ✅ (except login) | `Bearer <token>` from Step 1 |

---

## Step 1 — Authenticate

### `POST /login`

Obtain a JWT Bearer token. **Token expires in 1 hour.**

**Auth required:** `api-key` only (no JWT yet)
**Rate limit:** 10 requests / 15 min / IP

#### Request

```http
POST /api/v2/login
Content-Type: application/json
api-key: <encrypted_api_key>
```

```json
{
  "userName": "your_username",
  "password": "<AES_encrypted_password>",
  "actionNode": "1"
}
```

| Field | Type | Required | Max | Description |
|---|---|---|---|---|
| `userName` | string | ✅ | 20 | Partner username provided by JDB |
| `password` | string | ✅ | 60 | Password AES-encrypted with secret key provided by JDB |
| `actionNode` | string | ✅ | 1 | Must always be `"1"` |

#### Success Response `200`

```json
{
  "response": [
    {
      "data": {
        "id": 123456,
        "name": "Partner Name",
        "mail": "partner@example.com",
        "phone": "2055512345"
      },
      "status": true,
      "message": "Success",
      "responseCode": "00",
      "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  ]
}
```

> ✅ **Save `response[0].token`** — use it as `Authorization: Bearer <token>` in all subsequent requests.

#### Error Responses

| `responseCode` | HTTP | Cause | Action |
|---|---|---|---|
| `01` | 400 | Missing field | Check `userName`, `password`, `actionNode`, `api-key` |
| `02` | 401 | Username not found | Verify username with JDB |
| `03` | 401 | Wrong password | Check password encryption |
| `09` | 401 | Account not activated | Contact JDB support |
| `10` | 401 | Account locked | Contact JDB support |
| `11` | 401 | Invalid `api-key` | Verify your API key with JDB |
| `31` | 400 | Field length exceeded | Shorten the field value |
| `32` | 400 | `actionNode` must be `"1"` | Fix `actionNode` value |

---

## Step 2 — Get Prefix List

### `GET /prefix`

Returns valid name prefix codes to use in the `prefix` field.

**Auth required:** `api-key` + `Authorization: Bearer <token>`

#### Request

```http
GET /api/v2/prefix
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

No request body required.

#### Success Response `200`

```json
{
  "status": true,
  "message": "Success",
  "responseCode": "00",
  "data": [
    { "prefixCode": "MR",  "prefixName": "Mr"  },
    { "prefixCode": "MRS", "prefixName": "Mrs" },
    { "prefixCode": "MS",  "prefixName": "Ms"  },
    { "prefixCode": "DR",  "prefixName": "Dr"  }
  ]
}
```

> Use `prefixCode` or `prefixName` value as the `prefix` field in `POST /request/member/update`.

#### Error Responses

| `responseCode` | HTTP | Cause |
|---|---|---|
| `06` | 401 | Invalid or expired JWT token |
| `11` | 401 | Invalid `api-key` |
| `99` | 500 | Internal server error |

---

## Step 3 — Get Gender List

### `GET /gender`

Returns valid gender codes to use in the `gender` field.

**Auth required:** `api-key` + `Authorization: Bearer <token>`

#### Request

```http
GET /api/v2/gender
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

No request body required.

#### Success Response `200`

```json
{
  "status": true,
  "message": "Success",
  "responseCode": "00",
  "data": [
    { "genderCode": "M", "genderName": "Male"   },
    { "genderCode": "F", "genderName": "Female" }
  ]
}
```

> Use `genderCode` as the `gender` field in `POST /request/member/update`.

#### Error Responses

| `responseCode` | HTTP | Cause |
|---|---|---|
| `06` | 401 | Invalid or expired JWT token |
| `11` | 401 | Invalid `api-key` |
| `99` | 500 | Internal server error |

---

## Step 4 — Get ISD Code List

### `GET /ISDCode`

Returns international dialling codes to use in the `ISDNo` field.

**Auth required:** `api-key` + `Authorization: Bearer <token>`

#### Request

```http
GET /api/v2/ISDCode
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

No request body required.

#### Success Response `200`

```json
{
  "status": true,
  "message": "Success",
  "responseCode": "00",
  "data": [
    { "isdCode": "+856", "countryName": "Laos"      },
    { "isdCode": "+66",  "countryName": "Thailand"  },
    { "isdCode": "+1",   "countryName": "USA"       }
  ]
}
```

> Use `isdCode` as the `ISDNo` field in `POST /request/member/update`.

#### Error Responses

| `responseCode` | HTTP | Cause |
|---|---|---|
| `06` | 401 | Invalid or expired JWT token |
| `11` | 401 | Invalid `api-key` |
| `99` | 500 | Internal server error |

---

## Step 5 — Get Country List

### `GET /country`

Returns country codes to use in `birth_country` field.

**Auth required:** `api-key` + `Authorization: Bearer <token>`

#### Request

```http
GET /api/v2/country
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

No request body required.

#### Success Response `200`

```json
{
  "status": true,
  "message": "Success",
  "responseCode": "00",
  "data": [
    { "countryCode": "LA", "countryName": "Laos"          },
    { "countryCode": "TH", "countryName": "Thailand"      },
    { "countryCode": "US", "countryName": "United States" }
  ]
}
```

> Use `countryCode` (ISO 3166-1 alpha-2) as the `birth_country` field.

---

## Step 6 — Get Document Type List

### `GET /document`

Returns valid document types to use in `document_category`, `document_type`, and `document_desc` fields.

**Auth required:** `api-key` + `Authorization: Bearer <token>`

#### Request

```http
GET /api/v2/document
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

No request body required.

#### Success Response `200`

```json
{
  "status": true,
  "message": "Success",
  "responseCode": "00",
  "data": [
    {
      "documentCategory": "NATIONAL_ID",
      "documentType": "NID",
      "documentDesc": "National Identity Card"
    },
    {
      "documentCategory": "PASSPORT",
      "documentType": "PP",
      "documentDesc": "Passport"
    },
    {
      "documentCategory": "FAMILY_BOOK",
      "documentType": "FB",
      "documentDesc": "Family Book"
    }
  ]
}
```

> Map the fields as follows for `POST /request/member/update`:
>
> | Document field | Source |
> |---|---|
> | `document_category` | `documentCategory` |
> | `document_type` | `documentType` |
> | `document_desc` | `documentDesc` |

---

## Step 7 — Get Address (English)

### `POST /getAddressE`

Returns province, district, and village data in English for address fields.

**Auth required:** `api-key` + `Authorization: Bearer <token>`

#### Request

```http
POST /api/v2/getAddressE
Content-Type: application/json
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

```json
{}
```

(Empty body or omit body — no parameters required)

#### Success Response `200`

```json
{
  "status": true,
  "message": "Success",
  "responseCode": "00",
  "data": [
    {
      "provinceCode": "VTE",
      "provinceName": "Vientiane Capital",
      "districts": [
        {
          "districtCode": "CHA",
          "districtName": "Chanthabouly",
          "villages": [
            { "villageCode": "SIM", "villageName": "Simuang" },
            { "villageCode": "PHO", "villageName": "Phonxay" }
          ]
        }
      ]
    }
  ]
}
```

> Map the fields as follows:
>
> | Request field | Source |
> |---|---|
> | `province` | `provinceName` |
> | `district` | `districtName` |
> | `village` | `villageName` |

---

## Step 8 — Submit Member Update Request

### `POST /request/member/update`

Submit a member update request using an existing card number. The system automatically retrieves `CIF`, `account_no`, and `product_code` from the card.

**Auth required:** `api-key` + `Authorization: Bearer <token>`
**Rate limit:** 1,000 requests / 15 min / IP

#### Request

```http
POST /api/v2/request/member/update
Content-Type: application/json
api-key: <encrypted_api_key>
Authorization: Bearer <token>
```

```json
{
  "card_number":          "4466146697261212",
  "account_name":         "HAMISH TAILOR",
  "prefix":               "Mr",
  "first_name":           "Hamish",
  "last_name":            "Tailor",
  "gender":               "M",
  "dob":                  "1985-06-20",
  "place_of_birth":       "Vientiane",
  "birth_country":        "LA",
  "village":              "Phonxay",
  "district":             "Xaysetha",
  "province":             "Vientiane Capital",
  "ISDNo":                "+856",
  "telephone":            "2099812345",
  "mail":                 "hamish.tailor@example.com",
  "occupation":           "Business Owner",
  "document_category":    "PASSPORT",
  "document_type":        "PP",
  "document_desc":        "Passport",
  "document_id":          "AB123456",
  "document_issued_date": "2019-03-01",
  "document_expiry_date": "2029-03-01",
  "card_first_name":      "HAMISH",
  "card_last_name":       "TAILOR",
  "take_photo_card":      "<base64_string>",
  "take_photo_with_card": "<base64_string>",
  "signature":            "<base64_string>"
}
```

### Request Field Reference

#### 🔑 Card Lookup (auto-filled by system)

| Field | Type | Required | Description |
|---|---|---|---|
| `card_number` | string | ✅ | Existing JDB card number. System auto-fills `CIF`, `account_no`, `product_code`. Must already exist in JDB. |

#### 👤 Personal Information

| Field | Type | Required | Max | Description | Source |
|---|---|---|---|---|---|
| `account_name` | string | ✅ | 100 | Full account name | Partner input |
| `prefix` | string | ✅ | 5 | Title code | `GET /prefix` → `prefixCode` |
| `first_name` | string | ✅ | 50 | First name (English) | Partner input |
| `last_name` | string | ✅ | 50 | Last name (English) | Partner input |
| `gender` | string | ✅ | 1 | Gender code | `GET /gender` → `genderCode` |
| `dob` | string | ✅ | — | Date of birth `YYYY-MM-DD` | Partner input |
| `place_of_birth` | string | ✅ | 50 | City/village of birth | Partner input |
| `birth_country` | string | ✅ | 10 | ISO country code | `GET /country` → `countryCode` |
| `occupation` | string | ✅ | 100 | Occupation | Partner input |

#### 📍 Address

| Field | Type | Required | Max | Description | Source |
|---|---|---|---|---|---|
| `village` | string | ✅ | 50 | Village name | `POST /getAddressE` → `villageName` |
| `district` | string | ✅ | 50 | District name | `POST /getAddressE` → `districtName` |
| `province` | string | ✅ | 50 | Province name | `POST /getAddressE` → `provinceName` |

#### 📞 Contact

| Field | Type | Required | Max | Description | Source |
|---|---|---|---|---|---|
| `ISDNo` | string | ✅ | 4 | International dialling code | `GET /ISDCode` → `isdCode` |
| `telephone` | string | ✅ | 20 | Mobile number (digits only) | Partner input |
| `mail` | string | ✅ | 50 | Valid email address | Partner input |

#### 🪪 Identity Document

| Field | Type | Required | Max | Description | Source |
|---|---|---|---|---|---|
| `document_category` | string | ✅ | 100 | Document category | `GET /document` → `documentCategory` |
| `document_type` | string | ✅ | 100 | Document type code | `GET /document` → `documentType` |
| `document_desc` | string | ✅ | 100 | Document description | `GET /document` → `documentDesc` |
| `document_id` | string | ✅ | 20 | ID / passport number | Partner input |
| `document_issued_date` | string | ✅ | 20 | Issue date `YYYY-MM-DD` | Partner input |
| `document_expiry_date` | string | ✅ | 20 | Expiry date `YYYY-MM-DD` | Partner input |

#### 💳 Card Embossing

| Field | Type | Required | Max | Description |
|---|---|---|---|---|
| `card_first_name` | string | ✅ | 50 | First name on card — **UPPERCASE only** |
| `card_last_name` | string | ✅ | 50 | Last name on card — **UPPERCASE only** |

#### ⚙️ Optional Fields (system defaults applied)

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `currency` | string | ❌ | `"USD"` | Account currency — omit to use default |
| `minimum_amount` | string/number | ❌ | `"0"` | Minimum balance — omit to use default |
| `user_internet_banking` | string | ❌ | `"N"` | Enrol internet banking: `"Y"` or `"N"` |

> These fields do **not** need to be sent. If omitted, defaults are applied automatically.

#### 🖼️ Images (Base64)

| Field | Type | Required | Description |
|---|---|---|---|
| `take_photo_card` | string | ✅ | Photo of the identity document (Base64) |
| `take_photo_with_card` | string | ✅ | Photo of customer holding the document (Base64) |
| `signature` | string | ✅ | Customer signature (Base64) |

**Image encoding rules:**
- Format: **JPEG or PNG**
- Encoding: **Pure Base64** — no `data:image/...;base64,` prefix
- Max size: **5 MB** per image

Correct:
```
/9j/4AAQSkZJRgABAQEASABIAAD...
```
Incorrect:
```
data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEASABIAAD...
```

---

#### Success Response `200`

```json
{
  "message": "Successful request; kindly wait as staff checks information.",
  "responseCode": "00",
  "status": true,
  "batch_no": 748291034,
  "transationId": 748291034
}
```

| Field | Type | Description |
|---|---|---|
| `responseCode` | string | `"00"` = accepted successfully |
| `status` | boolean | `true` = request accepted |
| `batch_no` | number | **Save this.** Use to track status and match webhook callbacks. |
| `transationId` | number | Same as `batch_no` |

---

#### Error Responses

| `responseCode` | HTTP | Cause | Action |
|---|---|---|---|
| `01` | 400 | Missing required field(s) — `message` contains field name(s) | Add the missing field(s) |
| `04` | 400 | Invalid email / card name / document dates | Fix the invalid value |
| `06` | 401 | JWT token invalid or expired | Re-authenticate via `POST /login` |
| `11` | 401 | Invalid `api-key` | Verify your `api-key` with JDB |
| `12` | 401 | JWT token expired | Re-authenticate via `POST /login` |
| `31` | 400 | Field length exceeded — `message` contains field name(s) | Shorten the field value |
| `ERR_CARD_NOT_FOUND` | 400 | `card_number` not found in JDB system | Verify the card number |
| `ERR_CARD_API` | 502 | JDB card lookup service temporarily unavailable | Retry after 30 seconds |
| `99` | 500 | Internal server error | Retry; contact JDB if persists |

---

## Webhook Callback

After JDB staff reviews the request, a webhook notification is sent to your registered URL.

```json
{
  "type":      "UPDATE_MEMBER_REQUEST",
  "batchNo":   748291034,
  "idFrom":    512847361,
  "status":    "APPROVED",
  "partnerId": 123456,
  "timestamp": "2026-04-24T10:30:00.000Z"
}
```

| Field | Description |
|---|---|
| `type` | Always `"UPDATE_MEMBER_REQUEST"` for this endpoint |
| `batchNo` | Matches `batch_no` from the submit response |
| `status` | `REQUEST` → `APPROVED` or `REJECTED` |

> Your webhook endpoint **must return HTTP `200`** to confirm receipt.

---

## Complete Integration Checklist

- [ ] Obtain `api-key` from JDB
- [ ] Obtain username / AES-encrypted password from JDB
- [ ] Register your webhook URL with JDB
- [ ] Call `POST /login` → save token
- [ ] Call `GET /prefix` → cache prefix list
- [ ] Call `GET /gender` → cache gender list
- [ ] Call `GET /ISDCode` → cache dialling codes
- [ ] Call `GET /country` → cache country codes
- [ ] Call `GET /document` → cache document types
- [ ] Call `POST /getAddressE` → cache address data
- [ ] Build the request body mapping data from reference APIs
- [ ] Ensure images are **pure Base64** (no `data:image/...` prefix)
- [ ] Submit `POST /request/member/update` → save `batch_no`
- [ ] Implement token refresh on `responseCode: "06"` or `"12"`
- [ ] Handle `ERR_CARD_NOT_FOUND` in your error handling
- [ ] Webhook endpoint returns HTTP `200` on receipt
- [ ] Match webhook `batchNo` to stored `batch_no`

---

## Reference: Field-to-API Mapping Summary

| Request Field | API to call | Response field to use |
|---|---|---|
| `prefix` | `GET /prefix` | `prefixCode` |
| `gender` | `GET /gender` | `genderCode` |
| `ISDNo` | `GET /ISDCode` | `isdCode` |
| `birth_country` | `GET /country` | `countryCode` |
| `document_category` | `GET /document` | `documentCategory` |
| `document_type` | `GET /document` | `documentType` |
| `document_desc` | `GET /document` | `documentDesc` |
| `province` | `POST /getAddressE` | `provinceName` |
| `district` | `POST /getAddressE` | `districtName` |
| `village` | `POST /getAddressE` | `villageName` |
| `card_number` | *(partner provides)* | Auto-lookup: `CIF`, `account_no`, `product_code` |
| `currency` | *(not required)* | Default: `"USD"` |
| `minimum_amount` | *(not required)* | Default: `"0"` |
| `user_internet_banking` | *(not required)* | Default: `"N"` |

---

*© 2026 JDB Bank — Partner API Documentation. Confidential. Authorised partners only.*
