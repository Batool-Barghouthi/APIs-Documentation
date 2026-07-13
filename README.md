## Upload Policy Attachment API

### Endpoint
POST /ords/nic/BASE64/UPLOAD

---

### Description
Uploads a policy attachment supplied as a **Base64-encoded string** and stores it as a file on the server.
The API is used to archive documents or images associated with a policy, such as policy copies, licenses, claim attachments, or other archived files.
The request body is the Base64 content of the file; the target directory and file name are supplied as request headers.

---

### Headers
| Header | Value | Required | Description |
|--------|-------|----------|-------------|
| Content-Type | text/plain | Yes | The request body is Base64 text. |
| p_directory_name | String | Yes | Oracle DIRECTORY object where the file will be written. |
| p_file_name | String | Yes | Name to store the uploaded file under. |
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
| p_img_file_type | String | Yes | Attachment file type (e.g. pdf, png, jpg). |
| p_img_code | Number | Yes | Attachment image code. |
| p_mst_pol_year | Number | Yes | Policy year. |
| p_mst_pol_no | Number | Yes | Policy number. |
| p_img_branch | Number | Yes | Attachment branch code. |


**Content-Type**
```text
text/plain
```

---

### Request Body
| Body | Type | Required | Description |
|------|------|----------|-------------|
| body | String (Base64) | Yes | Base64-encoded contents of the file to upload. |

**Request Body**
```text
iVBORw0KGgoAAAANSUhEUgAA...
```

---

### Example Request
```http
POST {{url}}BASE64/UPLOAD
Content-Type: text/plain
p_directory_name: shared_dir
p_file_name: bat.png

iVBORw0KGgoAAAANSUhEUgAA...
```

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

### Notes
- The request body must be a valid **Base64-encoded string**; it is decoded server-side and written to the target directory as a binary file.
- The target location is defined by the **`p_directory_name`** header, which must be a valid Oracle **DIRECTORY** object with write access granted to the ORDS parsing schema.
- The **`p_file_name`** header sets the stored file name.
- The response **Content-Type** is **`application/json`**.
- A `status` of `"S"` and `code` of `0` indicate success; a `status` of `"E"` indicates failure, with `code` carrying the HTTP status or Oracle error code.
- Supported attachment types include PDF, PNG, JPG, JPEG, and other archived document formats.
