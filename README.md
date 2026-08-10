# Travel Coupon API — General Insurance

ORDS REST API for issuing NIC travel insurance coupons: price quotation, coupon creation, and policy printing.

| | |
|---|---|
| **Base URL (test)** | `http://172.16.2.160:8889/ords/nic/TRAVEL_COUPON` |
| **ORDS schema alias** | `nic` |
| **Module** | `TRAVEL_COUPON` |
| **Backend** | `xx_workflow.gen_api_pkg` (over database link `@TESTDB`) |
| **Content-Type** | `application/json` |
| **Auth** | _To be confirmed — see Open Questions_ |

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/calculate_price` | Quote the premium for a travel coupon before issuing it |
| `POST` | `/CreateCoupon` | Issue the travel coupon and create the policy document |
| `POST` | `/print` | Render the issued policy via JasperReports |

**Typical flow:** `calculate_price` → show price to customer → `CreateCoupon` → take `policy_info` from the response → `print`.

---

## Conventions

### Response envelope

Every endpoint returns a JSON object carrying a status flag.

| Field | Type | Description |
|---|---|---|
| `status` | string | `S` = success, `E` = error |
| `code` | number | HTTP-style code (`400`, `422`, `500`) or business error code (e.g. `1001`) |
| `error_type` | string | Error category — see table below |
| `message` | string | Human-readable message |
| `details` | string | Optional. Usually `SQLERRM` for server-side failures |

> **Note:** `CreateCoupon` currently returns the success flag as `Status` (capital S) while errors use `status` (lowercase). Clients should read the key case-insensitively until this is normalised. See Known Issues.

### Error types

| `error_type` | Meaning | Raised by |
|---|---|---|
| `JSON_PARSE` | Request body is not valid JSON, or the expected wrapper object is missing | all |
| `REQUIRED` | A mandatory field is absent | `/print` |
| `TRAVEL_COUPON` | Business rule rejected the coupon | `/CreateCoupon` |
| `PLSQL` / `SERVER` | Unhandled database exception | all |

### Data formats

- **Dates** — `DD-MM-YYYY` (e.g. `11-02-2026`).
- **Numbers** — sent as JSON numbers, not strings (`"p_id": 406213058`).
- **Mobile numbers** — sent as strings to preserve the leading zero (`"0599123456"`).

---

## 1. Calculate Price

Returns the premium for a proposed travel coupon without persisting anything.

```
POST /ords/nic/TRAVEL_COUPON/calculate_price
```

### Request body

The payload is wrapped in a `travel_coupon` object. The wrapper is extracted and passed through to the pricing package as-is.

```json
{
  "travel_coupon": {
    "p_id": 406213058,
    "p_birth_dt": "01-01-1990",
    "p_ins_st_dt": "11-02-2026",
    "p_ins_ed_dt": "24-02-2026",
    "p_am_can_flag": 1
  }
}
```

Field definitions are shared with `/CreateCoupon` — see the [Field Reference](#field-reference).

### Behaviour

The handler supplies two values server-side; they are **not** read from the request body:

| Parameter | Value | Note |
|---|---|---|
| `p_branch` | `10` | Hardcoded in the handler |
| `p_user_no` | `1001` | Hardcoded in the handler |

The response from `gen_api_pkg.calculate_travel_policy_price` is written to the output stream verbatim.

### Success response

_Sample pending — the shape is whatever the pricing package emits in `o_json_out`._

### Error responses

```json
{
  "status": "E",
  "code": 400,
  "error_type": "JSON_PARSE",
  "message": "Invalid JSON body"
}
```

```json
{
  "status": "E",
  "code": 500,
  "error_type": "SERVER",
  "message": "PL/SQL Error",
  "details": "ORA-01403: no data found"
}
```

### Example

```bash
curl -X POST http://172.16.2.160:8889/ords/nic/TRAVEL_COUPON/calculate_price \
  -H "Content-Type: application/json" \
  -d '{
        "travel_coupon": {
          "p_id": 406213058,
          "p_birth_dt": "01-01-1990",
          "p_ins_st_dt": "11-02-2026",
          "p_ins_ed_dt": "24-02-2026",
          "p_am_can_flag": 1
        }
      }'
```

---

## 2. Create Coupon

Creates the customer (if new), issues the travel coupon, and returns the policy keys needed for printing and accounting.

```
POST /ords/nic/TRAVEL_COUPON/CreateCoupon
```

### Request body

```json
{
  "travel_coupon": {
    "p_id": 406213058,
    "p_birth_dt": "01-01-1990",
    "p_ins_st_dt": "11-02-2026",
    "p_ins_ed_dt": "24-02-2026",
    "p_passport_id": 987654,
    "p_am_can_flag": 1,
    "p_mobile_no": "0599123456",
    "acm_email": "test@test.com",
    "acm_aname": "Ahmed",
    "acm_lname": "Ali"
  }
}
```

### Success response

```json
{
  "policy_info": {
    "doc_no": 48249,
    "branch": 10,
    "office": 1,
    "uw_year": 2026,
    "doc_type": 1,
    "maj_ins_type": 34,
    "min_ins_type": 4,
    "cust_no": 8406213058,
    "db_acc_no": 113210014942
  },
  "message": "Travel coupon created successfully",
  "Status": "S"
}
```

#### `policy_info`

| Field | Type | Description |
|---|---|---|
| `doc_no` | number | Policy document number — the coupon identifier |
| `branch` | number | Issuing branch code |
| `office` | number | Issuing office code |
| `uw_year` | number | Underwriting year |
| `doc_type` | number | Document type (`1` = policy) |
| `maj_ins_type` | number | Major insurance type — `34` for travel |
| `min_ins_type` | number | Minor insurance type — `4` for travel coupon |
| `cust_no` | number | Customer number in `insgen` |
| `db_acc_no` | number | Debit account number raised for the premium |

All nine fields map directly onto the `/print` request body.

### Error response

```json
{
  "status": "E",
  "code": 1001,
  "message": "Invalid customer ID or date range"
}
```

Errors originating in the handler itself carry an `error_type`:

```json
{
  "status": "E",
  "error_type": "PLSQL",
  "message": "ORA-06502: PL/SQL: numeric or value error"
}
```

### Example

```bash
curl -X POST http://172.16.2.160:8889/ords/nic/TRAVEL_COUPON/CreateCoupon \
  -H "Content-Type: application/json" \
  -d '{
        "travel_coupon": {
          "p_id": 406213058,
          "p_birth_dt": "01-01-1990",
          "p_ins_st_dt": "11-02-2026",
          "p_ins_ed_dt": "24-02-2026",
          "p_passport_id": 987654,
          "p_am_can_flag": 1,
          "p_mobile_no": "0599123456",
          "acm_email": "test@test.com",
          "acm_aname": "Ahmed",
          "acm_lname": "Ali"
        }
      }'
```

---

## 3. Print Policy

Renders the `POLICY_TRAVEL_INSURANCE` JasperReports template for an issued coupon.

```
POST /ords/nic/TRAVEL_COUPON/print
```

### Request body

Unlike the other two endpoints, this payload is **flat** — there is no `travel_coupon` wrapper. Feed it the `policy_info` block returned by `/CreateCoupon`.

```json
{
  "branch": 10,
  "office": 1,
  "doc_no": 48249,
  "uw_year": 2026,
  "doc_type": 1,
  "maj_ins_type": 34,
  "min_ins_type": 4
}
```

| Field | Type | Required | Jasper parameter |
|---|---|:---:|---|
| `branch` | number | ✔ | `BRANCH` |
| `doc_no` | number | ✔ | `DOC_NO` |
| `uw_year` | number | ✔ | `DOC_UW_YEAR` |
| `office` | number | | `OFFICE` |
| `doc_type` | number | | `DOC_TYPE` |
| `maj_ins_type` | number | | `MAJ_INS_TYPE` |
| `min_ins_type` | number | | `MIN_INS_TYPE` |

### Responses

This is the only endpoint that sets real HTTP status codes.

| HTTP | Body |
|---|---|
| `200 OK` | `{"status":"S","code":0,"message":"Policy printed successfully"}` |
| `400 Bad Request` | `error_type: JSON_PARSE` — body is not valid JSON |
| `422 Unprocessable Entity` | `error_type: REQUIRED` — `branch`, `doc_no` or `uw_year` missing |
| `500 Internal Server Error` | `error_type: SERVER` — report engine or database failure |

```json
{
  "status": "E",
  "code": 422,
  "error_type": "REQUIRED",
  "message": "Missing required fields",
  "details": null
}
```

### Example

```bash
curl -X POST http://172.16.2.160:8889/ords/nic/TRAVEL_COUPON/print \
  -H "Content-Type: application/json" \
  -d '{
        "branch": 10,
        "office": 1,
        "doc_no": 48249,
        "uw_year": 2026,
        "doc_type": 1,
        "maj_ins_type": 34,
        "min_ins_type": 4
      }'
```

---

## Field Reference

Fields of the `travel_coupon` object, shared by `/calculate_price` and `/CreateCoupon`.

| Field | Type | Required | Example | Description |
|---|---|:---:|---|---|
| `p_id` | number | ✔ | `406213058` | National ID of the insured |
| `p_birth_dt` | string `DD-MM-YYYY` | ✔ | `01-01-1990` | Date of birth — drives the age band used in pricing |
| `p_ins_st_dt` | string `DD-MM-YYYY` | ✔ | `11-02-2026` | Cover start date |
| `p_ins_ed_dt` | string `DD-MM-YYYY` | ✔ | `24-02-2026` | Cover end date — duration drives the premium |
| `p_passport_id` | number | ✔ (create) | `987654` | Passport number |
| `p_am_can_flag` | number | ✔ | `1` | USA/Canada cover flag — `1` = included, `0` = excluded. Materially affects the premium |
| `p_mobile_no` | string | | `0599123456` | Contact mobile |
| `acm_email` | string | | `test@test.com` | Customer email (customer master) |
| `acm_aname` | string | | `Ahmed` | First name (customer master) |
| `acm_lname` | string | | `Ali` | Last name (customer master) |

> Field naming is inconsistent: policy inputs use a `p_` prefix, customer-master fields use an `acm_` prefix. Documented as-is; changing it is a breaking change for existing clients.

---

## Known Issues

Behaviours in the current handlers that clients need to work around, and that are worth fixing server-side.

1. **HTTP status is always 200 on `/calculate_price` and `/CreateCoupon`.** Both handlers write error bodies with `htp.p` without calling `owa_util.status_line`, so an HTTP-status-based client will treat every failure as a success. Clients must branch on the `status` field in the body. `/print` does this correctly and is the model to follow.

2. **`/CreateCoupon`'s error branch is unreachable.** `p_status`, `p_code` and `p_msg` are declared but never assigned, so `IF p_status <> 'S'` evaluates to `NULL` — never `TRUE` — and the block is dead code. Errors surface only if `create_travel_coupon_api_fn` encodes them inside its returned JSON, or via the `WHEN OTHERS` handler. Either populate the OUT parameters or delete the block.

3. **`/print` mixes report output and JSON in one response.** If `JASPERSERVER.CALL_REPORT` streams the rendered report to the HTTP response, the trailing `htp.p(JSON_OBJECT(...))` appends JSON to the end of the report bytes and corrupts it. Decide which contract this endpoint has: return the binary report (correct `Content-Type`, no JSON tail), or return JSON containing a URL/base64 payload.

4. **`/print` never sets a MIME header.** The other two handlers call `owa_util.mime_header('application/json', FALSE)`; this one does not.

5. **Success key casing.** `CreateCoupon` returns `"Status"`; all error paths return `"status"`.

6. **Missing `travel_coupon` wrapper is handled inconsistently.** `/calculate_price` wraps the `get_object` call in a `BEGIN…EXCEPTION` and returns a clean `JSON_PARSE` error; `/CreateCoupon` does not, so the same input falls through to the generic `PLSQL` handler with an opaque `ORA-` message.

7. **`ROLLBACK` in `/calculate_price`.** The exception handler rolls back although the endpoint is read-only. Harmless, but it signals that the pricing call may write — worth confirming.

8. **Hardcoded `branch = 10` / `user_no = 1001` in `/calculate_price`.** Quotes for other branches will price against branch 10. `/CreateCoupon` presumably derives these differently, which risks quote/issue mismatch.

9. **`@TESTDB` database link in the handler source.** Fine for test; the production handler must not carry a test link name.

---

## Open Questions

Points to resolve before this document is final:

- **Authentication** — is the module protected by an ORDS privilege/OAuth2 client, or open on the internal network?
- **`/calculate_price` success payload** — a real sample response is needed.
- **Business error codes** — `1001` is documented as *invalid customer ID or date range*. Is there a full catalogue (age limits, maximum trip duration, overlapping cover, blocked customer)?
- **Which fields does `/calculate_price` actually require?** It shares the wrapper with `/CreateCoupon`, but presumably ignores the customer-master fields.
- **`p_am_can_flag` semantics** — confirm `1` = USA/Canada included.
- **Idempotency** — is a repeated `/CreateCoupon` with the same ID and dates rejected, or does it issue a duplicate coupon?
- **Production base URL.**
