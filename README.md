## Motor Policy Search by Plate / Chassis

Returns motor policies (major type 33) that match a vehicle plate number or chassis number. Only the **latest endorsement** of each policy is returned.

**Method:** `GET`
**URL:** `{base_url}/ords/insproc/motor/v1/policies`
**Auth:** `Authorization: Bearer {access_token}` (OAuth2 client credentials)

### Query Parameters

| Parameter  | Type   | Required    | Description |
|------------|--------|-------------|-------------|
| `plate_no` | string | Conditional | Vehicle plate number. Partial match. Case, `-`, spaces and `.` are ignored (`12-345-67` = `1234567`). |
| `chas_no`  | string | Conditional | Chassis (VIN) number. Partial match, case-insensitive. |
| `limit`    | number | No          | Records per page. Default `25`. |
| `offset`   | number | No          | Number of records to skip. Default `0`. |

> At least one of `plate_no` or `chas_no` is required. If both are sent, a policy must match both. If neither is sent, `items` is empty.

### Example Requests

```
GET /ords/insproc/motor/v1/policies?plate_no=12-345-67
GET /ords/insproc/motor/v1/policies?chas_no=WVWZZZ1KZ
GET /ords/insproc/motor/v1/policies?plate_no=1234567&limit=10&offset=10
```

```bash
curl -H "Authorization: Bearer {access_token}" \
     "{base_url}/ords/insproc/motor/v1/policies?plate_no=12-345-67"
```

### Success Response — `200 OK`

```json
{
  "items": [
    {
      "pol_creater": "أحمد محمد",
      "cust_no": 100245,
      "common_pol_no": "33-01-2026-000123",
      "branch": 1,
      "office": 1,
      "doc_no": 5521,
      "uw_year": 2026,
      "doc_type": 1,
      "maj_ins_type": 33,
      "min_ins_type": 1,
      "pol_no": 123,
      "cer_no": "778899",
      "ins_start_date": "2026-01-01",
      "ins_end_date": "2026-12-31",
      "reg_date": "2026-01-01",
      "agent_no": 45,
      "db_acc_no": "1100250",
      "through": 2,
      "pkg_no": 3,
      "plate_no": "12-345-67",
      "chassis_no": "WVWZZZ1KZ6W000001"
    }
  ],
  "hasMore": false,
  "limit": 25,
  "offset": 0,
  "count": 1,
  "links": [ { "rel": "self", "href": "..." } ]
}
```

### Response Fields

| Field            | Description |
|------------------|-------------|
| `pol_creater`    | Name of the user who issued the policy (Arabic name if available) |
| `cust_no`        | Customer number |
| `common_pol_no`  | Common (display) policy number |
| `branch` / `office` | Issuing branch / office |
| `doc_no`, `doc_type`, `uw_year` | Policy document key |
| `maj_ins_type` / `min_ins_type` | Major (always 33 = Motor) / minor insurance type |
| `pol_no`         | Policy number |
| `cer_no`         | Motor card / certificate number |
| `ins_start_date` / `ins_end_date` | Coverage period (`YYYY-MM-DD`) |
| `reg_date`       | Registration date (`YYYY-MM-DD`) |
| `agent_no`       | Agent number |
| `db_acc_no`      | Debit account number |
| `through`        | Business channel code |
| `pkg_no`         | Package / office number |
| `plate_no`       | Plate number as stored |
| `chassis_no`     | Chassis number as stored |

### Paging

Results are sorted newest first (`reg_date` DESC). If `hasMore` is `true`, request the next page with `offset = offset + limit`, or follow the `next` link in `links`.

### Error Responses

| Status | Meaning |
|--------|---------|
| `401 Unauthorized` | Missing, invalid or expired token |
| `403 Forbidden`    | Client is not granted access to this API |
| `404 Not Found`    | Wrong URL |
| `500 Internal Server Error` | Database error. Contact NIC IT. |

> No matches returns `200` with an empty `items` array, not `404`.

### Notes

- Searches are partial (contains) matches. Send as much of the plate or chassis as you have, to keep results small and fast.
- A vehicle insured on several policies (renewals, transfers) returns one row per policy.
- Dates use `YYYY-MM-DD`. Name fields may contain Arabic text (UTF-8).
