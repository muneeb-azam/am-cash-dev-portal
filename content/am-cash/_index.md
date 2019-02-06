+++
title = "AM Cash Developer Portal"
menuTitle = "Home"
weight = 10
+++
---

This document outlines the AIRMILES Cash REST API platform.

## Dev Environments

| Environment | URL                                     |
|:------------|:----------------------------------------|
| Sandbox     | <https://sandbox.api.loyalty.com/v1/>   |
| Production  | <https://api.loyalty.com/v1/>           |

## Primary API Modules

{{% notice info %}}
There are 4 main API modules available on the AIRMILES Cash platform.
{{% /notice %}}

- Echo
- AM Cash Inquiry
- AM Cash Redemption
- Reversal (Void)**

AM Cash API returns RESTful HTTP status codes, but also retains `responseCode` parameter in the response body
for backwards compatibility purposes.
The `message` parameter in the response body provides additional detail information.

Under each API description, a mapping of HTTP status codes and historic AM Cash POS response codes will be provided.

### Authentication
Before AM Cash APIs can be called, the partner application must first authorize using `/v1/authorize`.
Once the partner application has completed the authorization process, a token exchange can be done using
`/v1/oauth/token`.

Once a Bearer token has been obtained, it must be included in the ```Authorization``` header for every request.

## Echo
The Echo API can be used to confirm that AM Cash is available and ready to process requests.

#### Request
```bash
curl -X POST \
  http://sandbox.api.loyalty.com/v1/partners/me/amcash/echo \
  -H 'Authorization: Bearer eyJ0e...iRQ' \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: application/json' \
  -d '{
    "traceId" : "1234567890123",
    "sponsorCode":"ANON",
    "storeId": "0001",
    "deviceId": "001",
    "vendorId": "00000000"
}'
```

#### Response
```bash
Status: 200 OK
Body:
{
    "transDateTime": "2018-10-09T10:11:17",
    "traceId": "1234567890123",
    "responseCode": "0000",
    "sponsorCode": "ANON",
    "storeId": "0001",
    "deviceId": "001",
    "vendorId": "00000000",
    "message": "Approved"
}
```

#### Status Codes & Response Codes

|HTTP status code | POS response code | Description |
|:----------------|:------------------|:------------|
|200|0000|AM Cash is up and running. |
|400|0035| Request sent contains invalid data. |
|408|0008|Request timed out. |
|500|0096| Server error. |
|503|0092| Scheduled outage. |

## AM Cash Inquiry

## AM Cash Redemption

## Reversal (Void)
