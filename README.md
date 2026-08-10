# NIC General Insurance — Travel Coupon API

**Version:** 1.0
**Base URL:** `https://api.nic-pal.com:8443/ords/nic/`
**Module:** `TRAVEL_COUPON` (pricing, issuance, printing)

---

## 1. Conventions

### Transport

| Item | Value |
|---|---|
| Protocol | HTTPS |
| Method | `POST` (all endpoints) |
| Request `Content-Type` | `application/json` |
| Response `Content-Type` | `application/json` |
| Character set | UTF-8 |
| Cache headers | `Cache-Control: no-cache`, `Pragma: no-cache` |

### Response envelope

Every JSON response carries a `status` discriminator:

| Field | Type | Description |
|---|---|---|
| `status` | string | `"S"` = success, `"E"` = error |
| `code` | number | `0` on success; error code otherwise |
| `message` | string | Human-readable outcome |
| `error_type` | string | Present only when `status = "E"`. See §5 |
| `details` | string | Present only when `status = "E"`; raw `SQLERRM` or diagnostic text |


### Date format

All dates are strings in **`DD-MM-YYYY`** format. Example: `"11-02-2026"` = 11 February 2026.

### Currency

`price` and `excess` are expressed in **USD**.

---

## 2. `POST TRAVEL_COUPON/calculate_price`

Returns the premium and excess (deductible) for a proposed travel policy **without
creating anything**. Safe to call repeatedly — read-only quotation.

**URL:** `https://api.nic-pal.com:8443/ords/nic/TRAVEL_COUPON/calculate_price`

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

### `travel_coupon` object

| Field | Type | Req. | Description |
|---|---|---|---|
| `p_id` | number | Yes | Customer national ID number |
| `p_birth_dt` | string | Yes | Date of birth, `DD-MM-YYYY`. Drives the age band used in pricing |
| `p_ins_st_dt` | string | Yes | Cover start date, `DD-MM-YYYY` |
| `p_ins_ed_dt` | string | Yes | Cover end date, `DD-MM-YYYY`. Must be ≥ start date; the day span drives the rate band |
| `p_passport_id` | number | Yes | Passport number |
| `p_am_can_flag` | number | Yes | Territorial scope. **`2` = USA / Canada included** (higher rate); **`1` = worldwide excluding USA / Canada** |
| `p_mobile_no` | string | Yes | Mobile number, local format (e.g. `"0599123456"`) |
| `acm_email` | string | Yes | Customer e-mail |
| `acm_aname` | string | Yes | First name |
| `acm_lname` | string | Yes | Last / family name |

Server-side metadata (`branch = 10`, `user_no = 1001`) is fixed inside the handler and
is **not** accepted from the caller.

### Success response — `200 OK`

```json
{
    "status": "S",
    "code": 0,
    "message": "Pricing calculated successfully",
    "price": 30,
    "excess": 100
}
```

| Field | Type | Description |
|---|---|---|
| `price` | number | Calculated gross premium, USD |
| `excess` | number | Policy excess / deductible, USD |

---

## 3. `POST TRAVEL_COUPON/CreateCoupon`

Issues the travel coupon: creates the customer (if new), the policy document, and the
debtor account entry. **This endpoint writes to the database.**

**URL:** `https://api.nic-pal.com:8443/ords/nic/TRAVEL_COUPON/CreateCoupon`

### Request body

Identical structure and field semantics to `/calculate_price` (§2). Send the same
payload you priced, so the premium the customer was quoted is the premium booked.

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

### Success response — `200 OK`

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

### `policy_info` object

| Field | Type | Description |
|---|---|---|
| `doc_no` | number | Policy document number — the coupon identifier |
| `branch` | number | Issuing branch code (`10`) |
| `office` | number | Issuing office code (`1`) |
| `uw_year` | number | Underwriting year |
| `doc_type` | number | Document type; `1` = policy |
| `maj_ins_type` | number | Major insurance type; `34` = Travel |
| `min_ins_type` | number | Minor insurance type; `4` = Travel coupon |
| `cust_no` | number | Generated customer number |
| `db_acc_no` | number | Debtor account number raised for the premium |

Store the full seven-key composite (`doc_no`, `branch`, `office`, `uw_year`,
`doc_type`, `maj_ins_type`, `min_ins_type`) — `/print` requires all of it.

This endpoint returns the discriminator as **`"Status"`** (capital S), while the other
endpoints return **`"status"`**. Client parsers must handle both.

### Error response

```json
{
  "status": "E",
  "code": 1001,
  "message": "Invalid customer ID or date range"
}
```

### Idempotency

**Not idempotent.** A repeated call with the same national ID and the same date range
is accepted and issues a **new, separate coupon** with a new `doc_no`. Duplicate
suppression is the caller's responsibility — retry only after a confirmed network
failure, and reconcile by `doc_no` afterwards.

---

## 4. `POST TRAVEL_COUPON/print`

Renders the issued policy through JasperReports Server and returns the document.

**URL:** `https://api.nic-pal.com:8443/ords/nic/TRAVEL_COUPON/print`
**Jasper report:** `POLICY_TRAVEL_INSURANCE`

### Request body

Unlike the other two endpoints, this one takes a **flat** object with no wrapper.
Keys are lowercase and case-sensitive.

```json
{
    "branch": 10,
    "office": 1,
    "doc_no": 48640,
    "uw_year": 2026,
    "doc_type": 1,
    "maj_ins_type": 34,
    "min_ins_type": 4
}
```

| Field | Type | Req. | Maps to Jasper parameter |
|---|---|---|---|
| `branch` | number | **Yes** | `BRANCH` |
| `office` | number | No | `OFFICE` |
| `doc_no` | number | **Yes** | `DOC_NO` |
| `uw_year` | number | **Yes** | `DOC_UW_YEAR` |
| `doc_type` | number | No | `DOC_TYPE` |
| `maj_ins_type` | number | No | `MAJ_INS_TYPE` |
| `min_ins_type` | number | No | `MIN_INS_TYPE` |

All values come straight from the `policy_info` block returned by `/CreateCoupon`.

### Responses

| HTTP | Body | Meaning |
|---|---|---|
| `200 OK` | `{"status":"S","code":0,"message":"Policy printed successfully"}` | Report generated |
| `400 Bad Request` | `error_type: "JSON_PARSE"` | Body is not valid JSON |
| `422 Unprocessable Entity` | `error_type: "REQUIRED"` | `doc_no`, `branch`, or `uw_year` missing or null |
| `500 Internal Server Error` | `error_type: "SERVER"` | Jasper or PL/SQL failure; `details` carries `SQLERRM` |

---

## 5. Error reference

### `error_type` values

| `error_type` | Endpoint | Cause |
|---|---|---|
| `JSON_PARSE` | all | Malformed JSON, or missing `travel_coupon` wrapper object |
| `REQUIRED` | `/print` | `doc_no`, `branch`, or `uw_year` null |
| `TRAVEL_COUPON` | `/CreateCoupon` | Business rule rejection |
| `PLSQL` | `/CreateCoupon` | Unhandled PL/SQL exception; transaction rolled back |
| `SERVER` | `/calculate_price`, `/print` | Unhandled exception; `details` carries `SQLERRM` |

### Known domain codes

| Code | Message | Meaning |
|---|---|---|
| `1001` | Invalid customer ID or date range | `p_id` failed validation, or `p_ins_ed_dt` precedes `p_ins_st_dt` / falls outside the permitted window |

---

## 6. Typical integration flow

```
1. POST TRAVEL_COUPON/calculate_price   → show price + excess to the customer
2. Customer confirms and pays
3. POST TRAVEL_COUPON/CreateCoupon      → persist doc_no + the full key set
4. POST TRAVEL_COUPON/print             → deliver the coupon document
```

Steps 1 and 2 are safe to repeat. Step 3 is not — see the idempotency note in §3.
