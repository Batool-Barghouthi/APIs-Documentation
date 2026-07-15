## Validate Policy Print Permission API

### Endpoint
POST /ords/nic/polices/check_print

---

### Description
Validates whether a policy is allowed to be printed based on the provided policy information and underwriting details.
The API receives policy details, calls the validation procedure INSProc.MOTOR_POLICY_API.VALIDATE_PRINT_POLICY, and returns an allowed flag indicating whether printing is permitted.


and returns an `allowed` flag indicating whether policy printing is permitted.

---

### Request Headers

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |

---

### Request Body

```json
{
  "BRANCH": 10,
  "OFFICE": 10,
  "DOC_NO": 700003,
  "DOC_UW_YEAR": 2026,
  "DOC_TYPE": 1,
  "MAJ_INS_TYPE": 33,
  "MIN_INS_TYPE": 9
}

### Success Response
**HTTP Status:** `200 OK`

**Response Body**
```json
{
  "status": "S",
  "code": 0,
  "message": "File uploaded successfully",
  "http_status": 200,
  "file_name": "bat.png",
  "directory": "shared_dir"
}
```

---

### Error Response
**HTTP Status:** `4xx` / `5xx`

**Response Body**
```json
{
  "status": "E",
  "code": 409,
  "message": "Upload endpoint returned a non-2xx status",
  "file_name": "bat.png",
  "directory": "shared_dir"
}
```

---

