# Signup API

**:boom:This API is currently in the request-for-comment stage and is not available in either test or production!**  
https://github.com/vippsas/vipps-developers/blob/master/contact.md

## Create merchant
### Request
`POST` `/signup/psp`
```json
{
    "organizationNumber": "123456789",
    "name": "Some Merchant AS",
    "logo": "base64encoded"
}
```

### Response
`201 Created`
```json
{
    "organizationNumber": "123456789",
    "name": "Some Merchant AS",
    "logo": "base64encoded",
    "serialNumber": "987654321",
    "active": true
}
```

## Get all merchants
### Request
`GET` `/signup/psp`

### Response
`200 OK`
```json
[
    {
        "organizationNumber": "123456789",
        "name": "Some Merchant AS",
        "logo": "base64encoded",
        "serialNumber": "987654321",
        "active": true
    }
]
```

## Get merchant
### Request
`GET` `/signup/psp/123456789`

### Response
`200 OK`
```json
{
    "organizationNumber": "123456789",
    "name": "Some Merchant AS",
    "logo": "base64encoded",
    "serialNumber": "987654321",
    "active": true
}
```

## Update merchant
### Request
`PATCH` `/signup/psp/123456789`
```json
{
    "name": "New Merchant Name AS",
    "logo": "newLogoBase64"
}
```

### Response
`200 OK`
```json
{
    "organizationNumber": "123456789",
    "name": "New Merchant Name AS",
    "logo": "newLogoBase64",
    "serialNumber": "987654321",
    "active": true
}
```

## Disable merchant
### Request
`PATCH` `/signup/psp/123456789`
```json
{
    "active": false
}
```

### Response
`200 OK`
```json
{
    "organizationNumber": "123456789",
    "name": "New Merchant Name AS",
    "logo": "newLogoBase64",
    "serialNumber": "987654321",
    "active": false
}
```
