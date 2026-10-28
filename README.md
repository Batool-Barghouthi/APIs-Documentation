## Motor Vehicle Policy Lookup API
### البحث عن وثائق التأمين حسب رقم اللوحة أو رقم الشاصي

Returns motor policies (major type 33) that match a vehicle plate number or chassis number. Only the **latest endorsement** of each policy is returned.

**Method:** `GET`
**Endpoint:** `https://api.nic-pal.com:8443/ords/nic/polices/by-vehicle`

### Query Parameters

| Parameter  | Type   | Required    | Description |
|------------|--------|-------------|-------------|
| `plate_no` | string | Conditional | Vehicle plate number. Partial match. Case, `-`, spaces and `.` are ignored (`9-175-098` = `9175098`). |
| `chas_no`  | string | Conditional | Chassis (VIN) number. Partial match, case-insensitive. |
| `limit`    | number | No          | Records per page. Default `25`. |
| `offset`   | number | No          | Number of records to skip. Default `0`. |

> At least one of `plate_no` or `chas_no` is required. If both are sent, a policy must match both. If neither is sent, `items` is empty.

### Example Requests

```
GET https://api.nic-pal.com:8443/ords/nic/polices/by-vehicle?plate_no=9175098
GET https://api.nic-pal.com:8443/ords/nic/polices/by-vehicle?chas_no=WVWZZZ1KZ
GET https://api.nic-pal.com:8443/ords/nic/polices/by-vehicle?plate_no=9175098&limit=10&offset=10
```

```bash
curl "https://api.nic-pal.com:8443/ords/nic/polices/by-vehicle?plate_no=9175098"
```

### Success Response — `200 OK`

```json
{
  "items": [
    {
      "pol_creater": "أحمد محمد",
      "cust_no": 100245,
      "common_pol_no": "33-01-2026-000123",
      "doc_no": 5521,
      "uw_year": 2026,
      "maj_ins_type": 33,
      "min_ins_type": "1-تأمين إلزامي",
      "pol_no": 123,
      "cer_no": "778899",
      "ins_start_date": "2026-01-01",
      "ins_end_date": "2026-12-31",
      "reg_date": "2026-01-01",
      "agent_no": 45,
      "db_acc_no": "1100250",
      "through": 2,
      "pkg_no": 3,
      "plate_no": "9175098",
      "chassis_no": "WVWZZZ1KZ6W000001"
    }
  ],
  "hasMore": false,
  "limit": 25,
  "offset": 0,
  "count": 1,
  "links": [
    { "rel": "self", "href": "https://api.nic-pal.com:8443/ords/nic/polices/by-vehicle?plate_no=9175098" }
  ]
}
```

### Response Fields

| Field            | Description |
|------------------|-------------|
| `pol_creater`    | Name of the user who issued the policy (Arabic name if available) |
| `cust_no`        | Customer number |
| `common_pol_no`  | Customer id number |
| `doc_no`         | Policy document number |
| `uw_year`        | Underwriting year |
| `min_ins_type`   | Minor insurance type as `code-description` (e.g. `1-تأمين إلزامي`) |
| `pol_no`         | Policy number |
| `cer_no`         | certificate number |
| `ins_start_date` | Coverage start date (`YYYY-MM-DD`) |
| `ins_end_date`   | Coverage end date (`YYYY-MM-DD`) |
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
| `400 Bad Request` | Invalid parameter value |
| `401 Unauthorized` | Missing or invalid token (if OAuth is enabled) |
| `404 Not Found`    | Wrong URL |
| `500 Internal Server Error` | Database or link error. Contact NIC IT. |

> No matches returns `200` with an empty `items` array, not `404`.

### Notes

- Searches are partial (contains) matches. Send as much of the plate or chassis as you have, to keep results small and fast.
- A vehicle insured on several policies (renewals, transfers) returns one row per policy.
- Dates use `YYYY-MM-DD`. Name fields may contain Arabic text (UTF-8).
