+++
title = "AM Cash API Library"
pre = "3. "
weight = 30
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

---

## Echo Test  

{{% notice info %}}
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

### Echo Response Messages

| HTTP Status Code | responseCode Value | Reason                                               |
|------------------|--------------------|------------------------------------------------------|
| 200              | 0                  | OK                                                   |
| 400              | 35                 | Bad request – missing, invalid parameters or values. |
| 401              | N\A                | Missing or   invalid Bearer token.                   |
| 408              | 8                  | Transaction timed out.                               |
| 500              | 96                 | Server   error                                       |
| 503              | 92                 | Scheduled outage                                     |

### Echo Response Model

| Field         | Description                                                                                            | Data Type                                |
|---------------|--------------------------------------------------------------------------------------------------------|------------------------------------------|
| responseCode  | The Response Code indicates the result of the   transaction.                                           | String,   length 4 digits.               |
| message       | Message that describes the status of the response.                                                     | String                                   |
| traceId       | The   traceId provided in the request.                                                                 | String,   length 12 digits.              |
| transDateTime | The transmission date and time from the Instant POS Platform using   ISO-8061 from AM Cash to partner. | String, length 20.                       |
| sponsorCode   | The   sponsorCode provided in the request.                                                             | String,   length 4.                      |
| vendorId      | The vendorId provided in the request.                                                                  | String, variable length up to 35 digits. |
| storeId       | The   storeId provided in the request.                                                                 | String,   length 4 digits.               |
| deviceId      | The deviceId provided in the request.                                                                  | String, length 3 digits.                 |

{{% notice tip %}}  
RESPONSES will always include responseCode and message fields.  
--and--  
SUCCESSES will always reflect body parameters sent in the request body.  
{{% /notice %}}

### Using Echo Test

The Echo Test request can be used as a means to check that the Instant POS Platform is up and running and there are no connectivity issues between the Partner Platform or POS Terminal and the Instant POS Platform. Its usage is similar to the well-known ping command: used to ensure one system can contact another system successfully.

It is encouraged the Echo Test request is used by the Partner Platform upon starting to ensure both systems communicate properly. Also, at the Partner’s discretion, periodic Echo Test requests could be sent during partner business hours to make sure the systems can communicate and that the Instant POS Platform is ready for processing. 

As mentioned above, the Echo Test message is used to test connectivity from the Partner Platform or POS Terminal to the Instant POS Platform, therefore, no response should be displayed in a POS Terminal.  

---

## Inquiry

{{% notice info %}}
An 'AM Cash Inquiry' request is used to get balances and product offer information.  
{{% /notice %}}

### POST /v1/partners/me/amcash/inquire

**Sample cURL:**
```curl
curl -X POST \
  https://sandbox.api.loyalty.com/v1/partners/me/amcash/inquire \
  -H 'Authorization: Bearer eyJ0e…6sA' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
    "collectorNumber": "###########",
    "postTaxBasketAmount": "10.00",
    "traceId": "############",
    "dateTime": "1972-10-04T04:39:59.503-0400",
    "cardEntryMode": "Scan",
    "acquirerId": "######",
    "vendorId": "######",
    "sponsorCode": "XXXX",
    "deviceId": "###",
    "storeId": "####"
}’
``` 

**Sample success response:**
```curl
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Mon, 29 Oct 2018 15:27:34 GMT
{
    "collectorNumber": "###########",
    "cashBalance": "1615",
    "dreamBalance": "4",
    "transDateTime": "2018-10-25T11:41:55",
    "availableCashBalance": "1615",
    "availableDollarAmount": "170.00",
    "traceId": "############",
    "dateTime": "2018-10-04T04:39:59",
    "acquirerId": "######",
    "responseCode": "0000",
    "sponsorCode": "XXXX",
    "storeId": "####",
    "deviceId": "###",
    "vendorId": "######",
    "unitValueAMCash": "95",
    "unitValueDollars": "10.00",
    "availableOfferUnits": "1",
    "dailyOfferLimit": "17",
    "offerType": "RGL",
    "offerId": "#######",
    "offerDescEn": "Smoke Test - AIR MILES Cash instant redemption at L1",
    "offerDescFr": "Smoke Test - Échange instantané Argent AIR MILES chez L1",
    "message": "Approved"
}
```

### Inquiry Request Parameters

| Parameter            | Sample Value                 | Description                                                                                                                                      | Parameter Type | Data Type                                  |
|----------------------|------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|----------------|--------------------------------------------|
| Content-Type*        | application/json             |                                                                                                                                                  | header         | String                                     |
| Authorization*       | <<Bearer Token>>             | The Bearer token obtained from   the Authorization endpoint.                                                                                     | header         | String                                     |
| traceId*             | 12345678                     | System Trace Number.                                                                                                                             | body           | String, length 12   digits.                |
| sponsorCode*         | <<provided by LoyaltyOne>>   | The code used to identify the   partner.                                                                                                         | body           | String, length 4.                          |
| storeId*             | 1                            | The Store Id.                                                                                                                                    | body           | String, length 4   digits.                 |
| deviceId*            | 1                            | The Device Id.                                                                                                                                   | body           | String, length 3 digits.                   |
| vendorId*            | <<provided by   LoyaltyOne>> | The vendor Id used to identify the partner.                                                                                                      | body           | String, variable   length up to 35 digits. |
| collectorNumber*     | 80000000001                  | The Collector Number.                                                                                                                            | body           | String, length up to 19 digits.            |
| postTaxBasketAmount* | 16.12                        | The dollar/monetary amount spent by the Collector to be   considered for AM Cash (post-tax).                                                     | body           | String, length up to   16 digits.          |
| dateTime*            | 1972-07-16T13:32:00.000+0400 | The local date/time when the   partner sends the request.      Format: ISO8601 combined Date/Time Format.                                        | body           | String, length up to 20.                   |
| cardEntryMode*       | scan                         | The CardEntryMode is used to identify how a transaction   completed at the POS.      Manual: “Manual”     Barcode: “Scan”     Magstripe: “Swipe” | body           | String, length 50   characters.            |
| acquirerId*          | 100000                       | The Acquirer ID identifies   the party that sends the request, usually this is the partner.                                                      | Body           | String, variable length up to 11 digits.   |

### Inquiry Response Messages

| HTTP Status Code | responseCode Value | Reason                                                 |
|------------------|--------------------|--------------------------------------------------------|
| 200              | 0                  | OK                                                     |
| 202              | 1                  | Approved, balance suppressed.                          |
| 202              | 5                  | Balance Suppressed and unable to redeem.               |
|                  | 6                  | Unable to redeem.                                      |
| 202              | 51                 | Insufficient Air Miles Reward Miles.                   |
| 202              | 65                 | Exceeded daily redemption limit.                       |
| 400              | 35                 | Bad request – missing, invalid parameters or   values. |
| 401              | N\A                | Missing or invalid Bearer token.                       |
| 401              | 2                  | Collector account inactive.                            |
| 401              | 25                 | Invalid Collector Number.                              |
| 403              | 9                  | Card not swiped or scanned.                            |
| 408              | 8                  | Transaction timed out.                                 |
| 500              | 96                 | Server error                                           |
| 503              | 92                 | Scheduled outage                                       |

### Inquiry Response Model

| Field                 | Description                                                                                                                                                                                                                         | Data Type                                  |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------|
| responseCode          | The Response Code indicates the result of the   transaction.                                                                                                                                                                        | String,   length 4 digits.                 |
| message               | Message that describes the   status of the response.                                                                                                                                                                                | String                                     |
| traceId               | The traceId provided in the request.                                                                                                                                                                                                | String,   length 12 digits.                |
| dateTime              | The transmission date and time   using ISO-8061 from Partner Platform to AM Cash.                                                                                                                                                   | String, length 20.                         |
| sponsorCode           | The sponsorCode provided in the request.                                                                                                                                                                                            | String,   length 4.                        |
| vendorId              | The vendorId provided in the   request.                                                                                                                                                                                             | String, variable length up to 35 digits.   |
| storeId               | The storeId provided in the request.                                                                                                                                                                                                | String,   length 4 digits.                 |
| deviceId              | The deviceId provided in the   request.                                                                                                                                                                                             | String, length 3 digits.                   |
| collectorNumber       | The collectorNumber provided in the request.                                                                                                                                                                                        | String,   length up to 19 digits.          |
| cashBalance           | The number of Air Miles Reward   Miles Cash balance the Collector has.                                                                                                                                                              | String, length up to 16 digits.            |
| dreamBalance          | The number of Air Miles Reward Miles Dream balance   the Collector has.                                                                                                                                                             | String,   length up to 16 digits.          |
| transDateTime         | The transmission date and time   from the Instant POS Platform using ISO-8061 from AM Cash to partner.                                                                                                                              | String, length 20.                         |
| availableCashBalance  | The amount of Cash Air Miles Reward Miles available   for redemption.                                                                                                                                                               | String,   length up to 16 digits.          |
| availableDollarAmount | The Available $ Amount is used   to return the dollar/monetary amount equivalent to the Cash Air Miles Reward   Miles available for redemption (rounded down to the nearest integer that is   multiple of the current offer value). | String, length up to 16 digits.            |
| acquirerId            | The acquirerId provided in the request.                                                                                                                                                                                             | String,   variable length up to 11 digits. |
| unitValueAMCash       | OPTIONAL – The number of Air   Miles Cash Miles for a unit of redemption.                                                                                                                                                           | String, length up to 3 digits.             |
| unitValueDollars      | OPTIONAL – The dollar value for a unit of   redemption.                                                                                                                                                                             | String,   length up to 3 digits.           |
| availableOfferUnits   | OPTIONAL - Offer units available   for the transaction.                                                                                                                                                                             | String, length 3 digits.                   |
| dailyOfferLimit       | OPTIONAL – The number of units a Collector can   redeem in a day.                                                                                                                                                                   | String,   length 3.                        |
| offerType             | OPTIONAL - Offer type could be   RGL (for Regular offers), SPL (for Special/Discounted offers)                                                                                                                                      | String, length 3.                          |
| offerId               | OPTIONAL – AM Cash Offer Id                                                                                                                                                                                                         | String,   length up to 10 digits.          |
| offerDescEn           | OPTIONAL – Offer description in   English                                                                                                                                                                                           | String, length up to 50.                   |
| offerDescFr           | OPTIONAL – Offer description in French                                                                                                                                                                                              | String,   length up to 50.                 |

{{% notice tip %}}  
Responses will always include `responseCode` and message fields. Only fields with values ("non-null" or "non-empty" values) will be part of the response model.  
Also, be aware that `unitValueAMCash`, `unitValueDollars`, `availableOfferUnits`, `dailyOfferLimit`, `offerType`, `offerId`, `offerDescEn`, `offerDescFr` are optional returns in success responses.
{{% /notice %}}  

### Using AM Cash Inquiry

The AM Cash Inquiry messages are used to retrieve Collector balance and available offers information. 
Once the Partner Platform receives the response, the POS Terminal can acquire the reward miles Cash and Dream balances; these balances should be shown in the receipt that the Collector will receive but should not be displayed in the POS Terminal. The receipt and POS Terminal can also display Available Cash Balance and Available $ Amount from the response. 

The optional elements in the response model will be returned if:  

* The Collector Number is swiped or scanned (‘cardEntryMode’ is either “swipe” or “scan”) .

* The Collector has enough Air Miles Reward Miles Dream Miles balance.

* The Collector has not exceeded the daily limit for redemptions.

The optional data elements contain information needed to show the Collector the number of Cash Reward Miles that they can redeem for and the discount that would be applied towards the purchase.

* The ‘unitValueAMCash’ value shows the number of miles that a unit equals to (e.g. each unit to redeem is equal to 95 AMRM). The Collector should be told about this value so he/she is aware of how many miles it will cost to do redemption. 

* The unitValueDollars value shows the number of dollars/monetary amount that a unit equals to (e.g. each unit that is redeemed is equal to a monetary amount of $10 off of the purchase). The Collector should be shown this value, so they are aware of the purchase discount that can get by redeeming certain amount of reward miles. 

* availableOfferUnits is the number of units available to be redeemed in a particular transaction. Based on the qualifying amount a Collector may be able to redeem for less than what they are allowed to redeem during a day. For example, if a qualifying amount is monetary amount $55, based on the Collector balance and daily cap they could redeem for monetary amount of $100. This value of ‘availableOfferUnits’ will be $5 as the Collector could redeem for 5 units which equal to monetary amount $50. 

* dailyOfferUnits is the number of units available to be redeemed through a 24 hours period as described above. The number of units is based on the offers’ limits, the Collector’s Cash balance and previous redemptions/reversals in the 24 hours period. 

* offerType: describes whether offer is regular or special/discounted. Used “RGL” (for Regular offers), “SPL” (for Special/Discounted offers)

* offerId: ID associated with the offer in our offer repository.

* offerDescEn: Offer description in English. The Collector should be shown this value, so they are aware about offer details. 

* offerDescFr: Offer description in French. The French Collector should be shown this value, so they are aware about offer details. 

The usage of the AM Cash Inquiry elements and how to integrate them with POS Terminals is shown in section 7.1.

### Call Sequence for AM Cash Inquiry

This sequence description assumes that the Partner Platform either relies (just proxies) or constructs/interprets the message (based on information sent by the POS Terminal/Instant POS Platform). This means that the logic (e.g. for retries or response code interpretation) can be done on the POS Terminal or the Partner Platform. It is up to the partner to decide where that logic should be placed. 

The Partner Platform (or the POS Terminal) should do the following according to the response received from the Instant POS Platform: 

* If ‘responseCode’ is 0000 (Approved) or 0001 (Approved – Balance Suppressed) is received, execution should proceed normally, and the AM Cash Inquiry response should be displayed in the POS Terminal as explained in section 7.1. Offer information is always returned in response if response code is 0000 or 0001.

* If ‘responseCode’ is 0008 (timeout) is received, the Partner Platform (or POS Terminal) should automatically retry once the AM Cash Inquiry request. This means that the cashier or Collector should not have to provide manual input for this retry. 

* If No Response is received, the Partner Platform (or POS Terminal) should automatically retry once the AM Cash Inquiry request. Again, this means that the cashier or Collector should not have to provide manual input for this retry.

* If the retry of the AM Cash Inquiry also fails (i.e. another 0008 responseCode is received, or no response is received), the appropriate error message should be displayed as suggested in section 6.4.1. 

* If any other response code is received (e.g. 0002, 0096, etc.), whether in a request or a subsequent retry of that request, the proper message should be displayed in accordance with section 6.4.1.

## Redemption

