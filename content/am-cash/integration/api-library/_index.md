+++
title = "AM Cash API Library"
pre = "4. "
weight = 40
+++
---

There are five endpoints available for AM Cash POS API. The first endpoint is the Authorization endpoint. There are four types of transactions that are interchanged between the Partner Platform and the Instant POS Platform. These service endpoints are: 

* Authorization
* Echo Test
* Inquiry
* Redemption 
* Reversal (VOID) 

This section describes the four API endpoints, including their purpose, request and response format and return codes. Each of the APIs is explained below in its own section. Each section contains sample request and responses, parameter descriptions and return codes. Suggestions on how to use each of these messages are also provided. 

## Handling Request and Response Messages  

As described above, the RESTful specification defines a transaction as a pair of request and response messages that are used for information interchange. 

Request message must include all elements that are marked as mandatory. Elements that are not mandatory are optional. Elements that are mandatory will be marked with a red asterisk. For example: collectNumber* 

Response messages also indicate which elements are always returned. Upon receiving a response message the POS Terminal or Partner Platform, the calling function should verify the Response Code (‘responseCode’) and/or the HTTP Status, because if the response code is an error condition (i.e. the transaction was not approved) not all elements may be returned. Data elements that contain null or empty values are excluded from the response body. The POS Terminal may use the response elements to display information and print a receipt (if applicable) according to the guidelines provided in the following sections.  

## Authorization  

Authorization is required in order to access AM Cash. Authorization is accomplished via OAuth2 authentication process. For partners integrating with LoyaltyOne's authorization utilizing OAuth 2.0, the appropriate flow and documentation will depend on the context and implementation of your application.  

*Please refer to the instructions below to determine which guide to follow:*  

**Scenario A.**  
You are relying on the collector to provide authentication to LoyaltyOne's auth server (i.e., the user will log into our service) in order to obtain authorization to a resource, use one of the following flows:  

   * If you are creating a static web application WITHOUT a backend service, use the Implicit Grant Flow.  
   
   * If you are creating a web application WITH a backend service (ie regular web app), use the Authorization Code Flow.  
   
   * If you are creating a native mobile application, use the Authorization Code Flow with PKCE.  

**Scenario B.**  
Your application is acting as a machine to machine and/or proxy service to access a resource, use the Client Credentials Grant Flow. 

{{% notice note %}}
Credentials for obtaining access will be provided by LoyaltyOne. These credentials MUST NOT be used or made available in public facing entities.  
Your application may serve as a backend service provider to proxy resources to a public client (webpage or mobile app) if required.
{{% /notice %}}

Once the authorization process is complete, the partner will obtain a valid Bearer Token. This Bearer Token must be included in the Authorization header of each API request.

*Please visit [https://github.com/LoyaltyOne/api-references#api-references](https://github.com/LoyaltyOne/api-references#api-references) for more information.*

## Echo Test  

{{% notice note %}}
ECHO is used to test and confirm that AM Cash is up and ready to process requests.  
{{% /notice %}}

### POST /v1/partners/me/amcash/echo

Authorize using the provided `client_id` & `client_secret` to receive a 'Bearer Token'.

**Sample cURL:**
```curl
curl -X POST \
  https://sandbox.api.loyalty.com/v1/partners/me/amcash/echo \
  -H 'Authorization: Bearer eyJ0e…6sA' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
    "traceId" : "999999999999",
    "sponsorCode":"XXXX",
    "storeId": "####",
    "deviceId": "###",
    "vendorId": "######"
}'
``` 

**Sample success response:**
```json
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Mon, 29 Oct 2018 15:27:34 GMT
{
    "transDateTime": "2018-10-24T03:59:03",
    "traceId": "############",
    "responseCode": "0000",
    "sponsorCode": "XXXX",
    "storeId": "####",
    "deviceId": "###",
    "vendorId": "######",
    "message": "Approved"
}
```

### Echo Request Parameters

| Parameter      | Value                        | Description                                                  | Parameter Type | Data Type                                  |
|----------------|------------------------------|--------------------------------------------------------------|----------------|--------------------------------------------|
| Content-Type*  | application/json             |                                                              | header         | String                                     |
| Authorization* | <<Bearer Token>>             | The Bearer token obtained from   the Authorization endpoint. | header         | String                                     |
| traceId*       | 12345678                     | System Trace Number.                                         | body           | String,   length 12 digits.                |
| sponsorCode*   | <<provided by   LoyaltyOne>> | The code used to identify the   partner.                     | body           | String, length 4.                          |
| storeId*       | 1                            | The Store Id.                                                | body           | String,   length 4 digits.                 |
| deviceId*      | 1                            | The Device Id.                                               | body           | String, length 3 digits.                   |
| vendorId*      | <<provided by LoyaltyOne>>   | The vendor Id used to identify the partner.                  | body           | String,   variable length up to 35 digits. |