## Upload Policy Attachment API

### Endpoint
POST /ords/nic/polices/check_print

---

### Description
Validates whether a policy is allowed to be printed based on the provided policy information and underwriting details.
The API receives policy details, calls the validation procedure INSProc.MOTOR_POLICY_API.VALIDATE_PRINT_POLICY, and returns an allowed flag indicating whether printing is permitted.

---

---

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

