---
title: NIC APIs Documentation
description: NIC ORDS APIs for Motor Underwriting and Travel Services
---

# NIC APIs Documentation

This documentation describes the NIC ORDS APIs used for Motor Underwriting and Travel Coupon services.

---

## Base URL

http://172.16.2.160:8889


---

## Common Headers

| Header | Value |
|------|------|
| Content-Type | application/json |

---
## Form Builder API 

### Endpoint

GET /ords/nic/docs/get_api_info

### Description

Returns all metadata required to dynamically build a user form.  
Each response item represents a single form field and includes:

- Field name and label
- Data type
- Whether the field is required
- Lookup (List of Values) information
- Parent–child dependency (if applicable)

### Query Parameters
Query Params
Name	Type	Required
p_code	number	Yes
```json

```
Success Response
```json
   {
            "code": 271,
            "field_name": "SCO_MO_SP_USE",
            "label": "الاستعمال الخاص",
            "data_type": "NUMBER",
            "required": 1,
            "list_code": null,
            "remarks_type": 2,
            "remarks": "lockup/get_sp_use",
            "parent_code": null
        }
```



## Motor Underwriting – Create Policy

### Endpoint

POST /ords/nic/motorUw/create_policy


### Description

Creates a new Motor Underwriting policy including:
- KYC (customer data)
- Vehicle details
- Drivers list  
Triggers the underwriting workflow automatically.

---

### Request Body

```json
{
  "kyc": {
    "CUSTOMER_ID": 785546541,
    "ACM_ANAME": "Ahmed BARG TT TT",
    "ACM_LNAME": "Ali BARG",
    "ACM_MOBILE_NO": "0599123456",
    "ACM_EMAIL": "ahmed@test.com",
    "ACM_ADDRESS": "Ramallah",
    "CUST_NATIONALITY": 275,
    "CUST_BIRTH_DATE": "01-01-1990",
    "CUST_GENDER": 1,
    "CUST_MOBILE": "0599123456"
  },
  "policy": {
    "MST_INS_ST_DT": "29-01-2026",
    "MST_INS_ED_DT": "28-01-2027",
    "MST_MIN_INS_TYPE": 3,
    "SCO_MO_MAKE": 1,
    "SCO_LOCATION_GEO_AREA": 1,
    "SCO_MO_PRICE_LOC": 1,
    "SCO_MO_PLATE_NO": "P7711245645",
    "SCO_MO_CHAS_NO": "45FGDH457",
    "SCO_FSUM_INSURED": 35000,
    "SCO_MO_PLATE_TYPE": 1,
    "SCO_MO_COLOR_CD": 2,
    "SCO_MO_MODEL": 1,
    "SCO_MO_SEATS": 5,
    "SCO_MO_PROD_YEAR": 2020,
    "SCO_MO_ENG_SIZE": 2000,
    "SCO_MO_SP_USE": 1,
    "SCO_MO_ALLOWED_PERSONS": 5,
    "SCO_MO_LOAD": 1,
    "GSD_SYMBOL": 12354,
    "drivers": [
      {
        "name_ar": "محمد أحمد",
        "name_en": "Mohammad Ahmad",
        "dt": "20-09-2000",
        "id": "123456789",
        "rsn": 1
      }
    ]
  }
}
```

## KYC Object Fields

| Field Name | Type | Required | Description |
|-----------|------|----------|-------------|
| CUSTOMER_ID | Number | Yes | National ID / Customer identifier |
| ACM_ANAME | String | Yes | Customer Arabic full name |
| ACM_LNAME | String | Yes | Customer English last name |
| ACM_MOBILE_NO | String | Yes | Mobile number |
| ACM_EMAIL | String | Yes | Email address |
| ACM_ADDRESS | String | Yes | Address |
| CUST_NATIONALITY | Number | Yes | Nationality code (e.g. 708) |
| CUST_BIRTH_DATE | String (DD-MM-YYYY) | Yes | Date of birth |
| CUST_GENDER | Number | Yes | 1 = Male, 2 = Female |
| CUST_MOBILE | String | Yes | Customer mobile number |

---

## Policy Object Fields

### Policy Information

| Field Name | Type | Required | Description |
|-----------|------|----------|-------------|
| MST_INS_ST_DT | String (DD-MM-YYYY) | Yes | Insurance start date |
| MST_INS_ED_DT | String (DD-MM-YYYY) | Yes | Insurance end date |
| MST_MIN_INS_TYPE | Number | Yes | Insurance type code |

---

### Vehicle Information

| Field Name | Type | Required | Description |
|-----------|------|----------|-------------|
| SCO_MO_MAKE | Number | Yes | Vehicle make code |
| SCO_MO_MODEL | Number | Yes | Vehicle model code |
| SCO_MO_PROD_YEAR | Number | Yes | Production year |
| SCO_MO_COLOR_CD | Number | Yes | Vehicle color code |
| SCO_MO_SEATS | Number | Yes | Number of seats |
| SCO_MO_ENG_SIZE | Number | Yes | Engine capacity (CC) |
| SCO_MO_CHAS_NO | String | Yes | Chassis number |
| SCO_MO_PLATE_NO | String | Yes | Plate number |
| SCO_MO_PLATE_TYPE | Number | Yes | Plate type |
| SCO_LOCATION_GEO_AREA | Number | Yes | Geographic area |
| SCO_MO_PRICE_LOC | Number | Yes | Location code |
| SCO_FSUM_INSURED | Number | Yes / Conditional | Vehicle price (depends on insurance type) |
| SCO_MO_SP_USE | Number | Yes | Special use code |
| SCO_MO_ALLOWED_PERSONS | Number | Yes | Allowed passengers |
| SCO_MO_LOAD | Number | Yes / Conditional | Manual value |
| GSD_SYMBOL | Number | Yes / Conditional | Policy symbol |

> **Note**  
> Fields marked as **Yes / Conditional** are required depending on the selected insurance type.

Success Response
```json
{
  "Status": "S",
  "code": 0,
  "message": "Motor policy created successfully",
  "workflow_req_id": 966439,
  "policy_info": {
    "doc_no": 900008,
    "branch": 10,
    "office": 10,
    "uw_year": 2026,
    "doc_type": 1,
    "maj_ins_type": 33,
    "min_ins_type": 3,
    "cust_no": 8406213058,
    "db_acc_no": 113220025612
  }
}
```
 Error Response
```json

{
  "status": "E",
  "code": 1001,
  "message": "CUSTOMER_ID is required"
}
```



## Motor Underwriting – Calculate Price
### Endpoint
POST /ords/nic/motorUw/calculate_price

## Description
Calculates insurance price and excess for a vehicle.
Uses the same request body structure as Create Policy.

Success Response
```json
{
  "status": "S",
  "code": 0,
  "message": "Policy price calculated successfully",
  "price": "1240",
  "excess": "3000"
}
```
Error Response
```json
{
  "status": "E",
  "code": 210,
  "message": "SCO_MO_PLATE_TYPE is required"
}
```

## Motor Underwriting – Error Codes

### Endpoint

GET /ords/nic/lockup/error_code

Query Parameters
Name	Type	Required
p_code	number	Yes
Success Response
```json
{
    "items": [
        {
            "name_ar": "تاريخ إنتهاء التأمين يجب أن يكون أكبر من تاريخ بدء التأمين",
            "name_en": "Insurance End Date Must Be Greater Than Inception Date",
            "code": 1
        },
        {
            "name_ar": "المؤمن ضمن القائمة السوداء يرجى مراجعة دائرة الاكتتاب",
            "name_en": "The customer is on the company’s blacklist.",
            "code": 2
        },
        {
            "name_ar": "تم تجاوز سقف الذمم المسموح يرجى مراجعة المحاسبة",
            "name_en": "The allowed credit ceiling has been exceeded. Please contact the accounting department.",
            "code": 3
        },
        {
            "name_ar": "مشكلة في حجز الشهادة للبوليصة",
            "name_en": "A problem occured in Reserve Certification",
            "code": 4
        },
        {
            "name_ar": "رقم الشاصي لا يحتوي سوى حروف وأرقام",
            "name_en": "Chasi No only contains Alphabetic and numbers",
            "code": 5
        }
        ]
   }
```
## System Lookups
### Endpoint

GET /ords/nic/lockup/getdata
### Query Parameters
Name	Type	Required
lockup_code	number	Yes
for example list of insurance types for item MST_MIN_INS_TYPE is lockup_code = 233  
Success Response
```json
{
    "items": [
        {
            "name_ar": "ACT",
            "name_en": "ACT",
            "code": 1
        },
        {
            "name_ar": "ACT+TP",
            "name_en": "ACT+TP",
            "code": 3
        },
        {
            "name_ar": "COMP+TP",
            "name_en": "COMP+TP",
            "code": 5
        },
        {
            "name_ar": "ACT+TP+COMP",
            "name_en": "ACT+TP+COMP",
            "code": 6
        },
        {
            "name_ar": "VIP II",
            "name_en": "VIP II",
            "code": 7
        },
        {
            "name_ar": "TP",
            "name_en": "TP",
            "code": 8
        },
        {
            "name_ar": "COMP",
            "name_en": "COMP",
            "code": 9
        },
        {
            "name_ar": "P.V",
            "name_en": "P.V",
            "code": 11
        }
    ]
}

```

## Get Vehicle Information
Endpoint
GET /ords/nic/motorUw/vehicle
### Query Parameters
Name	Type	Required
plate_no	string	Yes
Success Response
```json
{
  "status": "S",
  "vehicle_info": {
    "PRODUCTION_YEAR": "2023",
    "CHASSIS_NO": "65767768",
    "ENGINE_SIZE": "1591",
    "TOTAL_SEATS": "4",
    "COLOR_NAME_AR": "احمر"
  }
}




```
## Vehicle Field Mapping

| Field | Mapping System Field |
|------|----------------------|
| LICENSE_NO | NONE |
| PRODUCTION_YEAR | SCO_MO_PROD_YEAR |
| CARLICENSE_EXPIRY_DATE | NONE |
| REGISTRY_DATE | NONE |
| OWNER_NAME_AR | NONE |
| OWNER_CODE | NONE |
| VEHICLE_NO | NONE |
| VEHICLE_TYPE_NAME_AR | SCO_MO_PLATE_TYPE |
| CHASSIS_NO | SCO_MO_CHAS_NO |
| VEHICLE_ORIGIN_NAME_AR | NONE |
| REGISTRY_TYPE_NAME_AR | NONE |
| ENGINE_FUEL_TYPE_NAME_AR | NONE |
| ENGINE_NO | SCO_MO_ENG_NO |
| ENGINE_SIZE | SCO_MO_ENG_SIZE |
| ENGINE_TYPE | NONE |
| TOTAL_WEIGHT | SCO_MO_LOAD |
| MANUFACTURER_NAME_AR | SCO_MO_MAKE |
| ENGINE_MANUFACTURER_NAME_AR | NONE |
| FORCE_DISTRIBUTION_NAME_AR | NONE |
| BRAND_NAME | GSD_VEHICLE_TYPE |
| COLOR_NAME_AR | SCO_MO_COLOR_CD |
| TRUCK_HOOK_P | NONE |
| SEATS_NEAR_DRIVER | NONE |
| TOTAL_SEATS | SCO_MO_SEATS |
| FRONT_WHEELS | NONE |
| BACK_WHEELS | NONE |

Error Response
```json
{
  "status": "E",
  "message": "No response returned from API"
}
```

## General Insurance  – Insert Travel Coupon

### Endpoint

POST /ords/nic/TRAVEL_COUPON/CreateCoupon

Description
Creates a travel coupon for a customer.

Request Body
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
Success Response
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
Error Response
```json
{
  "status": "E",
  "code": 1001,
  "message": "Invalid customer ID or date range"
}
```








































