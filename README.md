## Get Policy Attachment API

### Endpoint

GET /ords/nic/polices/ATTACHMENT

---

### Description

Retrieves a policy attachment and returns its contents as a **Base64-encoded string**.

The API is used to download documents or images associated with a policy, such as policy copies, licenses, claim attachments, or other archived files.

The returned Base64 content can be decoded to reconstruct the original file.

---

### Headers

| Header | Value |
|--------|-------|
| Accept | text/plain |

---

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| p_file_name | String | Yes | Name of the attachment file. |
| p_directory_name | String | Yes | Directory containing the attachment. |
| p_img_doc_type | Number | Yes | Policy document type. |
| p_img_doc_min_ins | Number | Yes | Minor insurance type. |
| p_img_doc_maj_ins | Number | Yes | Major insurance type. |
| p_img_doc_uw_year | Number | Yes | Policy underwriting year. |
| p_img_doc_no | Number | Yes | Policy document number. |
| p_img_doc_office | Number | Yes | Policy office code. |
| p_img_doc_branch | Number | Yes | Policy branch code. |
| p_img_type | Number | Yes | Attachment type code. |
| p_user_number_aims | Number | Yes | User number requesting the attachment. |
| p_img_file_type | String | Yes | Attachment file type (e.g. `pdf`, `png`, `jpg`). |
| p_img_code | Number | Yes | Attachment image code. |
| p_mst_pol_year | Number | Yes | Policy year. |
| p_mst_pol_no | Number | Yes | Policy number. |
| p_img_branch | Number | Yes | Attachment branch code. |

---

### Example Request

```http
GET {{url}}/ords/nic/polices/ATTACHMENT?p_file_name=bat.png&p_directory_name=shared_dir&p_img_doc_type=1&p_img_doc_min_ins=9&p_img_doc_maj_ins=33&p_img_doc_uw_year=2026&p_img_doc_no=700003&p_img_doc_office=10&p_img_doc_branch=10&p_img_type=9000&p_user_number_aims=1029&p_img_file_type=png&p_img_code=9001&p_mst_pol_year=2026&p_mst_pol_no=700003&p_img_branch=10
```

---

### Success Response

**Content-Type**

```text
text/plain
```

**Response Body**

```text
iVBORw0KGgoAAAANSUhEUgAA...
```

The response body contains the **Base64-encoded contents** of the requested attachment.

---

### Error Response

```json
{
    "status": "E",
    "message": "Attachment not found."
}
```

or

```json
{
    "status": "E",
    "message": "Invalid attachment information."
}
```

---

### Notes

- The API returns the attachment as a **Base64-encoded string**.
- The response **Content-Type** is **`text/plain`**.
- Decode the returned Base64 string to reconstruct the original file.
- Supported attachment types include PDF, PNG, JPG, JPEG, and other archived document formats.
- All query parameters are required to uniquely identify the requested attachment.
