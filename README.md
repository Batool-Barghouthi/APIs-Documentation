---
title: NIC APIs Documentation
description: NIC ORDS APIs for Motor Underwriting and Travel Services
---

# NIC APIs Documentation

This documentation describes the NIC ORDS APIs used for Motor Underwriting and Travel Coupon services.

---

## Base URL

http://172.16.2.157:8889


---

## Common Headers

| Header | Value |
|------|------|
| Content-Type | application/json |

---

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
```json
Success Response
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
Motor Underwriting – Calculate Price
Endpoint
POST /ords/nic/motorUw/calculate_price
Description
Calculates insurance price and excess for a vehicle.
Uses the same request body structure as Create Policy.

Success Response
{
  "status": "S",
  "code": 0,
  "message": "Policy created successfully",
  "price": "1240",
  "excess": "3000"
}
Error Response
{
  "status": "E",
  "code": 7,
  "message": "This Vehicle is insured"
}
Motor Underwriting – Error Codes
Endpoint
GET /ords/nic/lockup/error_code
Query Parameters
Name	Type	Required
p_code	number	Yes
Success Response
{
  "items": [
    {
      "error_ar": "تاريخ إنتهاء التأمين يجب أن يكون أكبر من تاريخ بدء التأمين",
      "error_en": "Insurance End Date Must Be Greater Than Inception Date",
      "code": 1
    }
  ]
}
System Lookups
Endpoint
GET /ords/nic/lockup/getdata
Query Parameters
Name	Type	Required
lockup_code	number	Yes
Get Vehicle Information
Endpoint
GET /ords/nic/motorUw/vehicle
Query Parameters
Name	Type	Required
plate_no	string	Yes
Success Response
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
Error Response
{
  "status": "E",
  "message": "Vehicle not found"
}
Travel – Insert Travel Coupon
Endpoint
POST /ords/nic/create_travel/insert_travel_coupon
Description
Creates a travel coupon for a customer.

Request Body
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
Success Response
{
  "status": "S",
  "code": 0,
  "message": "Travel coupon inserted successfully",
  "coupon_no": "TRV123456"
}
Error Response
{
  "status": "E",
  "code": 1001,
  "message": "Invalid customer ID or date range"
}
